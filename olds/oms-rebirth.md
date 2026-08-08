# Rapport Architectural Majeur : La Gestion d'Ordres Orientée Données (Partie 1)

**Framework :** Aegis  
**Module :** `core/execution_engine` & `core/order_management`  
**Approche :** Architecture Orientée Données (Data-Driven Ledger)  
**Statut :** Spécifications de refactorisation (Partie 1/3)

---

## 1. Introduction

### Contexte
Dans le cadre du développement du framework de trading algorithmique **Aegis**, l’objectif principal est de concevoir un système **totalement agnostique de l'infrastructure externe**. Aegis doit être capable de faire tourner la même logique de stratégie (Core) qu'elle soit connectée à un moteur de simulation historique discret (comme la bibliothèque `Backtrader`) ou à une API de courtage en direct et en temps réel (comme l'API REST/WebSockets d'IG Markets). 

Pour garantir une intégrité comptable et financière absolue, le cœur d'Aegis s'est d'abord doté d'une structure rigide (`OrderGroup`) basée sur des matrices de transition de processus complexes (`_PARENT_MATRIX` et `_CHILD_MATRIX`). Ce modèle fonctionnait sur un principe de *Fail-Fast* : rejeter ou faire crasher le système à la moindre anomalie microstructurelle pour protéger le capital.

### Motivation
La viabilité d'un framework de trading repose sur sa capacité à éliminer le *sur-ajustement d'infrastructure*. Si le code métier doit être modifié pour s'adapter aux excentricités de Backtrader ou d'une API Live, le framework perd sa valeur agnostique. Commencer par un design ultra-strict était la meilleure décision possible : cela a permis de cartographier avec une précision chirurgicale toutes les anomalies et les excentricités microstructurelles de l'asynchronisme. La motivation de cette étude est de faire mûrir ce modèle de sécurité vers un système de niveau production, capable d'encaisser les réalités transactionnelles des marchés financiers sans générer de crashs injustifiés.

### Problématique Complexe
Le passage à la production et l'exécution d’une clôture de position initiée par le robot (`is_exit()`) ont mis en lumière trois incohérences architecturales profondes :

1. **La collision des boucles de contrôle (Inversion de Contrôle)** : Le modèle initial d'Aegis gérait le temps via une boucle fermée synchrone (`while True`) tirant les données via `next(market_feed)`. En mode Backtest, Backtrader possède sa propre boucle interne (`Cerebro.run()`). Forcer Backtrader à s'insérer dans un itérateur externe, ou bloquer la boucle en Live sur un flux de données, crée un angle mort où les événements du broker ne sont plus dépilés à temps.
2. **Le déphasage temporel des faits accomplis (Race Conditions)** : En faisant avancer le temps *avant* de vider la file d'attente, Backtrader traite et exécute les ordres en un seul pas de temps. Aegis reçoit alors des rafales d'événements transitoires déphasées (ex: un ordre d'entrée qui passe instantanément par `Created` ➔ `Submitted` ➔ `Accepted` ➔ `FILLED`). La machine d'état d'Aegis, trop rigide, interprète ces notifications d'infrastructure comme des retours en arrière ou des corruptions de flux (`CorruptedOrderGroupError`).
3. **L'asymétrie microstructurelle de la clôture** : L' `OrderGroup` d'Aegis a été conçu comme un système fermé et immuable à l'initialisation (1 ordre Parent d'entrée + N ordres Enfants de protection SL/TP). Or, lorsqu'Aegis demande une clôture de position, l'infrastructure (Backtrader ou une API Live comme celle d'IG Markets via son endpoint `DELETE /positions/otc`) génère de manière autonome un **nouvel ordre sauvage inverse** (un ordre `SELL` pour fermer un `LONG`). Cet ordre imprévu possède un identifiant inconnu d'Aegis, déclenchant une `UntrackedOrderException`.

---

## 2. Analyse Approfondie des Options Architecturales Évaluées

Quatre approches majeures ont été passées au crible pour résoudre le problème de l'ordre de clôture imprévu sans détruire les invariants comptables du Core.

* **Option A : Fusionner l'Exit dans la matrice Parent existante**  
  *Concept :* Traiter l'ordre d'Exit comme une extension de l'autorité du Parent.  
  *Rejet :* Risque élevé de collision d'états concurrents. L'ordre d'entrée d'origine étant toujours stocké à l'état `FILLED`, soumettre les notifications d'un second ordre (l'Exit) dans la même matrice provoquerait une corruption immédiate du cycle de vie global, le système ne pouvant pas arbitrer deux flux d'états simultanés pour un seul rôle.
* **Option B : Le pattern Coordinateur (Liaison par le Tracker)**  
  *Concept :* Envisager l'Exit comme un groupe d'ordres distinct. Le `ExecutionTracker` gère une structure intermédiaire `ExecutionLifecycle` reliant le Groupe d'Entrée (Parent, SL, TP) et le Groupe de Sortie (Exit).  
  *Rejet :* Solution rejetée car elle déplace la complexité structurelle vers le Tracker. Elle impose la création d'une plomberie asymétrique en mémoire, multiplie les entités pour un seul trade et surcharge le superviseur de responsabilités de synchronisation fine qui incombent logiquement au domaine de la transaction.
* **Option C : Le Jeton de Clôture dans l'Adaptateur (Le Mensonge Architectural)**  
  *Concept :* Masquer complètement l'existence de l'ordre d'Exit au Core. L'adaptateur intercepte l'ordre sauvage de l'infrastructure, l'absorbe, et simule pour le Core une annulation nominale des protections.  
  *Rejet :* Approche jugée trop dangereuse pour le mode Live. Si l'ordre d'Exit est rejeté ou partiellement exécuté par le courtier en direct, l'adaptateur se retrouve coincé dans un état fictif, rendant le Core d'Aegis totalement aveugle face au risque financier réel.
* **Option D : L'approche Professionnelle de la Réconciliation de Données (Sélectionnée)**  
  *Concept :* Aligner Aegis sur les standards de l'industrie (OMS institutionnels et moteurs comme QuantConnect). Le principe directeur est : **"Le broker a toujours le dernier mot"**. L'architecture abandonne la validation de processus rigide au profit d'une **Architecture Orientée Données (Data-Driven)**. Le système ne cherche plus à valider si l'histoire s'est déroulée dans le bon ordre : il enregistre les données, comptabilise les volumes, tolère les zones grises transitoires et calcule son état en fonction de la réalité du terrain.

# Rapport Architectural Majeur : La Gestion d'Ordres Orientée Données (Partie 2)

**Framework :** Aegis  
**Module :** `core/execution_engine` & `core/order_management`  
**Spécifications :** États, Microstructure et Cas Complexes (Partie 2/3)

---

## 3. Piliers de l'Architecture Orientée Données

Pour éliminer définitivement les conflits temporels, la réécriture repose sur la séparation stricte de l'intention administrative (l'Ordre) et de la réalité comptable (le Clearing).

### A. L'Ordre est Autonome et Linéaire
L' `OrderGroup` n'est plus une machine d'état globale et bloquante. Il devient un **registre comptable ouvert** (un dossier de suivi). Chaque ordre injecté dans ce registre possède sa propre machine d'état unitaire et isolée, totalement indépendante des autres tickets.

Voici le cycle de vie purement linéaire de chaque ordre individuel :

```text
       [ Ordre Soumis ]
              │
              ▼
         ( PENDING )
              │
      ┌───────┴────────────────────────┐
      ▼                                ▼
 ( EXPIRED / REJECTED )       ( PARTIALLY_FILLED ) ◄───┐ (Nouvelle exécution
                                       │               │  partielle)
                                       ▼               │
                                  ( FILLED ) ──────────┘
```

Lorsqu'un ordre reçoit une mise à jour via `notify_order_change`, il modifie son statut local au sein du dictionnaire sans jamais pouvoir faire crasher le thread principal du framework.

### B. L'État de Groupe Calculé (*Computed State*)
Le groupe d'ordres ne stocke plus d'état fétiche dans une variable globale. L'état du groupe est une **propriété calculée dynamiquement** à l'instant T par une fonction qui analyse la composition du registre et de la position réelle :

```python
@property
def state(self) -> OrderGroupState:
    if self.clearing.position_size == 0 and all(o.is_terminal for o in self.orders):
        return OrderGroupState.COMPLETED
    if any(o.is_exit_role and not o.is_terminal for o in self.orders):
        return OrderGroupState.CLOSING
    if self.entry_order.status == OrderState.FILLED and self.clearing.position_size > 0:
        return OrderGroupState.ACTIVE
    return OrderGroupState.PENDING
```

### C. Le Module `Clearing` (Gestion des Volumes)
Chaque groupe intègre un sous-composant `Clearing` exclusivement dédié à la comptabilité des lots (`notify_trade_change`). Il maintient la variable `position_size`. C'est ce composant qui absorbe naturellement les exécutions fragmentées : tant que le cumul des lots des ordres partiels n'a pas ramené l'exposition à zéro, le clearing maintient le trade ouvert, peu importe le désordre des tickets reçus.

---

## 4. Nomenclature Complète des États

### A. États Unitaires des Ordres (`OrderState`)
Chaque ticket possède un état individuel qui progresse uniquement vers le haut en termes de poids mathématique :

* **`PENDING` (Poids 1)** : L'ordre a été forgé par le Core et transmis à l'infrastructure.
* **`PARTIALLY_FILLED` (Poids 2)** : L'ordre a été partiellement apparié sur le marché.
* **`FILLED` (Poids 3 - Terminal)** : L'ordre a été exécuté à 100 % de son volume initial.
* **`CANCELED` (Poids 3 - Terminal)** : L'ordre a été retiré du carnet par le framework ou le broker.
* **`REJECTED` (Poids 3 - Terminal)** : Le broker a refusé l'ordre immédiatement (manque de marge, etc.).
* **`EXPIRED` (Poids 3 - Terminal)** : L'ordre a été effacé par le serveur car sa validité temporelle est écoulée.

### B. États Calculés du Groupe (`OrderGroupState`)
* **`PENDING`** : L'ordre parent d'entrée est à l'état `PENDING`. Pas encore de clearing.
* **`ACTIVE`** : L'ordre parent est `FILLED` / `PARTIALLY_FILLED` et la position réelle est ouverte.
* **`CLOSING`** : Un ordre de sortie `"-XT"` est engagé, OU une protection (SL/TP) a été exécutée.
* **`MUTATED` / `OVER_HEDGED`** : Urgence. Exposition financière inverse ou aberrante suite à une double exécution.
* **`AWAITING_TICKETS`** : Position à zéro au clearing, mais les notifications des tickets d'ordres sont en retard.
* **`COMPLETED` (Terminal)** : Position à `0` et tous les ordres du registre sont résolus (Poids 3).

---

## 5. Détails Profonds de Résolution des Cas Complexes

### 1. Le Clearing en Avance sur l'Ordre
Le module `Clearing` passe instantanément à `0`. La propriété `@property state` constate la position nulle mais voit qu'un ticket (`"-SL"`) est toujours enregistré `ACTIVE`. Au lieu de crasher, elle bascule temporairement le groupe en `OrderGroupState.AWAITING_TICKETS`, gelant l'évaluation du bot jusqu'à l'arrivée de l'événement d'ordre.

### 2. Le Clearing en Retard sur l'Ordre
L'ordre d'Exit passe à `FILLED`, mais le clearing affiche toujours une position ouverte. Le calcul dynamique de l'état maintient le groupe en `CLOSING`. Le système attend simplement les cycles de scrutation suivants pour que la position financière réelle se mette à jour sans lever d'erreur.

### 3. Les Ordres arrivant dans le Désordre (Poids Mathématiques)
Une notification `FILLED` arrive avant la confirmation `PENDING`. Le registre applique la règle d'**Idempotence et de Priorité d'État** :
\[\text{NONE (0)} < \text{PENDING (1)} < \text{PARTIALLY\_FILLED (2)} < \text{FILLED / REJECTED / EXPIRED (3)}\]
L'état `FILLED` (poids 3) est stocké immédiatement. Lorsque le `PENDING` tardif (poids 1) arrive, il est rejeté car son poids est inférieur (1 < 3).

### 4. Collision d'Exécution et Double Exécution
L'Exit manuel `"-XT"` et le Stop Loss `"-SL"` s'exécutent simultanément avant annulation réciproque, inversant la position (vous passez Short de -1 lot). Le clearing calcule l'exposition réelle (`position_size = -1`). Le groupe passe en `OVER_HEDGED`. Le tracker intercepte ce flag et déploie instantanément un ordre de rachat d'urgence (*Market BUY de 1 lot*) pour nettoyer le compte.

### 5. Notification Réseau Doublon
Si un message WebSocket `FILLED` (poids 3) arrive deux fois, le registre constate que le poids reçu est égal au poids stocké (3 n'est pas supérieur à 3) et ignore le doublon, sécurisant le Clearing contre un double calcul erroné.

### 6. Le Rejet de l'ordre d'Exit (`REJECTED`)
L'événement `REJECTED` frappe l'ID `"-XT"`. L' `OrderGroup` passe le statut du ticket à `REJECTED` et, constatant que la position est toujours ouverte au clearing, fait **repasser le groupe à l'état `ACTIVE`**. Le framework reste en surveillance nominale, protégé par les SL/TP toujours actifs.

# Rapport Architectural Majeur : La Gestion d'Ordres Orientée Données (Partie 3)

**Framework :** Aegis  
**Module :** `core/execution_engine` & `core/order_management`  
**Structure & Dev-Plan :** Signatures, Réconciliation et Migration (Partie 3/3)

---

## 6. Description Structurelle des Composants

### A. Classe `ClearingLedger`
Gère exclusivement la comptabilité physique des lots et l'exposition réelle du symbole.

*   `symbol: str` : Identifiant de l'instrument.
*   `position_size: float` : Taille physique active (`0.0` pour *Flat*).
*   `average_price: float` : Prix moyen de revient.
*   `realized_pnl: float` : Profit et perte matérialisé.

```python
def update_metrics(self, trade_receipt: TradeReceipt) -> None:
    """Updates the position size, entry price, and PnL based on a verified clearing event."""

def force_reconciliation(self, broker_position_size: float, broker_avg_price: float) -> None:
    """Overwrites internal volume metrics with the absolute truth fetched from the broker snapshot."""
```

### B. Classe `OrderGroup`
Registre comptable hébergeant la collection d'ordres et calculant l'état logique.

*   `group_id: str` / `_parent_id: str`
*   `_orders: dict[str, Order]` / `_order_states: dict[str, OrderState]`
*   `clearing: ClearingLedger`
*   `needs_reconciliation: bool` : Flag d'incohérence ou de time-out réseau.

```python
def __init__(self, group_id: str, parent_order: Order): ...
@property
def state(self) -> OrderGroupState: ...
def register_exit_intention(self, exit_order: Order) -> None: ...
def notify_order_change(self, order_receipt: OrderReceipt) -> None: ...
def notify_trade_change(self, trade_receipt: TradeReceipt) -> None: ...
```

### C. Classe `ExecutionTracker`
Superviseur opérationnel charge du routage et des ordres de sauvetage.

*   `_active_groups: dict[str, OrderGroup]`
*   `_bot_to_group_map: dict[str, str]`
*   `_broker_adapter: BaseBrokerAdapter`

```python
def process_broker_event(self, event: BrokerEvent) -> list[Order]:
    """Intercepts, categorizes, and routes incoming broker updates. Returns corrective orders if needed."""

def terminate_execution(self, bot_id: str) -> None:
    """Orchestrates manual exit. Forges -XT, registers it, and commands infrastructure liquidation."""
```

---

## 7. Le Protocole de Réconciliation (La Ceinture de Sécurité)

### Le Snapshot de Démarrage (`Bootstrap Recovery`)
Au réveil d'Aegis suite à un crash, le tracker applique la réconciliation avant le premier cycle :
1. Le tracker efface toute sa mémoire RAM volatile.
2. Il télécharge le snapshot brut officiel du broker (Positions réelles + Ordres ouverts).
3. La fonction d'usine `OrderGroup.from_broker_snapshot()` reconstruit instantanément les structures en mémoire, synchronise les volumes du clearing et rattache les SL/TP actifs trouvés chez le courtier, réparant le contexte historique.

### Le Time-Out d'Incohérence
Si un message de flux est perdu (clearing bloqué à 1 lot alors que le ticket confirme une exécution complète depuis longtemps), le groupe active le flag `needs_reconciliation = True` après un délai de sécurité de X secondes. Le tracker force un appel API synchrone direct pour écraser le flux d'événements défaillant par la vérité du serveur distant.

---

## 8. Plan de Développement Détaillé (Migration Sécurisée)

```text
 PHASE 1 : Isolement       PHASE 2 : Hybridation    PHASE 3 : Déconnexion    PHASE 4 : Résilience
┌────────────────────┐    ┌─────────────────────┐  ┌─────────────────────┐  ┌───────────────────┐
│ • Créer -XT        │───►│ • Injecter Clearing │──►│ • Supprimer         │──►│ • Ajouter         │
│ • Sortie non-bloq. │    │ • Mode Double État  │  │   les matrices      │  │   la Réconcil.    │
└────────────────────┘    └─────────────────────┘  └─────────────────────┘  └───────────────────┘
```

### Phase 1 : Sécurisation de la Boucle et Injection du Suffixe `"-XT"`
*   **Action 1** : Modifier `BaseMarketFeed` pour rendre `next()` non-bloquant (retourne `None`). Adapter la boucle principale pour continuer à tourner à vide et dépiler les événements du broker via `poll_event()`.
*   **Action 2** : Ajouter la méthode `register_exit_intention(exit_order)` dans `OrderGroup`. Modifier `terminate_execution` dans le tracker pour fabriquer l'objet `Order` suffixé `"-XT"`, appeler l'enregistrement dans le groupe, puis le soumettre à l'adaptateur.
*   **Résultat** : L'ID est connu du Core avant sa notification ; l' `UntrackedOrderException` est résolue.

### Phase 2 : Hybridation et Introduction du `ClearingLedger`
*   **Action 1** : Coder la classe `ClearingLedger` et l'instancier dans `OrderGroup`.
*   **Action 2** : Connecter `notify_trade_change` (ligne 337 actuelle) pour alimenter ce `ClearingLedger`.
*   **Action 3 (Le Pont)** : Conserver vos matrices actuelles. Modifier le début de `notify_order_change` : si l'ID finit par `"-XT"`, mettre à jour son état et forcer `self._state` à `CLOSING`, bypassant l'évaluation matricielle classique pour ce ticket.
*   **Résultat** : Vos tests unitaires initiaux restent au vert, mais la clôture manuelle avec Backtrader fonctionne.

### Phase 3 : Déconnexion des Matrices et Bascule sur l'État Calculé
*   **Action 1** : Écrire la propriété `@property def state` dans `OrderGroup` (calcul basé sur la position et la finalité des tickets à poids 3).
*   **Action 2** : Remplacer l'affectation manuelle de `self._state` par des appels dynamiques à cette propriété partout dans le Core (`Engine` et `Tracker`).
*   **Action 3** : Supprimer définitivement les dictionnaires `_PARENT_MATRIX`, `_CHILD_MATRIX`, et la méthode `_evaluate_eviction_barrier`.
*   **Résultat** : Code nettoyé. Le désordre réseau ou le clearing en avance sont absorbés par le calcul dynamique.

### Phase 4 : Blindage Industriel (Reprise sur Erreur et Doublons)
*   **Action 1** : Implémenter la règle des poids mathématiques dans `notify_order_change` pour rejeter les notifications en retard ou doublons.
*   **Action 2** : Coder l'état calculé `OVER_HEDGED` et la fonction de rachat automatique dans le tracker.
*   **Action 3** : Implémenter la méthode `from_broker_snapshot` pour permettre la réconciliation native au démarrage.

---

## 9. Conclusion
En remplaçant les matrices de transition chronologiques par un **état calculé basé sur les faits comptables**, le code devient considérablement plus court, plus simple à maintenir et mathématiquement immunisé contre les crashs en cascade. Aegis s'appuie sur la seule constante universelle de toutes les infrastructures du monde : les tickets d'ordres et la taille réelle de la position.

