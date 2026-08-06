# 🏛️ RAPPORT D'ARCHITECTURE ET PLAN DE ROUTE : MODULE EXECUTION_TRACKER

## 1. Introduction

### Contexte
Le framework AEGIS est conçu comme une plateforme de trading algorithmique événementielle de grade institutionnel, bâtie sur les principes stricts de l'Architecture Hexagonale (Clean Architecture). Pour garantir la prévisibilité des algorithmes et la sécurité du capital, le système s'appuie sur le mode *Netting* (une seule position ou exposition globale autorisée par actif et par robot). L'orchestration transactionnelle de ce cycle est pilotée par le module central `ExecutionEngine`.

### Motivation
Dans le trading en conditions réelles, la transition entre la phase d'intention (la soumission d'ordres limites ou brackets) et la phase d'exposition (la position active sur le marché) est soumise à des contraintes microstructurelles asynchrones complexes. L'absence de synchronisation étanche de la mémoire vive face aux latences du réseau ou des simulateurs représente le plus grand risque de défaillance opérationnelle d'un framework (ordres orphelins, positions fantômes ou exécutions non traquées).

### Problématique
Actuellement, l'`ExecutionEngine` souffre d'une surcharge cognitive majeure (violation du principe de responsabilité unique / SRP). Il gère simultanément le routage réseau, la gestion des risques et la persistance comptable de la RAM via son dictionnaire local `self._order_groups`. 
De plus, la logique d'interruption de trade souffre d'une asymétrie microstructurelle entre les courtiers :
* Sous **IG Markets**, la coupure d'un trade s'exécute via un unique `DELETE` HTTP sur un identifiant global (`dealId`), le courtier nettoyant automatiquement les protections Stop Loss / Take Profit attachées.
* Sous **Backtrader**, la fermeture nécessite un appel à `strategy.close()` combiné au nettoyage ou à la cascade automatique des ordres enfants chaînés via le paramètre `parent=parent`.

Tenter d'implémenter un mécanisme d'annulation directe ou d'analyse d'état au sein de l'Engine alourdirait le code, le rendrait dépendant des détails de l'infrastructure et introduirait des conditions de concurrence (*race conditions*) critiques.

---

## 2. Synthèse Manageriale

L'arbitrage architectural acté dans ce document résout définitivement la tension entre la pureté métier (le Domaine) et les contraintes réseau (l'Infrastructure). La solution repose sur l'extraction de la mémoire vive logique de l'Engine vers un tout nouveau sous-module autonome : l'**`ExecutionTracker`**.

### Les bénéfices stratégiques de cette refonte :
1. **Sécurité amont absolue (Cadenassage prioritaire)** : L'`ExecutionTracker` inscrit l'intention de trade en RAM *immédiatement avant* l'envoi de la commande sur le réseau. Ce "verrouillage prioritaire" élimine la faille temporelle classique où une confirmation d'exécution ultra-rapide du courtier percute le framework avant que sa mémoire locale ne soit initialisée.
2. **Principe "Tell, Don't Ask" (Dis-lui quoi faire, ne lui demande pas son état)** : L'Engine ne cherche plus à décortiquer le statut des ordres (si un ordre pend ou est exécuté). Il applique une commande d'impulsion binaire. Le Tracker qualifie l'état interne de manière étanche et pilote l'infrastructure en lui dictant un ordre clair et stupide (`cancel_order` ou `close_position`).
3. **Élimination de la sur-ingénierie** : Grâce à la découverte que Backtrader et IG Markets gèrent tous deux la destruction des protections enfants par cascade native (à condition de cibler l'ordre parent racine), l'introduction d'états transitoires complexes (comme un `MacroState` ou des enums intermédiaires) a été rejetée. Le système gagne en concision, en performance et en lisibilité (KISS).
4. **Zéro Fuite de Mémoire** : Le cycle de vie mémoriel est bouclé de manière purement événementielle. Le Tracker gère l'entrée, applique les modifications en fonction des notifications du courtier, et libère automatiquement la RAM (opération `.pop()`) dès qu'un trade atteint son dénouement final.

---

## 3. Plan de Route et d'Implémentation Épuré

Ce plan de développement linéaire garantit des commits atomiques, stables, 100% verts, documentés et testables de manière isolée en TDD.

### Section 1 : Alignement de l'Infrastructure (Le Port et l'Adaptateur)

#### 📍 Jalon 1 : Évolution du Port et de l'Adaptateur Backtrader
* **Fichiers** : 
  * `broker_adapters/base_broker_adapter.py`
  * `broker_adapters/backtrader_broker_adapter.py`
* **Action** : 
  * Déclarer `cancel_order(self, order: Order) -> None` et `close_position(self, order: Order) -> None` en méthodes abstraites dans le Port d'infrastructure.
  * Implémenter ces deux méthodes dans l'adaptateur Backtrader (`broker.cancel(parent_order)` et `strategy.close(data)`). Le chaînage `parent=parent` existant sous Backtrader permet de purger automatiquement les enfants en cascade native.
* **Message de commit** : 
```text
feat(broker): add cancel_order and close_position for broker adapters
```

---

### Section 2 : L'ExecutionTracker (Domaine Pur & Autonome)

#### 📍 Jalon 2 : Initialisation et Netting (L'Entrée)
* **Fichier** : 
  * `core/execution_tracker.py` (Nouveau fichier)
* **Action** : 
  * Créer la classe `ExecutionTracker` avec sa map privée `_executions: dict[str, OrderGroup]`.
  * Implémenter `has_active_execution(self, bot_id: str) -> bool`.
  * Implémenter `register_order(self, bot_id: str, order: Order) -> None` : fabrique l'`OrderGroup` en interne et fige la RAM *avant* la transmission réseau.
* **Message de commit** : 
```text
feat(core): initialize ExecutionTracker and implement register_order
```

#### 📍 Jalon 3 : Cycle de Vie Événementiel (Le Milieu et la Fin)
* **Fichier** : 
  * `core/execution_tracker.py`
* **Action** : 
  * Implémenter la méthode publique `process_broker_event(self, event: BrokerEvent) -> None`.
  * Intercepter les `OrderReceipt` pour faire évoluer le statut interne de l'ordre parent (ex: `PENDING` ➔ `FILLED`).
  * Intercepter les `TradeReceipt` de clôture (`is_open=False`) ou les annulations (`CANCELED`) pour exécuter le `.pop()` définitif et libérer la RAM sans fuite de mémoire.
* **Message de commit** : 
```text
feat(core): add process_broker_event to ExecutionTracker
```

#### 📍 Jalon 4 : Logique d'Extinction Encapsulée (L'Interruption)
* **Fichier** : 
  * `core/execution_tracker.py`
* **Action** : 
  * Implémenter `terminate_execution(self, bot_id: str, broker_adapter: BaseBrokerAdapter) -> None`.
  * Récupérer l'`OrderGroup` en RAM (sans l'extraire pour laisser les flux asynchrones se terminer proprement au Jalon 3).
  * Inspecter le statut de l'ordre parent : s'il est toujours en attente (`PENDING`), elle appelle `broker_adapter.cancel_order(parent)`. S'il est exécuté (`FILLED`), elle appelle `broker_adapter.close_position(parent)`.
* **Message de commit** : 
```text
feat(core): add terminate_execution to ExecutionTracker
```

---

### Section 3 : Refactoring de l'Engine (Orchestration Globale)

#### 📍 Jalon 5 : Propriétés d'`ExposureIntent`
* **Fichier** : 
  * `core/models/trading.py`
* **Action** : 
  * Ajouter les propriétés en lecture seule (comme `.is_exit`, `.is_entry`, `.is_flat`) à la classe `ExposureIntent` afin d'encapsuler et d'éliminer définitivement les valeurs magiques `0.0` et `None` au niveau de l'Engine.
* **Message de commit** : 
```text
feat(core): add read-only properties to ExposureIntent
```

#### 📍 Jalon 6 : Injection du Tracker et Refactoring de la Boucle
* **Fichier** : 
  * `core/execution_engine.py`
* **Action** : 
  * Modifier le constructeur pour injecter `ExecutionTracker` et supprimer définitivement l'ancien stockage de RAM local `self._order_groups`.
  * Nettoyer `_process_broker_event`, `_handle_order_notification` and `_handle_trade_notification` pour transférer directement et de manière brute chaque événement reçu au Tracker via `self._execution_tracker.process_broker_event(broker_event)`.
  * Réécrire la section d'arbitrage de `run_execution_cycle()` sous la forme de 3 branches strictes `if / elif / else` s'appuyant sur les nouvelles propriétés explicites d'`ExposureIntent` et sur les appels au Tracker.
* **Message de commit** : 
```text
feat(core): inject ExecutionTracker and route broker events in engine
```

---

## 4. Conclusion

En appliquant ce plan de route, AEGIS se dote d'une infrastructure transactionnelle hautement découplée et d'une robustesse mathématique indispensable pour le passage en production. L'`ExecutionEngine` redevient un orchestrateur de flux purement fluide et virtuel. Les structures d'`OrderGroup` sont sanctuarisées dans un module dédié. La complexité microstructurelle de la liquidation (Backtrader vs IG) est élégamment masquée derrière des abstractions stupides portées par les ports d'infrastructure. Le système est désormais structurellement immunisé contre les désynchronisations de mémoire et prêt pour l'intégration future d'un interrupteur général d'urgence (Kill Switch).

