# 📑 RAPPORT DE PLANIFICATION TECHNIQUE ET FEUILLE DE ROUTE
## AEGIS FRAMEWORK — ENGINE RE-ENGINEERING (CHANTIER 1)

---

## 1. INTRODUCTION

### 1.1 Contexte
Le framework AEGIS est une infrastructure de trading quantitatif à haute fidélité. Sa boucle d'orchestration discrète consomme des flux de marché et pilote des robots algorithmiques de manière déterministe. Pour valider les modèles de recherche avant leur déploiement réel, le système s'appuie sur le moteur de simulation historique Backtrader v1.9.78.123 et s'interface en production avec des passerelles API asynchrones (IG Markets). La viabilité de cette architecture repose sur l'étanchéité absolue entre la couche d'infrastructure et les règles de gestion du Domaine.

### 1.2 Motivation
Afin d'éliminer tout biais micro-structurel lors de la recherche alpha, AEGIS impose une tolérance zéro face à l'approximation financière ou à l'aveuglement événementiel. Après avoir sécurisé l'intégrité arithmétique globale via l'usage exclusif du type `Decimal`, le framework doit unifier sa comptabilité transactionnelle. Un chercheur quantitatif doit avoir la garantie absolue qu'un modèle validé en simulation historique se comportera de manière rigoureusement identique en production réelle, sans distorsion mémorielle ou temporelle.

### 1.3 Problématique
Le moteur d'exécution actuel souffre d'une asynchronie aveugle : la méthode `_process_broker_event` est une coquille vide. L'Engine émet des ordres mais s'avère incapable d'intercepter et de traiter les notifications asynchrones de ses intermédiaires financiers (brokers). Cette absence de boucle de rétroaction engendre trois vulnérabilités critiques :
1. **L'aveuglement micro-structurel** : L'incapacité à assimiler les exécutions partielles de lots sur le marché.
2. **Le risque d'ordres orphelins** : Le manque de contrôle sur le cycle de vie des ordres de protection attachés (Brackets de type Stop-Loss et Take-Profit) qui s'activent et survivent en séquence à l'insu de l'Engine.
3. **L'asynchronie de vue** : Le risque majeur que des composants chaînés (le calibreur de taille et le validateur de risque) prennent des décisions contradictoires au cours d'un même cycle en interrogeant l'infrastructure sur des fenêtres temporelles mouvantes.

---

## 2. SYNTHÈSE MANAGÉRIALE (MANAGERIAL SUMMARY)

D'un point de vue stratégique, l'exécution de ce plan de développement transforme AEGIS d'un simulateur théorique en un automate de routage institutionnel blindé contre les pertes en capital liées aux défaillances de plomberie logicielle.

### Impact Business & Retour sur Investissement (ROI)
* **Protection Absolue du Capital (Risque de Ruine Éliminé)** : L'implémentation de la brique de sécurité stricte (`UntrackedOrderException`) et de la gestion de fermeture automatisée des groupes d'ordres agit comme un disjoncteur d'urgence. Elle éradique les "bugs silencieux" et les exécutions d'ordres fantômes qui pénalisent la performance des algorithmes en production.
* **Efficacité Industrielle Maximisée (Philosophie KISS)** : En résolvant ces frictions complexes par des structures natives (`dict`, `dataclass`) et en refusant la prolifération de classes managériales redondantes, le framework conserve une agilité maximale. Le code reste pur, lisible pour les mathématiciens et les chercheurs, minimisant le coût de maintenance et accélérant le *Time-to-Market*.
* **Élimination de la Dette Technique (Refactoring Jalon 0)** : Le choix stratégique de découper le jalon initial en trois étapes de transition douce sécurise la stabilité du build. La dette technique liée à l'asynchronie de vue est étouffée dans l'œuf avant d'ajouter la moindre complexité algorithmique.

---

## 3. RAPPORT DÉTAILLÉ DU PLAN DE DÉVELOPPEMENT

Le développement est orchestré en commits atomiques indépendants. À chaque sous-jalon, le framework doit exécuter son harnais de tests et ses scripts de vérification pour valider un état 100 % opérationnel (*100% test, coverage, lint*).

### 🏁 Jalon 0 : Refactoring de Nettoyage et Unification Temporelle (Pré-requis)

#### 🔹 Sous-Jalon 0.1 : Introduction du Conteneur `BrokerSnapshot`
* **Objectif** : Mettre en place la nouvelle structure d'accueil immuable et exposer le getter unifié sans perturber la visibilité de l'existant.
* **Changements de Production** :
  * Créer la dataclass immuable `BrokerSnapshot` (contenant `account: AccountSnapshot` et `ledger: PositionLedger`).
  * Ajouter la nouvelle méthode publique `get_broker_snapshot() -> BrokerSnapshot` dans `BaseBrokerAdapter` et `BacktraderBrokerAdapter`. Elle appelle les getters publics actuels pour composer son retour.
  * *Les anciens getters `get_account_snapshot()` and `get_position_ledger()` restent publics et inchangés à cette étape pour assurer une continuité totale.*
  * Mettre à jour le début de `run_execution_cycle` dans `core/execution_engine.py` pour capturer et distribuer ce snapshot global.
* **Changements de Tests** :
  * Intégrer les tests unitaires pour `BrokerSnapshot` et mettre à jour le mock de l'Engine.

#### 🔹 Sous-Jalon 0.2 : Isolation Sémantique de `AccountSnapshot`
* **Objectif** : Isoler le volet financier et couper son accès public direct depuis l'adaptateur pour forcer l'usage du snapshot unifié.
* **Changements de Production** :
  * Passer l'ancien getter de l'adaptateur en visibilité protégée : `get_account_snapshot()` ➔ `_get_account_snapshot()`.
  * Mettre à jour `BrokerSnapshot` pour qu'il utilise cette méthode protégée sous le capot.
  * S'assurer que plus aucun composant du moteur n'appelle l'ancienne version publique (tout le monde consomme désormais `broker_snapshot.account`).
* **Changements de Tests** :
  * Réaligner les mocks et les tests financiers pour valider l'étanchéité de ce périmètre.

#### 🔹 Sous-Jalon 0.3 : Isolation Sémantique et Mutation de `PositionLedgerSnapshot`
* **Objectif** : Clôturer le refactoring préalable en mutant l'inventaire des positions, en changeant son nom pour acter sa nature d'instantané et en fermant son accès public direct.
* **Changements de Production** :
  * Renommer la classe `PositionLedger` en `PositionLedgerSnapshot` dans `core/models/positions.py`.
  * Mettre à jour sa référence à l'intérieur du conteneur `BrokerSnapshot`.
  * Passer le dernier getter de l'adaptateur en visibilité protégée : `get_position_ledger()` ➔ `_get_position_ledger_snapshot()`.
  * Réaligner l'intégralité des imports descendants à travers tout le framework.
* **Changements de Tests** :
  * Mettre à jour l'ensemble des fixtures, usines de données (`factories.py`) et assertions de tests pour valider le type `PositionLedgerSnapshot`.

### 🏁 Jalon 1.1 : L'Abstraction `OrderGroup` et la Capture à la Soumission

#### 🔹 Sous-Jalon 1.1.1 : Création du Type et Enregistrement Volatile
* **Objectif** : Enregistrer l'intention globale de trading (Parent + Brackets) au sein d'un conteneur agnostique de manière asynchrone dès la soumission.
* **Changements de Production** :
  * Implémenter le modèle `OrderGroup` dans `core/models/orders.py` (contenant un `group_id`, un statut global de groupe et le dictionnaire d'ordres suivis).
  * Initialiser la map volatile `self._active_groups: dict[str, OrderGroup] = {}` dans `__init__` de l'Engine.
  * À l'appel asynchrone de `submit_order`, peupler immédiatement cette map avec l'identifiant du groupe généré par AEGIS.
* **Changements de Tests** :
  * Écrire un test unitaire validant qu'à la soumission, l'Engine instancie et stocke l'objet `OrderGroup` de façon non bloquante.

#### 🔹 Sous-Jalon 1.1.2 : Réconciliation microstructurelle et Routage des Événements
* **Objectif** : Intercepter et router les notifications pour mettre à jour l'état de l'ordre individuel (`FILLED`) tout en préservant le groupe jusqu'à sa complétion complète (`COMPLETED`).
* **Changements de Production** :
  * Dans `_process_broker_event`, filtrer les événements `EventType.ORDER_NOTIFICATION` portant un `OrderReceipt`.
  * Régler le routeur pour qu'il transmette le reçu à l'`OrderGroup` adéquat (l'adaptateur injectant le `group_id` d'AEGIS en face des références techniques de Backtrader ou d'IG).
  * Si un ordre passe à `FILLED`, l'Engine sait qu'il est en position. Si le groupe complet n'a plus aucune protection active (les brackets OCO se sont nettoyés), le groupe passe à `COMPLETED` et l'Engine l'évince de sa mémoire vive.
* **Changements de Tests** :
  * Injecter des scénarios d'exécutions partielles et d'ordres brackets enfants : valider que l'ordre parent passe bien à `FILLED` (sémantique respectée) et que le groupe n'est purgé de la RAM qu'au signal de complétion finale.

---

### 🏁 Jalon 1.2 : Le Disjoncteur de Sécurité

#### 🔹 Sous-Jalon 1.2.1 : Intégration de `UntrackedOrderException`
* **Objectif** : Déclencher un arrêt d'urgence immédiat si une notification microstructurelle inconnue se présente.
* **Changements de Production** :
  * Ajouter l'exception `UntrackedOrderException` (héritant de `AegisError`) dans `core/exceptions.py`.
  * Dans `_process_broker_event`, insérer le garde-fou strict : si le `group_id` d'un reçu n'est pas présent dans `self._active_groups`, lever immédiatement `UntrackedOrderException`.
* **Changements de Tests** :
  * Écrire un test de sécurité injectant un événement avec un identifiant de groupe corrompu ou fantôme au milieu de la boucle, et vérifier à l'aide de `pytest.raises` que l'Engine coupe immédiatement ses moteurs.

#### 🔹 Sous-Jalon 1.2.2 : Clôture Anticipée Sécurisée
* **Objectif** : Permettre à l'Engine de commander l'annulation asynchrone d'un groupe entier pour prémunir le compte contre les ordres orphelins.
* **Changements de Production** :
  * Implémenter la méthode `cancel_group(group_id: str)` dans l'Engine.
  * L'adaptateur intercepte cette demande globale et se charge de la plomberie technique (annuler les références individuelles une par une sur Backtrader, ou liquider le deal principal sur l'API d'IG, ce qui nettoie nativement les brackets liés sur leurs serveurs).
* **Changements de Tests** :
  * Valider qu'un ordre de clôture anticipé déclenché par l'Engine provoque l'annulation de toutes les protections en vol et ramène proprement la map volatile à zéro élément.

## 4. CONCLUSION

Ce plan de développement structure de manière définitive l'exécution du Chantier 1. En articulant le framework autour d'une transition douce (Jalon 0) pour figer la vérité temporelle via `BrokerSnapshot`, puis en déployant la gestion sémantique par `OrderGroup` gardée par le disjoncteur `UntrackedOrderException`, AEGIS acquiert une robustesse industrielle. Le framework est armé pour converger à 100 % entre la théorie de la recherche et la pratique de la production.

---
