# Rapport d'Architecture Technique : Framework de Trading Aegis
**Version :** 2.2 (Production - Topologie Bus & Annexes)  
**Classification :** Secret Industriel / Confidentiel  
**Statut :** Validé pour Déploiement  

---

## Introduction

Le framework **Aegis** est une infrastructure de trading algorithmique propriétaire à haute performance et haute disponibilité. Conçu pour s'interposer entre les flux de cotations de courtiers (comme IG Markets) et des moteurs de décision quantitatifs, Aegis résout les problématiques critiques de la production : isolation des secrets financiers, résilience face aux pannes internet, exécution déterministe et sécurité périmétrique absolue. 

Ce rapport documente l'architecture découplée en micro-services asynchrones (Event-Driven) d'Aegis, mettant en exergue sa topologie en bus de messages (ZeroMQ), le rôle souverain du poste utilisateur, et le cloisonnement de ses sauvegardes.

---

## Architecture Globale & Topologie du Bus ZeroMQ

L'écosystème global se segmente en trois environnements physiques distincts afin de garantir qu'aucun composant critique ne soit exposé directement à Internet :

1. **Le Poste Utilisateur (Cockpit Humain)** : Point d'accès unique et détenteur exclusif des secrets maîtres.
2. **Le Serveur Principal (Production)** : Héberge l'intelligence logique et l'orchestration des données de marché.
3. **Le Serveur Secondaire (Backup)** : Serveur passif de stockage à froid, dédié aux données historiques et logs.

### Schéma d'Architecture : La Colonne Vertébrale ZeroMQ

```text
┌─────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                       POSTE UTILISATEUR (COCKPIT)                                       │
│  - Détient exclusivement l'Username / Password / Clé API maîtres                                         │
│  - Effectue le Login initial ➔ Génère les Tokens (CST / X-SECURITY-TOKEN)                               │
│  - Injecte directement les TOKENS dans la Brique 1 du serveur via le tunnel sécurisé                    │
└──────────────────────────────────────────────────┬──────────────────────────────────────────────────────┘
                                                   │ (1. Injection brute des TOKENS en RAM)
                                                   ▼
┌─────────────────────────────────────── SERVEUR PRINCIPAL (PROD) ───────────────────────────────────────┐
│                                                                                                         │
│  ┌────────────────────────┐         (2. Fournit les Tokens)       ┌──────────────────────────────────┐  │
│  │ Brique 1: Authenticator│ ────────────────────────────────────► │     Brique 3: IG Connector       │  │
│  │ (Keep-Alive RAM Only)  │                                       │     (REST / Lightstreamer)       │  │
│  └────────────────────────┐                                       └────────────────┬─────────────────┘  │
│                                                                                    │                    │
│                                           ╔════════════════════════════════════════╧═════════════════╗  │
│                                           ║               BUS SÉCURISÉ ZEROMQ (127.0.0.1)            ║  │
│                                           ║                                                          ║  │
│                                           ║  [Flux MKT / EXEC / HB] ──► Port 5555 (PUB / SUB)        ║  │
│                                           ║  [Flux EMERGENCY CMD]   ──► Port 5556 (PUSH / PULL)      ║  │
│                                           ╚═══════╦══════════════╦══════════════╦══════════════╦═════╝  │
│                                                   ║              ║              ║              ║        │
│                                                   ▼              ▼              ▼              ▼        │
│                                            ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌────────────┐  │
│                                            │  Brique 2: │ │  Brique 4: │ │  Brique 5: │ │  Brique 6: │  │
│                                            │  Le Bot    │ │  Data      │ │  Sys       │ │  PWA       │  │
│                                            │  Aegis     │ │  Recorder  │ │  Monitor   │ │  Dashboard │  │
│                                            └──────┬─────┘ └──────┬─────┘ └────────────┘ └────────────┘  │
│                                                   │              │                                      │
│                                      (Alertes JSON)              ▼                                      │
│                                                   │  [ Fichiers Archivés .zst ]                         │
│                                                   ▼              │                                      │
│                                            ┌────────────┐        │                                      │
│                                            │  Brique 7: │        │                                      │
│                                            │  Matrix    │        │                                      │
│                                            └────────────┘        │                                      │
└──────────────────────────────────────────────────────────────────┼──────────────────────────────────────┘
                                                                   │ (Restic Sync)
                                                                   ▼
┌─────────────────────────────────────── SERVEUR SECONDAIRE (BACKUP) ─────────────────────────────────────┐
│  - Dépôt de stockage distant (Chiffré localement en AES-256 en amont via Restic)                        │
│  - Interdiction stricte de stocker le fichier .env (Pas de secrets de reconnexion)                      │
│  - Contient UNIQUEMENT : Fichiers Ticks Historiques (.tar.zst) + Journaux de logs systèmes             │
└─────────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

## Analyse Détaillée des Briques Logicielles

### Brique 1 : L'Authenticator (Le Mainteneur de Session)
* **Rôle** : Cette brique n'a plus de rôle de connexion initiale. Elle reçoit les tokens (`CST` et `X-SECURITY-TOKEN`) générés et injectés par le Cockpit Utilisateur. Elle stocke ces jetons **exclusivement en mémoire RAM** (aucun fichier de configuration persistant sur le serveur).
* **Flux** : Sa tâche est double :
  1. Elle distribue les jetons valides à la Brique 3 au démarrage du flux réseau.
  2. Elle fait tourner une boucle de micro-tâches asynchrones (`asyncio`) qui envoie un ping (Keep-Alive) toutes les 5 minutes à l'API d'IG Markets pour rafraîchir et maintenir la session active. Si la session expire ou coupe, elle est incapable de redémarrer sans une nouvelle injection manuelle depuis le Cockpit.

### Brique 3 : L'IG Connector (La Passerelle Réseau Brute)
* **Rôle** : Cette brique est l'unique fenêtre ouverte sur l'API d'IG Markets. Elle prend en charge la connexion WebSocket permanente (Lightstreamer) pour recevoir les flux de ticks bruts et exécute les ordres de trading en REST.
* **Connexion Bus** : Elle récupère les tokens de la Brique 1 pour s'initialiser. Dès qu'un tick est reçu, elle le nettoie, calcule la latence d'infrastructure (`heure_locale - timestamp_ig`) et redresse les quantités d'ordres au format absolu (`abs()`). Elle se greffe en haut du bus ZeroMQ en tant que **`zmq.PUB` (Port 5555)** pour diffuser ses données sous forme de payloads JSON normalisés.

### Brique 2 : Le Bot Aegis (Le Moteur Déterministe)
* **Rôle & Autonomie** : Totalement indépendant des couches réseau d'IG. C'est une machine à états purement déterministe qui prend les décisions de trading quantitatif.
* **Connexion Bus** : Il vient se brancher sur la colonne vertébrale ZeroMQ via deux sockets :
  1. Un socket **`zmq.SUB` (Port 5555)** pour avaler en continu les ticks normalisés diffusés par la Brique 3.
  2. Un socket **`zmq.PULL` (Port 5556)** branché sur le canal de contrôle prioritaire pour recevoir instantanément l'ordre de coupure d'urgence (Killswitch).

### Brique 4 : Le Data Recorder (L'Accumulateur Historique)
* **Rôle & Autonomie** : Il n'a aucun lien avec le Bot ou le Connecteur. C'est un service d'arrière-plan dédié à l'écriture disque.
* **Connexion Bus** : Il ouvre un socket **`zmq.SUB` (Port 5555)**. Dès qu'un message de type `"MKT"` (Market Data) circule sur le bus, il l'intercepte et l'écrit ligne par ligne dans un CSV journalier (`buffering=1`). Chaque nuit à `00:05`, il ferme le fichier, appelle l'outil **Zstandard (`zstd`)** de Meta pour générer une archive compressée `.tar.zst` (gain de 85% d'espace), et supprime le fichier source.

### Brique 5 : Le System Monitor (La Télémétrie Matérielle)
* **Rôle & Connexion Bus** : Script indépendant qui utilise `psutil` pour inspecter la CPU, la RAM (RSS) et la dérive de l'horloge NTP toutes les 5 secondes. Pour centraliser ses métriques sans ajouter d'infrastructure, il encapsule ses données dans un paquet `"type": "SYS"` qu'il injecte directement sur le bus ZeroMQ (**Port 5555**).

### Brique 6 : Le Dashboard PWA (L'Interface Visuelle)
* **Rôle & Autonomie** : Interface web Streamlit. Elle écoute strictement sur la boucle locale `127.0.0.1:8501`. Elle est totalement indépendante du Bot. Elle se contente de lire les fichiers CSV de la Brique 4 pour tracer les graphiques et intercepte les paquets de la Brique 5 circulant sur le bus pour afficher la santé du serveur.

### Brique 7 : Matrix Synapse (La Messagerie Furtive)
* **Rôle** : Serveur de chat privé et sécurisé auto-hébergé sur la machine. Il reçoit les alertes JSON poussées par le Bot en cas de dysfonctionnement technique ou de comportement anormal du marché.

---

## Le Poste Utilisateur (Cockpit) & Le Dispositif Mobile

Le poste de l'utilisateur agit comme la seule "clé de contact" et l'unique tour de contrôle du système.

* **La Règle des Secrets** : Le fichier de configuration sensible `.env` et les identifiants maîtres ne résident **que** sur la machine physique de l'utilisateur (ou sa clé d'authentification matérielle). Aucun script automatique sur le serveur ne dispose du droit de générer une session de lui-même.
* **Le Tunneling Chiffré** : L'accès aux interfaces visuelles et de messagerie s'effectue en couplant des redirections de ports à travers un tuyau SSH ou VPN WireGuard durci (authentification par clé asymétrique **Ed25519**, port SSH masqué `48192`) :
  ```bash
  ssh -N -L 9000:127.0.0.1:8501 -L 9001:127.0.0.1:8008 -p 48192 utilisateur@IP_SERVEUR
  ```
* **Le Killswitch Tactile Mobile** : En situation de mobilité, l'utilisateur active le VPN WireGuard sur son smartphone pour rejoindre le réseau local. Il accède à sa PWA sécurisée et peut presser le bouton d'urgence **Killswitch**. Cette action pousse une commande d'annulation immédiate sur le canal ZeroMQ `PULL` (Port 5556), forçant le Bot à verrouiller ses entrées de stratégie et à liquider les positions en cours.

---

## Le Serveur Secondaire : Stockage Exclusif à Froid

Le serveur secondaire est une machine d'archivage déconnectée de la logique applicative.

* **Exclusion Absolue des Secrets** : Le fichier `.env` ou toute trace de tokens de session **sont strictement interdits de transfert vers cette machine**. Si le serveur secondaire est compromis, l'attaquant ne met la main que sur des fichiers de données passées.
* **Le Coffre-Fort de Données (Restic)** : Chaque nuit à `00:15`, le serveur principal chiffre localement (en AES-256) ses répertoires de données à l'aide de l'outil **Restic**. Les paquets chiffrés sont poussés vers le serveur secondaire.
* **Contenu du Stockage** : Le serveur secondaire stocke uniquement :
  1. Les archives compressées de ticks historiques (`.tar.zst`).
  2. Les sauvegardes des journaux de logs système (historiques d'exécution Linux, traces d'alertes).
  Restic gère la politique de rétention à l'extérieur pour ne conserver que les 7 derniers snapshots quotidiens, 4 hebdomadaires et 12 mensuels.

## Annexe A : Stratégie de Sécurité Périmétrique & Durcissement (Security Hardening)

Pour sanctuariser le capital disponible sur le compte de trading IG Markets et protéger le secret industriel de vos algorithmes, le serveur de production applique une politique de **durcissement strict de niveau bancaire**.

### A.1 : Configuration et Règles du Pare-feu (`UFW` / `iptables`)
Le principe fondamental de la sécurité d'Aegis est le **refus systématique de toute connexion entrante** (Default Deny). Le serveur de trading n'est pas un serveur web public.
* Tout le trafic entrant par défaut est bloqué (`ufw default deny incoming`).
* Tout le trafic sortant par défaut est autorisé (`ufw default allow outgoing`) pour permettre aux briques 1 et 3 de communiquer avec les API d'IG.
* **L'unique exception** : Le port SSH personnalisé (ex: `Port 48192`) est ouvert, mais **exclusivement restreint à l'adresse IP publique fixe de votre Cockpit Utilisateur**. Tout paquet SSH provenant d'une autre IP est jeté silencieusement au niveau du noyau Linux (Drop) sans répondre, rendant le serveur invisible aux scanners de ports.

### A.2 : Isolation Absolue des Interfaces Locales (Loopback)
Toutes les communications inter-briques (ZeroMQ sur les ports 5555 et 5556, Streamlit sur le port 8501, Matrix sur le port 8008) sont configurées pour se lier **uniquement à l'adresse `127.0.0.1`** (interface de boucle locale `lo`). 
* Même si une erreur de configuration survenait dans le pare-feu global, ces ports sont physiquement incapables d'écouter ou de répondre à des paquets provenant d'une interface réseau externe (comme `eth0`). Ils n'existent que dans la mémoire RAM du serveur.

### A.3 : Authentification SSH et Système Anti-Brute Force
* L'accès par mot de passe est désactivé au niveau du démon SSH (`PasswordAuthentication no`). Seules les clés asymétriques à haute résistance cryptographique **Ed25519** sont acceptées.
* Le compte `root` a interdiction stricte de se connecter à distance (`PermitRootLogin no`).
* L'outil **Fail2Ban** est déployé en tâche de fond. Il surveille le fichier `/var/log/auth.log`. Si une adresse IP tente d'initier une connexion SSH en échec plus de 3 fois, Fail2Ban injecte immédiatement une règle de bannissement stricte dans le pare-feu pour bloquer cette IP pendant 24 heures.
* Le Bot Aegis (Brique 2) et le Data Recorder (Brique 4) s'exécutent sous un utilisateur système Linux dédié sans privilèges (`aegis_runner`). Si une faille de sécurité survenait via une bibliothèque tierce installée par pip, l'attaquant se retrouverait enfermé dans un espace utilisateur sans aucun droit d'accès aux commandes d'administration du système (`sudo`).

### A.4 : Sécurisation du Dispositif Mobile et du Killswitch
Lorsque vous pilotez le bot depuis votre téléphone, la sécurité repose sur deux piliers :
1. **Réseau (WireGuard)** : Votre téléphone doit initier une connexion VPN chiffrée WireGuard pour obtenir une IP privée (ex: `10.0.0.2`). C'est ce tuyau chiffré qui lui permet de "voir" le port `8501` du Dashboard Streamlit.
2. **Filtrage d'Identifiant (White-listing)** : Le bouton de commande d'urgence (Killswitch) envoyé par votre application mobile transite par un script filtrant. Ce script vérifie de manière stricte votre **Identifiant Unique de Compte (ID Cryptographique)**. Si un message de commande provient d'un autre utilisateur, le paquet est immédiatement détruit et une alerte critique `CRITICAL [INTRUSION]` est loguée dans le journal système.

---

## Conclusion

L'architecture révisée du framework Aegis garantit un niveau de sécurité industriel et souverain. En privant le serveur de production du stockage permanent des identifiants maîtres (congelés sur le Cockpit Utilisateur), en matérialisant les flux internes par une colonne vertébrale ZeroMQ parfaitement cloisonnée (Ports 5555 et 5556), et en interdisant la présence de secrets sur le serveur secondaire de backup, le système élimine les principaux vectors d'attaque de l'écosystème. Aegis se positionne comme une infrastructure de trading hermétique, hautement performante et mathématiquement isolée des risques d'intrusion ou de dérive réseau.

