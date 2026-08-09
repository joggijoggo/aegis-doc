# PARTIE 1 : JALON 0 ET JALON 1 (L'INFRASTRUCTURE DE DONNÉES)

## 🛠️ JALON 0 : EXTENSION ET CAPTURE DU CARNET D'ORDRES (PRÉPARATOIRE)
*L'objectif est d'assurer que l'infrastructure externe est capable de fournir la liste des ordres en cours au tracker, au même titre que l'account et les positions, avant d'activer la réconciliation.*

### Sous-Jalon 0.1 : Connecteur de Capture Backtrader
*   **Objectif** : Brancher un point d'écoute au sein de la boucle ou du store Backtrader pour appeler systématiquement la méthode native `strategy.broker.get_orders_open(safe=False)`. Extraire la liste brute des objets `Order` en cours de validité.
*   **Livrable** : Flux de carnet émulé en RAM disponible à chaque début de cycle de la stratégie.

### Sous-Jalon 0.2 : Connecteur de Capture REST IG Markets
*   **Objectif** : Implémenter l'appel HTTP synchrone sur l'endpoint officiel d'IG Markets : `/workingorders`. Parser le dictionnaire JSON racine `'workingOrders'` renvoyé par les serveurs d'IG pour collecter les structures de travail brutes.
*   **Livrable** : Handler de requêtage capable de ramener l'état du carnet IG à la demande, en respectant les contraintes de formatage du courtier.

---

## 🧼 JALON 1 : L'INFRASTRUCTURE COMPTABLE AGNOSTIQUE (LE COMMON GROUND)
*L'objectif est d'isoler l'OrderGroup comme un miroir comptable purement passif et de le doter de son API de guérison déterministe.*

### Sous-Jalon 1.1 : Standardisation Sémantique du Ledger et du Carnet
*   **Objectif** : Implémenter les helpers de lecture unifiés. Exposer la vacuité du capital (`is_flat()`) au niveau du Ledger et le statut d'interruption du clearing (`is_clearing_closed`) en lecture seule sur l'OrderGroup.
*   **Livrable** : Propriétés et méthodes sémantiques pures, prêtes à être lues par l'extérieur.

### Sous-Jalon 1.2 : Libération de la Tuyauterie Passive (`notify`)
*   **Objectif** : Retirer définitivement tous les blocs de crash précoces (`if self._is_corrupted: raise ...`) au sommet des canaux d'ingestion réseau. L'OMS doit collecter les données du marché en continu sans jamais bloquer le thread de trading.
*   **Livrable** : Méthodes `notify_order_change` et `notify_trade_change` entièrement perméables et muettes.

### Sous-Jalon 1.3 : Le Moteur de Réconciliation Interne
*   **Objectif** : Coder la méthode publique `reconcile()` au cœur de l'OrderGroup. Elle utilise le Snapshot d'ordres désormais disponible grâce au Jalon 0 pour écraser la position physique du ledger, aligner le flag de clearing, et purger (flush) le carnet local en toute sécurité en basculant de force les ordres orphelins absents du set vers l'état terminal `CANCELED`.
*   **Livrable** : Moteur d'auto-alignement in-memory déterministe et hautement testable de manière unitaire.

# PARTIE 2 : JALON 2 ET JALON 3 (LA MACHINE À ÉTATS ET LA SÉCURITÉ)

## 🛡️ JALON 2 : LA SÉCURITÉ DE L'EXECUTION TRACKER (LES DEUX SYSTÈMES ET LA GRÂCE)
*L'objectif est de transférer l'intelligence opérationnelle et la gestion du risque vers le contrôleur externe (l'ExecutionTracker) en isolant cliniquement le réseau et le risque de marché.*

### Sous-Jalon 2.1 : Déploiement du Système 1 (L'Audit de Désynchronisation Réseau)
*   **Objectif** : Extraire le Système 1 dans sa sous-fonction dédiée `_audit_network_coherence()`. Implémenter le registre privé de persistance de l'Option B (compteur de ticks/tours de boucle du moteur). Si le ledger et le clearing divergent, le compteur s'incrémente. S'il franchit le seuil de grâce (seuil ≥ 2), le tracker valide une panne réseau réelle et déclenche la phase de réconciliation.
*   **Livrable** : Amortisseur de bruit réseau capable d'absorber les inversions temporelles de paquets sans générer de faux crashs.

### Sous-Jalon 2.2 : Déploiement du Système 2 (La Surveillance de l'Over-Hedge)
*   **Objectif** : Extraire le Système 2 dans sa sous-fonction dédiée `_evaluate_market_risk()`. Elle interroge la méthode quantitative brute `is_over_hedged()`. Si le danger de marché immédiat est avéré (position ouverte face à un carnet d'ordres logiques mort ou vide), elle verrouille le groupe de manière passive et lève l'exception financière de domaine (*Fail-Fast* transitoire).
*   **Livrable** : Disjoncteur financier de sécurité immédiat protégeant le capital contre les rejets ou la mort des ordres de protection.

---

## ⏱️ JALON 3 : INTÉGRATION NORMÉE DE L'ÉTAT LÉGAL `EXPIRED`
*L'objectif de ce jalon est de formaliser la fin de validité temporelle des ordres au sein du carnet, et d'assurer une clôture nominale des cycles sans déclencher de fausses alertes d'over-hedging.*

### Sous-Jalon 3.1 : Évolution de l'Invariance et du Poids Réseau de l'Ordre
*   **Objectif** : Promouvoir `OrderState.EXPIRED` au statut d'état terminal dans les enums. L'intégrer aux matrices de pondération réseau avec la priorité maximale pour écraser instantanément tout message d'état antérieur partiel arrivé en retard.
*   **Livrable** : Alignement de l'infrastructure des types sur les contraintes temporelles des bourses.

### Sous-Jalon 3.2 : Résolution de l'Expiration de l'Entrée Parent à Plat
*   **Objectif** : Ajuster l'évaluation de la propriété calculée `.state` dans sa branche à plat (`is_flat == True`). Si l'ordre d'entrée parent expire en fin de session sans avoir engagé de capital, le groupe d'ordres global doit résoudre et se clore proprement à l'état nominal `CANCELED`.
*   **Livrable** : Fermeture fluide et déterministe des intentions d'achats ou de ventes non exécutées.


# PARTIE 3 : JALON 4 ET JALON 5 (FLUX CONTINUS ET CONTRE-ATTAQUE ACTIVE)

## 🌊 JALON 4 : TUNNEL DES REMPLISSAGES INCRÉMENTAUX (`PARTIALLY_FILLED`)
*L'objectif est d'doter l'OMS d'un moteur d'accumulation volumétrique pour absorber les vagues de liquidité fractionnées successives.*

### Sous-Jalon 4.1 : Perméabilité Réseau de l'État Partiel
*   **Objectif** : Ouvrir les vannes de filtrage au sein de `notify_order_change` pour autoriser explicitement la transition continue d'un état partiel vers un autre état partiel (`PARTIALLY_FILLED` → `PARTIALLY_FILLED`), avec un poids réseau intermédiaire.
*   **Livrable** : Capacité physique d'ingérer les flux fragmentés sans blocage logique.

### Sous-Jalon 4.2 : Registre d'Accumulation et Protocole de Saturation "Maxer"
*   **Objectif** : Déployer un dictionnaire privé de suivi des volumes cumulés par ticket (`self._executed_quantities`). Coder la barrière de saturation : lorsque le statut final `FILLED` arrive, calculer le solde exact manquant à envoyer au ledger d'après la taille d'origine, saturer le compteur au maximum nominal de la commande et rejeter définitivement tout message partiel tardif arrivant en retard.
*   **Livrable** : Protection comptable absolue contre le risque de double-comptabilisation ou de pollution du ledger par doublons réseau.

### Sous-Jalon 4.3 : Refactoring Volumétrique Flottant et Dimensionnement de l'Exit
*   **Objectif** : Convertir les fonctions de vérifications de risques (`is_over_hedged`, `children_triggered`, `exit_triggered`) en calculs dynamiques par soustraction du volume cumulé exécuté. Calibrer la taille de l'ordre d'urgence `-XT` généré lors d'une liquidation sur la valeur absolue instantanée de la position du ledger.
*   **Livrable** : Dimensionnement micrométrique des parachutes de sécurité ajusté à la taille réelle de l'exposition.

---

## 🛡️ JALON 5 : CONTRE-ATTAQUE ACTIVE ET NORMALISATION AGNOSTIQUE
*L'objectif final est de brancher le tracker sur l'API de production d'IG Markets et les objets réels de Backtrader, et de transformer les crashs logiciels en actions d'autodéfense.*

### Sous-Jalon 5.1 : Couche d'Adaptation et Normalisation des Snapshots
*   **Objectif** : Coder les traducteurs finaux au sommet de chaque cycle du moteur. Convertir la structure brute d'ordres récupérée au Jalon 0 (objets Backtrader et JSON IG `/workingorders`) ainsi que les positions vers notre format agnostique unifié (`market_exposure`, `is_clearing_closed`, `working_order_ids: set[str]`). Implémenter le mapping par la logique de Triple Barrière (Info → RAM → Invariant).
*   **Livrable** : Module de capture et de normalisation de données 100 % agnostique.

### Sous-Jalon 5.2 : Unification de la Lignée de Sortie et Coupe-Circuit
*   **Objectif** : Remplacer l'ID d'exit unique par une liste continue d'identifiants d'urgences (`self._exit_order_ids`). Implémenter la méthode unifiée de détection des ordres en vol pour activer un verrouillage de sécurité anti-inondation : interdire au tracker de renvoyer un ordre d'urgence si un exit précédent est toujours actif en bourse.
*   **Livrable** : Coupe-circuit anti-emballement protégeant le compte contre le spamming d'ordres sur le marché.

### Sous-Jalon 5.3 : Déploiement de la Boucle Active d'Autodéfense
*   **Objectif** : Dans la méthode `_evaluate_market_risk` de l'ExecutionTracker, supprimer définitivement la ligne transitoire `raise`. La remplacer par la commande de tir automatique : calculer la taille exacte, inverser la polarité du ledger, et propulser l'ordre `MARKET` de liquidation unifié auprès du courtier. Le ledger retombe à zéro, et le groupe se ferme nativement à l'état `COMPLETED`.
*   **Livrable** : Autonomie complète du robot Aegis, capable de cicatriser et de défendre son capital de manière chirurgicale face aux défaillances de marché.

