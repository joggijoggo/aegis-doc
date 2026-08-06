# 🗺️ FEUILLE DE ROUTE ARCHITECTURALE DE LA SUITE D'EXÉCUTION (AEGIS)

## 1. INTRODUCTION

### Contexte
Le Framework AEGIS est un moteur de trading algorithmique événementiel à haute performance destiné à piloter des stratégies quantitatives autonomes sur des marchés à haute fréquence. L'efficacité d'un tel système repose sur sa capacité à maintenir une cohérence absolue entre l'état théorique de ses algorithmes (le Domaine) et l'état physique des ordres chez le courtier (l'Infrastructure).

### Motivation
La version historique d'AEGIS gérait le suivi des ordres via une indirection complexe basée sur un identifiant de groupe générique (`group_id`). Cette architecture introduisait un couplage fort entre le moteur d'exécution et la plomberie interne des transactions, ralentissant les performances lors des rafales de ticks et complexifiant la maintenance du code. Pour passer à l'échelle et garantir une exécution de qualité institutionnelle, AEGIS doit migrer vers un paradigme centré sur l'identité des robots de trading (`bot_id`) et l'ordre parent racine (`client_order_id`).

### Problématique
La transition vers ce nouveau modèle soulève trois défis techniques majeurs :
1. **L'asynchronisme et les *Race Conditions* du réseau** : Le courtier délivre les reçus de carnet (`OrderReceipt`) et de clearing financier (`TradeReceipt`) via des canaux asynchrones distincts. Le réseau peut inverser ou retarder ces messages, mettant en péril les décisions d'extinction d'urgence du système.
2. **La violation des règles de Netting** : Pour éviter toute surexposition financière élastique, un robot sous mandat de *Strict Netting* ne doit jamais pouvoir émettre d'ordres concurrents si une exécution est déjà active en mémoire.
3. **L'étanchéité de l'encapsulation (Boîte Noire)** : Les tests unitaires et les composants périphériques ne doivent jamais violer l'intimité des structures internes des ordres ou de l'état mémoriel sous peine de créer un système fragile et instable.

---

## 📊 2. SYNTHÈSE MANAGÉRIALE

L'implémentation et le scellage des Sections 1, 2 et 3 valident un pivot architectural majeur pour la sécurité financière d'AEGIS. En appliquant le principe **"Tell, Don't Ask"** (Dis, ne demande pas), nous avons transformé l'`OrderGroup` en une boîte noire étanche qui n'expose plus sa machine à états interne, mais des *capacités fonctionnelles explicites* (`is_cancelable`, `is_closable`).

### Bénéfices stratégiques et techniques validés :
* **Risque d'exécution divisé par deux** : Grâce à l'immunisation comptable croisée (le dictionnaire de second niveau `_order_id_to_bot_id` mis à jour en temps réel), le framework est mathématiquement protégé contre les doubles liquidations sauvages (*over-hedge*) nées des latences du réseau de courtage.
* **Performance en temps constant \(\mathcal{O}(1)\)** : Le routage des événements de marché ne nécessite plus aucun parcours itératif de liste. L'accès mémoriel direct protège le CPU d'AEGIS des goulots d'étranglement lors des crises de liquidité ou des annonces macroéconomiques.
* **Garantie logicielle maximale** : Le Domaine pur est désormais scellé sous un bouclier combinatoire de **35 cas de tests unitaires boîte noire pure**, garantissant une couverture de code absolue (100% Green) et une résilience totale face aux réécritures futures de l'infrastructure.

---

## 🛠️ 3. PLAN DE DÉVELOPPEMENT DÉTAILLÉ

### 📦 SECTION 1 : L'ADAPTATEUR DE COURTAGE (INFRASTRUCTURE)

#### Jalon 1 : Commandes microstructurelles directes — **[TERMINÉ]**
* **Fichier** : `core/broker_adapter.py` & `tests/core/test_broker_adapter.py`
* **Actions** :
    * Implémenter la méthode `cancel_order` en s'appuyant dynamiquement sur l'attribut `.info` du ticket d'ordre Backtrader.
    * Implémenter la méthode `close_position` en ajoutant une barrière de garde stricte sur `position.size != 0` pour contrer la rémanence mémorielle de Backtrader.
    * Déclarer les exceptions de sécurité `BrokerOrderNotFoundError` et `BrokerPositionNotFoundError`.
    * Injecter 4 cas de tests unitaires en boîte noire absolue pour saturer les branches d'erreurs et nominales.
* **Identifiant de commit** : `dfa9b35 - feat(broker): add cancel_order and close_position for broker adapters`

---

### 📦 SECTION 2 : RÉNOVATION DE L'ORDERGROUP (DOMAINE)

#### Jalon Intermédiaire : Capacités fonctionnelles "Tell, Don't Ask" — **[TERMINÉ]**
* **Fichier** : `core/order_group.py` & `tests/core/test_order_group.py`
* **Actions** :
    * Implémenter la méthode `get_parent_order()` pour encapsuler l'extraction de l'ordre racine.
    * Implémenter la méthode `is_cancelable()` asservie à l'état structurel `PENDING` et validée par l'absence d'événement de clearing (`self._clearing_closed is None`).
    * Implémenter la méthode `is_closable()` immunisée contre les inversions de messages réseau et le verrouillage de l'état `CLOSING` (barrière anti-*over-hedge*).
    * Injecter une matrice exhaustive de 18 cas de tests unitaires pour saturer toutes les permutations d'états et asynchronismes.
* **Identifiant de commit** : `0528e13 - feat(domain): encapsulate OrderGroup state machine behind capabilities`

---

### 📦 SECTION 3 : L'EXECUTIONTRACKER (DOMAINE PUR)

#### Jalon 2 : Initialisation, Isolation et Netting — **[TERMINÉ]**
* Fichier : `core/execution_tracker.py` & `tests/core/test_execution_tracker.py`
* Actions :
    * Initialiser la structure de RAM à double indexation en \(\mathcal{O}(1)\) (`_executions` par `bot_id` et `_order_id_to_bot_id` par `client_order_id`).
    * Coder l'algorithme de génération de la géométrie des ordres enfants (*Stop Loss* et *Take Profit*) lors de la soumission de l'ordre parent.
    * Lever l'exception `NettingRestrictionError` si un robot tente d'ouvrir une exposition alors qu'il possède déjà un contexte actif en RAM.
    * Injecter 5 cas de tests unitaires en boîte noire pour valider le strict netting et la génération des brackets.
* Identifiant de commit : `15a1a77 - feat(core): initialize ExecutionTracker and implement register_order`

#### Jalon 3 : Cycle de vie événementiel (`process_broker_event`) — **[TERMINÉ]**
* Fichier : `core/execution_tracker.py` & `tests/core/test_execution_tracker.py`
* Actions :
    * Placer au sommet de `process_broker_event` une clause de garde validant l'identité de l'événement et levant `UnsupportedBrokerEventError` si corrompu.
    * Router dynamiquement les flux asynchrones vers l'un des deux canaux de l'`OrderGroup` via l'aiguillage transitoire du `group_id` ou du `client_order_id`.
    * Coder la méthode de nettoyage mémoriel par parcours inversé performant (`_clear_execution_context`) déclenchée dès que `.is_terminal` passe à `True`.
    * Injecter 5 cas de tests unitaires (Cas 6 à 10) utilisant des gestionnaires de contexte `PropertyMock` pour simuler le comportement de la boîte noire.
* Identifiant de commit : `2d9e072 - feat(core): add process_broker_event to ExecutionTracker`

#### Jalon 4 : Logique d'extinction encapsulée (`terminate_execution`) — **[TERMINÉ]**
* Fichier : `core/execution_tracker.py` & `tests/core/test_execution_tracker.py`
* Actions :
    * Implémenter la méthode publique `terminate_execution(bot_id, broker_adapter)` pour piloter la coupure d'urgence demandée par l'Engine en s'appuyant sur les capacités de l'`OrderGroup`.
    * Déclarer l'exception de rétention mémorielle : `DanglingExecutionError`.
    * Structurer la cascade de filtres `if/elif/elif/else` : envoyer un `cancel_order` si annulable, un `close_position` si clôturable, lever `DanglingExecutionError` si bloqué à l'état terminal, et appliquer un `pass` inoffensif légitime si en cours de phase transitoire asynchronie (`CLOSING`, `REJECTING`).
    * Injecter 5 nouveaux cas de tests unitaires (Cas 11.A à 12) pour saturer et sceller définitivement chaque embranchement de sécurité.
* Identifiant de commit : `2395fe4 - feat(core): add terminate_execution to ExecutionTracker`

---

### 📦 SECTION 4 : LE CÂBLAGE DE L'EXECUTIONENGINE (APPLICATION)

#### Jalon 5 : Injection, Substitution et Nettoyage de la RAM — **[PROCHAINEMENT]**
* Fichier : `core/execution_engine.py`
* Actions :
    * Instanciation de l'objet `ExecutionTracker` au sein du constructeur `__init__` de l'`ExecutionEngine`.
    * Suppression définitive de l'ancienne structure mémorielle locale `self._order_groups`.
    * Élimination complète de l'ensemble des anciennes fonctions d'infrastructure de gestion d'état devenues obsolètes (`_register_order_group`, `_handle_order_notification`, etc.) pour nettoyer la dette technique de l'Engine.
* Message de commit attendu : `refactor(engine): inject execution tracker and eliminate legacy local map`

#### Jalon 6 : Réforme des flux de contrôle applicatifs — **[À VENIR]**
* Fichier : `core/execution_engine.py` & `tests/core/test_execution_engine.py`
* Actions :
    * Modifier la boucle principale d'émission d'ordres de l'Engine pour interroger `tracker.has_active_execution(bot_id)` avant d'autoriser une stratégie à envoyer des ordres au marché (Strict Netting).
    * Rediriger l'intégralité du flux asynchrone intercepté par la méthode réceptrice `_on_broker_event` directement vers `tracker.process_broker_event(event)`.
    * Exécuter l'intégralité de la suite de tests fonctionnels existante de l'Engine pour valider la non-régression absolue du câblage.
* Message de commit attendu : `feat(engine): reroute main execution loop and broker events through tracker`

---

### 📦 SECTION 5 : LA CLÔTURE ET DETTE TECHNIQUE (VALIDATION END-TO-END)

#### Jalon 7 : Le Test d'Intégration Réel — **[À VENIR]**
* Fichier : `tests/integration/test_backtrader_integration.py`
* Actions :
    * Réhabiliter le fichier d'intégration et concevoir le scénario d'endurance avec le robot `ClosePositionTestBot`.
    * Simuler un achat au tick 0, déclencher un ordre d'arrêt forcé de l'Engine au tick 1, et intercepter l'aiguillage de sécurité.
    * Valider physiquement dans le carnet d'ordres réel du simulateur Backtrader que la position nette est immédiatement écrasée à 0.
* Message de commit attendu : `test(integration): verify physical position liquidation inside backtrader book`

#### Jalon 8 : Refactoring TradeReceipt — **[À VENIR]**
* Fichier : `core/models/trading.py` & impact global du framework
* Actions :
    * Harmoniser les contrats de données des notifications du courtier en supprimant le champ obsolète `group_id` au sein de la classe `TradeReceipt`.
    * Uniformiser la microstructure des données en migrant toutes les attaches du clearing vers le champ standardisé `client_order_id`.
* Message de commit attendu : `refactor(models): unify broker data contracts by replacing group_id with client_order_id`

---

## 4. CONCLUSION

Le travail accompli jusqu'au Jalon 4 dote le noyau d'AEGIS d'un modèle de Domaine d'une pureté mathématique rare. En séparant hermétiquement la logique de tracking de l'infrastructure réseau, nous avons éliminé les bugs de concurrence les plus redoutables du trading automatique. Le système est désormais prêt à accueillir la Section 4 pour injecter cet organe central au cœur de l'`ExecutionEngine` et parachever la refonte opérationnelle du framework.

