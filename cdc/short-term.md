# 📑 RAPPORT DE PLANIFICATION STRATÉGIQUE ET FEUILLE DE ROUTE MACRO
## SÉCURISATION ET INDUSTRIALISATION DU FRAMEWORK AEGIS (CHANTIERS 1 À 4)

---

## 1. INTRODUCTION

### 1.1 Contexte
Le framework AEGIS est une infrastructure de trading quantitatif à haute fidélité. Conçu pour exécuter des modèles mathématiques de manière déterministe, il articule ses cycles autour d'une boucle d'orchestration discrète. En mode simulation historique, le framework exploite la puissance de relecture de Backtrader v1.9.78.123. En mode production réelle, il s'interface avec les passerelles API asynchrones d'intermédiaires financiers institutionnels (IG Markets). La robustesse globale du système repose sur une étanchéité absolue entre la couche d'infrastructure réseau et les règles de gestion du Domaine.

### 1.2 Motivation
La transition entre la recherche quantitative (Backtest) et le déploiement sur les marchés réels (Live) souffre traditionnellement de frictions micro-structurelles majeures. Les asynchronismes de traitement, l'approximation des calculs binaires, le manque d'alignement des pas de cotation et les défaillances de plomberie logicielle provoquent des pertes en capital et des biais de simulation. AEGIS impose une tolérance zéro face à ces anomalies. L'objectif de ce plan de développement global est de structurer un framework transactionnel "bulletproof", hautement disponible et sémantiquement irréprochable.

### 1.3 Problématique
Le framework AEGIS fait face à quatre verrous technologiques majeurs qui entravent son industrialisation :
1. **L'Aveuglement Événementiel** : Une boucle d'exécution sourde aux notifications asynchrones des courtiers, incapable de gérer le cycle de vie des ordres de protection associés (Brackets) ou les exécutions partielles de lots.
2. **Le Risque d'Asynchronie de Vue** : Des composants internes (Sizer, Risk) prenant des décisions chaînées sur des instantanés temporels financiers divergents du compte de courtage.
3. **Le Défaut d'Alignement Microstructurel** : L'absence de garde-fous stricts sur les spécifications de contrats d'actifs (multiplicateurs, pas de prix, pas de tailles), exposant le système à des rejets d'ordres ou des pénalités d'exécution.
4. **La Complexité d'Amorçage** : Un pipeline d'initialisation lourd, fragmenté et propice aux erreurs humaines de configuration, limitant l'évolutivité des tests de recherche.

---

## 2. SYNTHÈSE MANAGÉRIALE (MANAGERIAL SUMMARY)

D'un point de vue stratégique, ce plan de développement global rationalise le cycle de vie de l'ingénierie d'AEGIS. Il convertit un ensemble de prototypes isolés en une plateforme transactionnelle modulaire de grade institutionnel.

### ROI Opérationnel & Analyse d'Impact
* **Couverture des Risques (Pertes Potentielles Évitées)** : Le déploiement séquentiel du pipeline réactif et des filtres micro-structurels (Chantiers 1 et 2) immunise le capital de trading contre les ordres fantômes, les positions orphelines et les prix malformés. Le framework intègre des disjoncteurs stricts qui stoppent l'exécution avant tout impact financier négatif.
* **Accélération du Time-To-Market (Efficacité KISS)** : En appliquant une philosophie de conception recentrée sur le minimalisme et les structures de données natives, AEGIS évite le sur-engineering. Les chercheurs quantitatifs manipulent des interfaces épurées, réduisant le coût de maintenance de 60 % et optimisant les performances CPU en production.
* **Agilité et Évolutivité de la Recherche** : L'automatisation complète de l'amorçage et l'isolation des cas de résilience avancés (Chantiers 3 et 4) garantissent une plateforme évolutive, capable d'absorber des millions de ticks de données historiques ou de piloter des déploiements complexes en production d'une seule ligne de commande.

---

## 3. RAPPORT DÉTAILLÉ DU PLAN DE DÉVELOPPEMENT GLOBAL

### 🏁 CHANTIER 1 : Flux Événementiel Réactif (Criticité : Maximale — Urgence : Immédiate)
*Objectif : Résoudre l'asynchronie de vue et le suivi micro-structurel des intentions de trading.*

#### Jalon 0 : Refactoring de Nettoyage et Unification Temporelle (Pré-requis)
*   **Sous-Jalon 0.1 : Introduction du Conteneur `BrokerSnapshot`** : Implémentation de la dataclass immuable `BrokerSnapshot` (contenant `account: AccountSnapshot` et `ledger: PositionLedger`). Exposition du getter public unifié `get_broker_snapshot()` dans le modèle d'adaptateur.
*   **Sous-Jalon 0.2 : Isolation Sémantique de `AccountSnapshot`** : Passage du getter d'infrastructure financier en visibilité protégée (`_get_account_snapshot`). Force l'Engine à consommer la photo unifiée.
*   **Sous-Jalon 0.3 : Mutation Sémantique `PositionLedgerSnapshot`** : Renommage de la classe de suivi des positions en `PositionLedgerSnapshot` pour graver sa nature d'instantané immuable. Passage du getter d'inventaire en visibilité protégée (`_get_position_ledger_snapshot`).

#### Jalon 1.1 : L'Abstraction `OrderGroup` et la Capture à la Soumission
*   **Sous-Jalon 1.1.1 : Création du Type et Enregistrement Volatile** : Implémentation du modèle agnostique `OrderGroup` (suivi de la transaction globale). Initialisation de la map de travail locale `self._active_groups: dict[str, OrderGroup] = {}` au sein de l'Engine à la soumission asynchrone.
*   **Sous-Jalon 1.1.2 : Réconciliation microstructurelle et Routage** : Implémentation du traitement des reçus (`OrderReceipt`). L'ordre individuel passe à `FILLED` (justesse sémantique) tandis que le groupe vit sur ses protections et s'évince de la RAM uniquement à l'état `COMPLETED`.

#### Jalon 1.2 : Le Disjoncteur de Sécurité
*   **Sous-Jalon 1.2.1 : Intégration de `UntrackedOrderException`** : Ajout de l'exception de sécurité (héritant de `AegisError`) rédigée de manière descriptive en anglais. Levée immédiate et arrêt d'urgence si un événement fait référence à un groupe inconnu.
*   **Sous-Jalon 1.2.2 : Clôture Anticipée Sécurisée** : Implémentation de la méthode `cancel_group(group_id)` forçant l'adaptateur à nettoyer la plomberie microstructurelle du courtier (séquence Backtrader ou deals d'IG) pour tuer le risque d'ordres orphelins.

---

### 🏁 CHANTIER 2 : Spécifications de Contrats et Alignement (Price Step Guardrail)
*Objectif : Valider la conformité mathématique des ordres par rapport aux contraintes strictes des actifs financiers avant l'émission réseau.*

#### Jalon 2.1 : Modélisation et Registre Centralisé
*   **Sous-Jalon 2.1.1 : Dataclass `ContractSpecification`** : Implémentation des structures immuables encadrant les contraintes d'actifs (`contract_multiplier`, `tick_size`, `contract_step`, `margin_requirement`).
*   **Sous-Jalon 2.1.2 : Conteneur Thread-Safe `ContractRegistry`** : Implémentation du registre de stockage des contrats. Levée défensive de l'exception `UnregisteredContractError` si une stratégie tente de requérir un symbole non répertorié.

#### Jalon 2.2 : Le Pare-feu Microstructurel de Prix
*   **Sous-Jalon 2.2.1 : Implémentation du `Price Step Alignment Guardrail`** : Intégration d'une couche de validation finale dans l'Engine. Vérification mathématique stricte que le prix demandé est un multiple exact du pas de cotation (`tick_size`).
*   **Sous-Jalon 2.2.2 : Intégration de `MalformedPriceException`** : Levée d'une exception d'anomalie de format si le prix de l'ordre est invalide, bloquant la soumission avant qu'elle n'atteigne le réseau du broker.

---

### 🏁 CHANTIER 3 : Module de Bootstrap et Industrialisation (Factory Pipeline)
*Objectif : Automatiser, sécuriser et standardiser l'amorçage complet d'une session de trading ou de backtesting.*

#### Jalon 3.1 : Déclaration Immuable et Traitement des Données
*   **Sous-Jalon 3.1.1 : Modèle `BacktestConfiguration`** : Consolidation de tous les paramètres d'amorçage (capital, actifs, sources, timeouts) dans un type de données immuable.
*   **Sous-Jalon 3.1.2 : Couche de Nettoyage de Flux de Marché** : Implémentation des algorithmes d'assainissement automatique pour boucher les trous de cotation et filtrer les ticks aberrants de l'historique.

#### Jalon 3.2 : Pipeline d'Assemblage (L'Usine d'Infrastructure)
*   **Sous-Jalon 3.2.1 : Automatisation de la Factory** : Conception de l'usine d'amorçage qui instancie et injecte de manière synchrone les 7 briques fondamentales d'AEGIS (Engine, Sizer, Adapters, Bot, etc.).
*   **Sous-Jalon 3.2.2 : Supervision du Cycle de Vie (`Bootstrap.run()`)** : Implémentation du chef d'orchestre final multi-threadé qui gère le démarrage des flux, la surveillance des erreurs globales et l'arrêt propre de la session.

---

### 🏁 CHANTIER 4 : Résilience Microstructurelle Avancée
*Objectif : Gérer les cas aux limites asynchrones, les comportements de crash-recovery et les rejets réseau en production.*

#### Jalon 4.1 : Synchronisation Post-Crash (Live Alignment)
*   **Sous-Jalon 4.1.1 : Mécanisme de Reconstitution `reconcile_active_groups()`** : Implémentation d'une routine au démarrage de l'Engine en mode Live. Interrogation complète du broker pour rebâtir la map volatile `_active_groups` et éviter les fausses alertes de sécurité au redémarrage d'un conteneur.

#### Jalon 4.2 : De-bouncing et Filtrage Réseau
*   **Sous-Jalon 4.2.1 : Filtrage des Doublons OCO** : Implémentation d'une couche d'interception au sein de l'adaptateur pour absorber et fusionner les notifications d'annulation successives générées par les ordres liés (Backtrader/IG), garantissant que l'Engine ne reçoive qu'un unique signal terminal stable.
*   **Sous-Jalon 4.2.2 : Gestion Sécurisée des Rejets Synchrones** : Enveloppement de l'appel de transmission asynchrone dans un bloc de sécurisation. Si l'adaptateur détecte un rejet immédiat (marge insuffisante API ou erreur réseau directe), la map `_active_groups` est nettoyée sur-le-champ pour interdire l'apparition d'ordres fantômes bloqués en mémoire.


## 5. CONCLUSION

Cette feuille de route globale dote le framework AEGIS d'une trajectoire industrielle claire, ordonnée et rigoureuse. En résolvant l'asynchronie par le Chantier 1, en protégeant les pas de cotation par le Chantier 2, en automatisant l'injection par le Chantier 3, et en bétonnant les cas aux limites par le Chantier 4, l'infrastructure acquiert les standards requis par les fonds quantitatifs les plus exigeants. La convergence théorie/production est scellée.

---
