# 🛡️ FRAMEWORK AEGIS : RAPPORT D'ARCHITECTURE OMS
**Date :** 9 août 2026  
**Version :** v2.0.0-RC1  
**Statut :** Spécifications Validées  

---

## 1. INTRODUCTION

### 1.1 Contexte du Framework Aegis
Le framework **Aegis** est un système d'exécution à haute performance et de gestion des ordres (*Order Management System* - OMS) conçu pour le trading algorithmique systématique et quantitatif. Au cœur de l'infrastructure, l'interconnexion entre trois composants clés régit la stabilité financière de l'application :
1. **`OrderGroup`** : Agit comme une machine à états logique centralisée, responsable de la cohérence d'un groupe d'ordres interconnectés (ordre parent d'entrée et ordres brackets enfants de protection : *Stop-Loss* et *Take-Profit*).
2. **`ExecutionTracker`** : Gère le routage événementiel asynchrone des flux en direct en provenance des passerelles de courtage, l'aiguillage des paquets et le cycle de vie RAM des processus de trading.
3. **`ClearingLedger`** : Centralise la comptabilité physique pure d'un groupe d'ordres étanche (taille de position nette, prix moyen pondéré d'acquisition et profits et pertes réalisés), indépendamment de l'état des carnets d'ordres logiques.

### 1.2 Motivation
Dans un environnement de production multi-actifs, les décalages de latence réseau, le désordre des paquets d'exécution et la fragmentation microscopique de la liquidité boursière sont des réalités omniprésentes. La motivation principale de ce projet de refonte est de migrer le noyau d'Aegis d'une vision binaire et rigide de laboratoire ("Tout ou Rien") vers un **modèle d'ingénierie continue et asynchrone**, capable de s'auto-protéger de manière autonome en direct, sans jamais subir de crash applicatif bloquant, tout en sanctuarisant le capital exposé.

### 1.3 Problématique
L'audit microstructural de la base de code existante a révélé quatre failles architecturales majeures mettant le système en danger immédiat de sinistre financier :
* **Le Verrou Aveugle de la Corruption (Jalon 1)** : Lorsqu'une anomalie ou une exécution hors-protocole survient, le framework lève une exception lourde (`CorruptedOrderGroupError`) au sein même de ses fonctions passives d'écoute (`notify_order_change`). Cela rend l'OMS instantanément aveugle. Il cesse de mettre à jour son ledger comptable, paralyse son thread de trading et laisse la position physique sous-jacente ouverte au marché sans contrôle.
* **L'Absence de Gestion Temporelle (Jalon 2)** : Le framework ne gère pas les fins de validité des ordres (`EXPIRED`). L'expiration d'une barrière de protection ou l'échec d'un ordre d'urgence laisse une position physique orpheline et à nu, sans que la machine à états ne sache qualifier le risque.
* **La Faille du Flux Fractionné (Jalon 3)** : Le framework rejette les paquets réseau de poids identique et regarde les quantités initiales statiques des ordres. En cas d'exécutions partielles successives (*vagues de liquidité*), l'OMS ignore les accumulations de volume, calcule mal la quantité restante en vol (*Leaves*), et génère des ordres d'urgence surdimensionnés, risquant d'inverser accidentellement la position du compte à découvert.
* **La Passivité face au Risque (Jalon 4)** : Le système détecte les situations de sur-couverture ou de sous-couverture (*Over-Hedging* / *Under-Hedging*), mais ne possède aucune boucle active d'action pour intervenir de manière autonome. Il subit les décalages du marché sans contre-attaquer électriquement pour aplatir le risque.

---

## 2. SYNTHÈSE MANAGÉRIALE

Cette refonte dote le framework Aegis d'un protocole de résilience microstructurelle de niveau industriel, articulé autour de deux principes directeurs :
1. **L'Immuabilité de l'Écoute Passive** : Les canaux de communication et de réception de données boursières (`notify`) ne doivent **plus jamais être interrompus par des exceptions logicielles**. Quoi qu'il arrive sur le marché (anomalie, rejet ou exécution clandestine), l'OMS doit impérativement encaisser l'information pour maintenir une lecture à 100 % fidèle de la réalité financière du compte.
2. **L'Arbitrage par l'Invariant Physique du Ledger** : La logique comptable supplante la sémantique textuelle des états de commande. La machine à états globale ne cherche plus à deviner des combinaisons infinies de statuts d'ordres ; elle attend que le carnet boursier s'immobilise, puis compare les intentions logiques avec la taille de position physique réelle du ledger unique pour qualifier le cycle (Fermeture nominale, mise en attente sécurisée ou situation de danger).

### Cinématique Globale du Flux de Résilience

```text
[Flux Courtier] ──> [Filtre Réseau Perméable (Jalon 3)] ──> [Mise à jour Continue du Ledger]
                                                                     │
  ┌──────────────────────────────────────────────────────────────────┘
  ▼
[Carnet Immobile ?] ──> [NON] ──> Transition Opérationnelle (ACTIVE / CLOSING)
        │
     [OUI] ──> Interrogation Invariant Physique du Ledger
        │
        ├──> Position Nette == 0.0 ──┬──> [Aegis] ─────────> OrderGroupState.COMPLETED
        │                            └──> [Externe] ───────> OrderGroupState.CLANDESTINE_CLOSED
        │
        └──> Position Nette != 0.0 ────────────────────────> OrderGroupState.UNPROTECTED
                                                                     │
                                                                     ▼
                                                          [Boucle Active (Jalon 4)]
                                                          Ordre MARKET de Coupure Ajusté
                                                                     │
                                                                     ▼
                                                          [Ledger Remis à Plat]
                                                          OrderGroupState.COMPLETED
```

En déployant cette feuille de route cumulative de 4 jalons, Aegis passe d'un système passif vulnérable à un **OMS auto-nettoyant et auto-défensif**, immunisé contre l'asynchronisme des réseaux de trading.

## 3. SPÉCIFICATIONS TECHNIQUES DÉTAILLÉES

### 🧼 JALON 1 : Assainissement et Perméabilité des Flux Passifs

#### 1.1 Objectif Opérationnel
Éliminer le reliquat historique de blocage applicatif aveugle au profit d'un mécanisme de surveillance et de verrouillage stratégique fin. L'OMS doit sanctuariser ses canaux de réception passive pour décoder l'intégralité des flux du courtier, tout en catégorisant la nature exacte du risque au sein de l'énumération unique globale du groupe d'ordres.

#### 1.2 Spécifications d'Infrastructure (`core/models/enums.py`)
* **Refactoring de l'Enum Globale** : Supprimer définitivement la valeur `CORRUPTED` de l'énumération `OrderGroupState`.
* **Insertion des États d'Anomalie Unifiés** : Introduire deux nouveaux états logiques étanches :
    * `CLANDESTINE_CLOSED = 'CLANDESTINE_CLOSED'` : Signalé lorsque le clearing boursier notifie la fermeture complète d'une position physique en dehors du protocole nominal d'Aegis. La position est à plat, le capital est hors de danger, mais les univers sont désynchronisés.
    * `UNPROTECTED = 'UNPROTECTED'` : Signalé lorsque le carnet d'ordres s'immobilise définitivement alors qu'une position physique reste ouverte et exposée au marché sans aucune barrière de protection bracket active. Le capital est en danger immédiat.
* **Définition de l'Invariance Terminale** : Les états `CLANDESTINE_CLOSED` et `UNPROTECTED` retournent strictement **`False`** sur leur propriété `.is_terminal`. Cela force l'`ExecutionTracker` à maintenir ces robots vivants en mémoire RAM pour permettre l'intervention humaine (réarmement) ou la contre-attaque automatisée.

#### 1.3 Spécifications Métier (`core/order_group.py`)
* **Suppression du Verrou Bloquant** : Retirer intégralement le bloc opératoire `if self._is_corrupted: raise CorruptedOrderGroupError(...)` situé en tête des méthodes `notify_order_change` et `notify_trade_change`.
* **Drapeau de Clôture Clandestine (`notify_trade_change`)** : Lever la sentinelle logique interne `self._is_clandestine_closed = True` si et seulement si :
    \[(trade\_receipt.is\_open == \text{False}) \land (self.\_exit\_order\_ids == \emptyset) \land (\text{all child orders} \neq \text{OrderState.FILLED})\]
* **Règle Algorithmique Globale de Résolution d'État (`@property def state`)** : Lorsque tous les ordres du groupe (parent, brackets enfants et liste d'exits) ont atteint un état terminal défini (`bracket_orders_terminal and exit_orders_terminal == True`), l'OMS applique l'arbitrage exclusif de l'invariant du ledger :
    * **Si la position physique est nulle** (`self._ledger.position_size == 0.0`) :
        * `if self._is_clandestine_closed` → `return OrderGroupState.CLANDESTINE_CLOSED`
        * `else` → `return OrderGroupState.COMPLETED`
    * **Si la position physique n'est pas nulle** (`self._ledger.position_size != 0.0`) :
        * `return OrderGroupState.UNPROTECTED`

---

### ⏱️ JALON 2 : Intégration Normée de l'état `EXPIRED`

#### 2.1 Objectif Opérationnel
Intégrer les fins de validité temporelle des ordres (*Day limits* boursières ou ordres d'urgence *IOC* non appariés) comme des événements nominaux du cycle de vie du framework Aegis.

#### 2.2 Spécifications d'Infrastructure (`core/models/enums.py`)
* **Invariance Terminale de l'Ordre** : Étendre la propriété calculée `@property def is_terminal` de l'énumération `OrderState` pour y include explicitement `OrderState.EXPIRED`. 
    ```python
    return self in {OrderState.CANCELED, OrderState.FILLED, OrderState.REJECTED, OrderState.EXPIRED}
    ```
    Cela permet aux boucles d'agrégation d'Aegis de comptabiliser un ordre expiré comme un objet mort qui ne reviendra jamais en arrière.

#### 2.3 Spécifications Métier (`core/order_group.py`)
* **Ajustement des Filtres de Rejet** : Retirer `OrderState.EXPIRED` du set immuable de classe `_UNSUPPORTED_STATES`.
* **Grille de Priorité Réseau** : Insérer l'association `OrderState.EXPIRED: 2` au sein de la table de hachage `_STATE_WEIGHTS`. Attribuer le poids maximal `2` garantit le rejet instantané et silencieux de tout paquet obsolète ou inversé arrivant en retard sur le réseau (ex : doublon tardif d'un statut `PENDING`).
* **Résolution à Plat de l'Ordre Parent** : Au sein de la propriété `@property def state`, insérer l'interception de l'expiration de l'ordre d'entrée dans la branche physique à plat :
    ```python
    if is_flat:
        if parent_state == OrderState.EXPIRED:
            return OrderGroupState.CANCELED
    ```
* **Connexion Cumulative avec le Jalon 1** : Si un ordre enfant de protection (Stop-Loss/Take-Profit) ou un ordre de liquidation d'urgence expire en cours de session alors que la position sur le ledger est ouverte, le carnet s'immobilise. L'invariant physique du Jalon 1 interroge le ledger : la taille de position n'étant pas à plat, le groupe bascule automatiquement vers l'état global de danger **`OrderGroupState.UNPROTECTED`**, préparant le terrain pour l'autodéfense.

### 🌊 JALON 3 : Tunnel des Remplissages Incrémentaux (`PARTIALLY_FILLED`)

#### 3.1 Objectif Opérationnel
Migrer le moteur d'exécution d'Aegis vers un modèle continu capable d'encaisser l'asynchronisme et le fractionnement microscopique de la liquidité (remplissages partiels multiples d'un même ordre), tout en s'immunisant contre la latence des vagues de messages grâce à un mécanisme de saturation d'urgence.

#### 3.2 Spécifications Réseau (`core/order_group.py`)
* **Désactivation des Verrous** : Retirer `OrderState.PARTIALLY_FILLED` du set `_UNSUPPORTED_STATES`.
* **Ajustement de la Priorité Réseau** : Insérer `OrderState.PARTIALLY_FILLED: 1` dans `_STATE_WEIGHTS`.
* **Théorème de Perméabilité Sélective (`notify_order_change`)** : Modifier le filtre d'interception réseau précoce. Deux paquets consécutifs de même poids doivent être rejetés, **sauf** si l'état stocké en mémoire et l'état entrant du ticket sont tous les deux `PARTIALLY_FILLED` :
    ```python
    if incoming_weight <= current_weight:
        if not (current_state == OrderState.PARTIALLY_FILLED and order_receipt.state == OrderState.PARTIALLY_FILLED):
            return  # Rejet strict des régressions d'états logiques et des doublons terminaux
    ```

#### 3.3 Mécanique Comptable et Sécurité Volumétrique ("Maxer")
* **Registre des Volumes Exécutés** : Instancier au sein du constructeur `__init__` un dictionnaire privé de traçabilité volumétrique cumulée : `self._executed_quantities: dict[str, Decimal] = {}`, initialisé à `0.0` pour chaque ordre du groupe.
* **Protocole de Saturation au statut `FILLED`** : Si l'état entrant d'un ticket est `FILLED`, pour neutraliser le risque de double imputation comptable lié à des messages partiels fragmentés arrivant en retard, le système calcule le solde manquant exact, sature le compteur au maximum nominal de la commande d'origine, et transmet uniquement ce solde isolé au ledger :
    ```python
    if order_receipt.state == OrderState.FILLED:
        qty_to_clear = order.quantity - self._executed_quantities.get(order_id, Decimal('0.0'))
        self._executed_quantities[order_id] = order.quantity
        self._order_states[order_id] = OrderState.FILLED
        if qty_to_clear > 0:
            self._ledger.update_exposure(side=order.side, quantity=qty_to_clear, price=order_receipt.average_execution_price)
        return
    ```
* **Traitement des Vagues Partielles** : Si l'état entrant est `PARTIALLY_FILLED`, le compteur accumule la quantité brute transmise par le tick d'infrastructure. Puisque `OrderReceipt.executed_quantity` représente par spécification le delta isolé du tick actuel, cette quantité différentielle est envoyée directement au ledger via `self._ledger.update_exposure()`.
* **Refactoring de `is_over_hedged()`** : Modifier la boucle d'accumulation de la protection. Le volume en vol ne doit plus lire la propriété statique `.quantity` de l'ordre initial, mais soustraire dynamiquement le volume cumulé déjà exécuté :
    \[\text{Volume En Vol (Leaves)} = \text{Order.quantity} - \text{self.\_executed\_quantities[order\_id]}\]

#### 3.4 Spécifications des Garde-fous et de l'Exit
* **Transition Transitoire `CLOSING`** : Les sentinelles logiques `children_triggered` et `exit_triggered` de la propriété `@property def state` s'activent de manière physique dès que le compteur d'exécutions d'une protection enfant ou d'un ordre de sortie décolle de zéro : `self._executed_quantities.get(order_id, 0) > 0`. Le groupe bascule instantanément à l'état global transitoire `OrderGroupState.CLOSING`.
* **Maintien Strict des Garde-fous Existants** : Les fonctions `is_cancelable()` et `is_closable()` restent structurellement **inchangées**. La fonction `is_closable()` exclut strictement l'état `CLOSING`. Dès qu'une protection enfant commence à être grignotée par le marché, l'OMS s'interdit d'émettre un ordre d'urgence concurrent, laissant le canal nominal déjà ouvert dérouler sa liquidation pour éviter de dupliquer les flux de vente.
* **Redimensionnement Dynamique de l'Exit (`terminate_execution`)** : Si une coupure urgente est demandée par l'opérateur sur une position partielle (groupe toujours à l'état `ACTIVE`), la taille de l'ordre `-XT` est écrasée pour s'aligner dynamiquement sur la valeur absolue de la position physique instantanée du ledger :
    ```python
    quantity = abs(order_group.ledger.position_size)
    ```
* **Acceptation des Collisions Réseau et Arbitrage Final** : Si un second remplissage partiel de l'ordre parent survient après ou pendant l'envoi de l'ordre de sortie ajusté, le système laisse couler le flux de manière gracieuse. Une fois le carnet immobilisé, si la position du ledger n'est pas nulle, la règle de l'invariant du Jalon 1 fait basculer le groupe en **`OrderGroupState.UNPROTECTED`**.

---

## 🛡️ JALON 4 : Passer de la Détection à l'Autodéfense (Boucle Active)

#### 4.1 Objectif Opérationnel
Automatiser la réaction face au risque de désalignement d'exposition en direct. Si le groupe bascule à l'état de danger physique **`OrderGroupState.UNPROTECTED`** (Jalon 1, causé par les Jalons 2 ou 3), l'OMS doit neutraliser le risque de manière autonome en propulsant une salve de coupure d'urgence de type `MARKET` pour tout remettre à plat à zéro.

#### 4.2 Spécifications d'Unification du Carnet d'Ordres (`core/order_group.py`)
Pour éviter l'apparition d'ordres "fantômes" inconnus de la machine à états et capitaliser à 100 % sur la robustesse du filtrage réseau et de la mécanique volumétrique du Jalon 3, l'ordre d'autodéfense est sémantiquement fusionné avec les ordres de sortie forcée.
* **Lignée de Sortie Unifiée** : La variable unique `self._exit_order_id` est supprimée et remplacée dans le constructeur par une liste de traçabilité des sorties successives : `self._exit_order_ids: list[str] = []`. Tout ordre d'urgence (manuel ou automatique) y est injecté de manière unifiée (`self._exit_order_ids.append(client_order_id)`).
* **Le Verrouillage d'Anti-Emballement Global (`any`)** : Pour interdire l'envoi d'ordres d'urgence en double pendant qu'un ordre précédent navigue sur le réseau ou subit des remplissages fractionnés, le groupe applique une vérification de fin de course sur l'ensemble de la lignée de sortie :
    ```python
    def has_exit_in_flight(self) -> bool:
        return any(
            not self._order_states[exit_id].is_terminal
            for exit_id in self._exit_order_ids
        )
    ```

#### 4.3 Logique de la Boucle Active et Période de Grâce (`core/execution_tracker.py`)
* **Registre de Persistance des Ticks** : L'`ExecutionTracker` instancie au sein de son constructeur un dictionnaire de suivi des alertes consécutives pour matérialiser le socle de la future période de grâce : `self._over_hedge_ticks: dict[str, int] = {}`.
* **Point d'Interception du Tracker (`process_broker_event`)** : À la fin de la méthode principale de traitement des flux, immédiatement après l'exécution de `notify_order_change` ou `notify_trade_change`, et **avant** la validation de la purge `order_group.is_terminal()`, exécuter le contrôle de sécurité :
    * Si `order_group.is_over_hedged()` retourne `True` (Leaves calculés au Jalon 3) :
        * Incrémenter le compteur de persistance de l'anomalie : `self._over_hedge_ticks[group_id] += 1`.
        * Si le compteur franchit le seuil (`1` pour ce jalon afin d'exécuter une coupure immédiate, et structurellement prêt à accueillir une période de grâce de `N` ticks dans le futur) **ET** que la fonction unifiée `order_group.has_exit_in_flight()` est `False` :
            1. **Calcul de Taille Ajustée** : `quantity = abs(order_group.ledger.position_size)`.
            2. **Orientation Vectorielle** : Orienter l'ordre à l'opposé de la polarité du ledger (un `SELL` si la position est longue, un `BUY` si la position est courte).
            3. **Propulsion Électrique** : Générer l'ordre d'urgence unifié de type `OrderType.MARKET`, l'enregistrer dans la liste `self._exit_order_ids` (Jalon 4) et le propulser via `broker_adapter.close_position(exit_order)`.
            4. **Fail-Fast** : Lever l'exception de domaine dédiée `OverHedgedExposureError` dans sa docstring pour figer transitoirement la stratégie sous-jacente du robot.
    * **La Résolution de Guérison Finale** : Dès que l'ordre de colmatage de la boucle active est exécuté à 100 % par le courtier, la position du ledger retombe à `0.0`. Au tick suivant, l'algorithme de l'invariant du ledger du Jalon 1 réévalue le carnet immobile et fait basculer le groupe de l'état `UNPROTECTED` vers l'état de sécurité finale **`OrderGroupState.COMPLETED`**, fermant le cycle proprement.

## 4. CONCLUSIONS AND NEXT STEPS

### 4.1 Conclusions
Cette refonte de l'OMS Aegis apporte une réponse rigoureuse, mathématique et pragmatique aux failles inhérentes aux flux asynchrones de microstructure de marché. 

En dissociant les flux passifs d'écoute (qui restent ouverts à 100 % au Jalon 1 grâce au démantèlement du blocage aveugle historique) des flux actifs de décision, et en s'appuyant exclusivement sur l'invariant physique du ledger, le système acquiert une stabilité totale. L'introduction du dictionnaire `self._executed_quantities` et du protocole "Maxer" (Jalon 3) immunise l'infrastructure comptable contre la latence réseau. Enfin, l'unification par liste et boucle globale `any` de la lignée de sortie (Jalon 4) permet à l'autodéfense active de liquider les anomalies de manière séquentielle et sécurisée, sans aucun risque d'emballement d'ordres ou de duplication de flux.

### 4.2 Prochaines Étapes Opérationnelles
La feuille de route d'ingénierie cumulative étant définitivement validée, les prochaines étapes de notre session de travail sont les suivantes :
1. **Chantier du Jalon 1 (Assainissement)** : Ouvrir le fichier `core/models/enums.py` pour y injecter les statuts `CLANDESTINE_CLOSED` et `UNPROTECTED`, nettoyer `core/order_group.py` en supprimant la levée précoce de `CorruptedOrderGroupError`, et implémenter la règle de filtrage asymétrique par le ledger au sein de `@property def state`.
2. **Campagne de Tests Unitaires du Jalon 1** : Rédiger les scripts pytest simulant une injection de reçus réseau sur un groupe en anomalie et valider que le ledger enregistre les données de position de manière perméable sans aucun crash logiciel.

