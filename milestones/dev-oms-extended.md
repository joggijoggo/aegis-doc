# 🧼 JALON 1 : ASSAINISSEMENT ET PERMÉABILITÉ DES FLUX PASSIFS

## 🛑 Sous-Jalon 1.1 : Introduction de `CLANDESTINE_CLOSED` (Coexistence Pattern)
* **Objectif** : Isoler l'anomalie de la position fermée en externe par le courtier alors que le compte est à plat. L'ancien mécanisme d'exception et le drapeau `_is_corrupted` sont conservés en parallèle pour protéger la suite de tests existante.
* **Code (`core/models/enums.py`)** : Ajouter `CLANDESTINE_CLOSED = 'CLANDESTINE_CLOSED'` à `OrderGroupState`. Configurer sa propriété `@property def is_terminal` pour renvoyer `False`.
* **Code (`core/order_group.py`)** : Initialiser `self._is_clandestine_closed: bool = False` dans `__init__`. Dans `notify_trade_change`, lors d'une fermeture clandestine, ajouter `self._is_clandestine_closed = True` à côté de l'affectation historique `self._is_corrupted = True`.
* **Code (`core/order_group.py` - `@property def state`)** : Dans le bloc `if bracket_orders_terminal and exit_order_terminal:`, insérer :
  ```python
  if self._ledger.position_size == Decimal('0.0') and self._is_clandestine_closed:
      return OrderGroupState.CLANDESTINE_CLOSED
  ```
* **Tests** : Rédiger `test_clandestine_close_state_resolution` pour injecter un `TradeReceipt(is_open=False)` sur un groupe actif et valider le retour de `CLANDESTINE_CLOSED`.
* **💾 Commit Atomique 1.1** : `feat(oms): introduce and wire CLANDESTINE_CLOSED state for external flat closures`

## 🛑 Sous-Jalon 1.2 : Introduction de `UNPROTECTED` (Coexistence Pattern)
* **Objectif** : Isoler l'anomalie du carnet d'ordres immobile alors que la position physique est toujours ouverte au marché.
* **Code (`core/models/enums.py`)** : Ajouter `UNPROTECTED = 'UNPROTECTED'` à `OrderGroupState` (`is_terminal = False`).
* **Code (`core/order_group.py` - `@property def state`)** : Au sein du bloc de résolution final du carnet immobile, insérer l'arbitrage direct du ledger :
  ```python
  if self._ledger.position_size != Decimal('0.0'):
      return OrderGroupState.UNPROTECTED
  ```
* **Tests** : Rédiger `test_unprotected_state_resolution` en simulant un carnet d'ordres mort avec une taille de position résiduelle du ledger non nulle et valider le retour de `UNPROTECTED`.
* **💾 Commit Atomique 1.2** : `feat(oms): introduce and wire UNPROTECTED state for unhedged terminal positions`

## 🧼 Sous-Jalon 1.3 : Libération de la Tuyauterie Passive (`notify`)
* **Objectif** : Démantèlement du disjoncteur d'exception aveugle historique. Les états fins qualifiant le risque à 100 %, l'OMS ouvre ses vannes de streaming de données en direct.
* **Code (`core/order_group.py`)** : Supprimer définitivement les blocs `if self._is_corrupted: raise CorruptedOrderGroupError(...)` en tête des méthodes `notify_order_change` et `notify_trade_change`. Nettoyer les affectations internes `self._is_corrupted = True` et son initialisation dans `__init__`.
* **Tests** : Supprimer ou réécrire le test historique s'assurant qu'une exception était levée lors d'une notification sur groupe contaminé.
* **💾 Commit Atomique 1.3** : `refactor(oms): remove hard-coded corrupted exceptions and internal flags from passive paths`

## 🗑️ Sous-Jalon 1.4 : Purge Sémantique de l'Ancien Monde
* **Objectif** : Nettoyer intégralement les structures de données en éliminant les labels et exceptions morts.
* **Code (`core/models/enums.py` & `core/exceptions.py`)** : Retirer `CORRUPTED` de `OrderGroupState` et supprimer la classe `CorruptedOrderGroupError`.
* **Tests** : Purger les dernières assertions ou imports résiduels de la suite globale.
* **💾 Commit Atomique 1.4** : `cleanup(oms): permanently remove historical CORRUPTED state and exception classes`

# ⏱️ JALON 2 : INTÉGRATION NORMÉE DE L'ÉTAT `EXPIRED`

## 🛑 Sous-Jalon 2.1 : Évolution de l'Invariance et du Poids Réseau de l'Ordre
* **Objectif** : Donner à l'état `EXPIRED` sa nature terminale et sa priorité maximale dans le filtrage réseau.
* **Code (`core/models/enums.py`)** : Ajouter `OrderState.EXPIRED` au sein du set de la propriété `@property def is_terminal` de l'énumération `OrderState`.
* **Code (`core/order_group.py`)** : Retirer `OrderState.EXPIRED` de `_UNSUPPORTED_STATES`. Insérer l'association `OrderState.EXPIRED: 2` dans `_STATE_WEIGHTS`.
* **Tests** : Rédiger `test_expired_order_network_weight` prouvant qu'un reçu d'ordre `EXPIRED` fige le statut et qu'un paquet `PENDING` tardif arrivant sur le même identifiant est rejeté.
* **💾 Commit Atomique 2.1** : `feat(enum): promote OrderState.EXPIRED to terminal status with max network weight`

## 🔄 Sous-Jalon 2.2 : Résolution de l'Expiration de l'Entrée Parent à Plat
* **Objectif** : Clore proprement le cycle nominal si l'ordre d'entrée parent expire en fin de validité temporelle sans engager de capital.
* **Code (`core/order_group.py` - `@property def state`)** : Dans la branche `if is_flat:`, ajouter :
  ```python
  if parent_state == OrderState.EXPIRED:
      return OrderGroupState.CANCELED
  ```
* **Tests** : Rédiger `test_expired_parent_order_resolution` validant qu'une expiration de l'entrée parent à plat résout le groupe en `CANCELED`.
* **💾 Commit Atomique 2.2** : `feat(oms): resolve expired parent entrance orders into global CANCELED state`

# 🌊 JALON 3 : TUNNEL DES REMPLISSAGES INCRÉMENTAUX (`PARTIALLY_FILLED`)

## 🛑 Sous-Jalon 3.1 : Perméabilité Réseau de l'État Partiel
* **Objectif** : Autoriser le framework à absorber des vagues de liquidité successives tout en bloquant les régressions logiques.
* **Code (`core/order_group.py`)** : Retirer `OrderState.PARTIALLY_FILLED` de `_UNSUPPORTED_STATES` et l'ajouter à `_STATE_WEIGHTS` avec le poids `1`.
* **Code (`core/order_group.py` - `notify_order_change`)** : Modifier le filtre d'interception réseau :
  ```python
  if incoming_weight <= current_weight:
      if not (current_state == OrderState.PARTIALLY_FILLED and order_receipt.state == OrderState.PARTIALLY_FILLED):
          return
  ```
* **Tests** : Rédiger `test_partial_execution_permeability` validant que trois reçus `PARTIALLY_FILLED` consécutifs traversent le filtre, mais qu'un `PENDING` tardif subséquent est bloqué.
* **💾 Commit Atomique 3.1** : `feat(network): implement selective permeability filter for partial executions`

## 🛑 Sous-Jalon 3.2 : Registre d'Accumulation et Protocole "Maxer"
* **Objectif** : Déployer le suivi volumétrique cumulé et la barrière de saturation face à la latence réseau des messages partiels fragmentés tardifs.
* **Code (`core/order_group.py`)** : Initialiser `self._executed_quantities: dict[str, Decimal] = {}` dans `__init__`. Dans `notify_order_change`, structurer la mise à jour :
  ```python
  if order_receipt.state == OrderState.FILLED:
      qty_to_clear = order.quantity - self._executed_quantities.get(order_id, Decimal('0.0'))
      self._executed_quantities[order_id] = order.quantity
      self._order_states[order_id] = OrderState.FILLED
      if qty_to_clear > 0:
          self._ledger.update_exposure(side=order.side, quantity=qty_to_clear, price=order_receipt.average_execution_price)
      return
  ```
  Si `order_receipt.state == OrderState.PARTIALLY_FILLED`, accumuler la quantité brute du tick dans le compteur et la transmettre directement au ledger. Isoler l'accès en jetant tout partiel entrant si le statut en mémoire est déjà `FILLED`.
* **Tests** : Rédiger `test_filled_volume_saturation_protection` en injectant un partiel, puis le paquet terminal `FILLED`, puis un partiel tardif arrivé en retard (vérifier qu'il est ignoré à 100 %).
* **💾 Commit Atomique 3.2** : `feat(oms): implement internal execution ledger register with full saturation protocol`

## 🛑 Sous-Jalon 3.3 : Refactoring Volumétrique Flottant et Dimensionnement de l'Exit
* **Objectif** : Convertir la détection des volumes en valeurs physiques dynamiques et calibrer la taille de l'ordre d'urgence `-XT` lors d'une liquidation sur entrée partielle.
* **Code (`core/order_group.py`)** : Dans `is_over_hedged()`, remplacer la quantité statique par le calcul du volume résiduel réel actif : `ordre.quantity - self._executed_quantities[order_id]`. Configurer `children_triggered` et `exit_triggered` pour s'activer dès que `self._executed_quantities.get(order_id, 0) > 0` (transition immédiate vers l'état `CLOSING`).
* **Code (`core/execution_tracker.py`)** : Dans `terminate_execution`, écraser la taille de l'ordre d'urgence pour s'aligner sur le ledger : `quantity = abs(order_group.ledger.position_size)`.
* **Tests** : Rédiger `test_dynamic_exit_sizing_on_partial_exposure` simulant une entrée à 3 lots sur 10. Forcer l'exit et valider que l'ordre `-XT` généré possède une taille exacte de 3 lots.
* **💾 Commit Atomique 3.3** : `refactor(oms): transition safety volume calculations and exit sizing to dynamic floating values`


# 🛡️ JALON 4 : PASSER DE LA DÉTECTION À L'AUTODÉFENSE (BOUCLE ACTIVE)

## 🛑 Sous-Jalon 4.1 : Unification de la Lignée de Sortie et Coupe-Circuit
* **Objectif** : Unifier le traçage des out-flows d'urgence au sein du groupe et implémenter le verrouillage d'anti-emballement global par bouclage complet.
* **Code (`core/order_group.py`)** : Remplacer `self._exit_order_id = None` par `self._exit_order_ids: list[str] = []` dans `__init__`. Implémenter la méthode unifiée :
  ```python
  def has_exit_in_flight(self) -> bool:
      return any(not self._order_states[exit_id].is_terminal for exit_id in self._exit_order_ids)
  ```
  Adapter `is_closable()` pour rejeter si `has_exit_in_flight()` est vrai, et mettre à jour `@property def state` pour vérifier l'immobilisation via `all(self._order_states[exit_id].is_terminal for exit_id in self._exit_order_ids)`.
* **Tests** : Rédiger `test_exit_lineage_and_anti_flooding_lock` validant que tant qu'au moins un ordre de sortie de la liste n'est pas terminal, le verrouillage est actif et bloque toute nouvelle commande concurrentielle.
* **💾 Commit Atomique 4.1** : `feat(oms): unify exit tracking with continuous out-flow registry and global anti-flooding lock`

## 🛑 Sous-Jalon 4.2 : Déploiement de la Boucle Active d'Autodéfense
* **Objectif** : Intégrer l'intercepteur d'action autonome au sein de la boucle événementielle du tracker d'exécution.
* **Code (`core/exceptions.py`)** : Déclarer l'exception de domaine `class OverHedgedExposureError(Exception):` avec une docstring brute descriptive.
* **Code (`core/execution_tracker.py`)** : Initialiser `self._over_hedge_ticks: dict[str, int] = {}` dans `__init__`. À la toute fin de `process_broker_event`, insérer la boucle active :
  ```python
  if order_group.is_over_hedged():
      self._over_hedge_ticks[group_id] = self._over_hedge_ticks.get(group_id, 0) + 1
      if self._over_hedge_ticks[group_id] >= 1 and not order_group.has_exit_in_flight():
          quantity = abs(order_group.ledger.position_size)
          side = OrderSide.SELL if order_group.ledger.position_size > 0 else OrderSide.BUY
          exit_order = Order(client_order_id=f"{order_group.parent_id}-XT-{len(order_group._exit_order_ids) + 1}", type=OrderType.MARKET, side=side, quantity=quantity, ...)
          order_group.attach_exit_order(exit_order)
          self._order_id_to_bot_id[exit_order.client_order_id] = bot_id
          broker_adapter.close_position(exit_order)
          raise OverHedgedExposureError(f"Critical exposure misalignment detected on bot {bot_id}.")
  else:
      self._over_hedge_ticks[group_id] = 0
  ```
* **Tests** : Rédiger le test d'intégration global `test_autonomous_self_defense_execution_and_healing`. Provoquer l'expiration d'un enfant sur position ouverte. Valider que le groupe passe en `UNPROTECTED`, que le tracker intercepte l'alerte, propulse l'ordre `MARKET` ajusté, lève l'exception de domaine, et qu'à la réception du remplissage final de cet exit, le ledger retombe à zéro et le groupe se ferme proprement en `COMPLETED`.
* **💾 Commit Atomique 4.2** : `feat(oms): deploy autonomous active self-defense loop with dynamic market counter-attacks`


