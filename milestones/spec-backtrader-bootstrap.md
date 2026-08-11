# Cahier des Charges — Sous-système Backtest Aegis
## 1. Introduction

### Contexte
Le projet **Aegis** est un framework de trading quantitatif conçu en Python selon les principes de la *Clean Architecture* (architecture hexagonale). Son objectif principal est de séparer de manière étanche la logique métier (génération de signaux, évaluation des stratégies, gestion des risques et tailles des positions) des couches d'infrastructure technique (accès aux API de courtiers en direct, bases de données, ou moteurs de backtest).

### Motivation
Pour valider des stratégies quantitatives avant leur déploiement en conditions réelles, le framework doit être capable d'exécuter des simulations historiques précises (backtesting). Plutôt que de redévelopper un moteur de réévaluation des ordres complexe et sujet aux erreurs d'arrondis, le choix a été fait d'exploiter la maturité de **Backtrader**.

L'infrastructure actuelle d'Aegis intègre déjà un système de synchronisation multi-thread performant (pattern dit « Ping-Pong » via un objet `BacktraderBridge` et une stratégie miroir `BacktraderProxyStrategy`). Ce mécanisme permet d'exécuter en parallèle la boucle d'infrastructure (Backtrader) et la boucle de décision (Aegis Core), garantissant ainsi un agnosticisme total du bot.

### Problématique
Bien que la plomberie d'intégration asynchrone et les tests unitaires/d'intégration soient en place et validés, le framework ne dispose pas encore d'un point d'entrée opérationnel pour l'utilisateur final. Il manque :
1. Une couche de configuration formalisée pour injecter les contraintes micro-structurelles des courtiers (spreads, leviers, typage financier strict).
2. Un orchestrateur unifié encapsulant la complexité de la gestion des threads.
3. Un script d'amorçage (*bootstrap*) permettant d'exécuter une simulation sur de véritables fichiers de données historiques (CSV).

## 2. Spécifications Techniques des Composants

Pour finaliser l'environnement de backtest, trois briques logicielles distinctes doivent être intégrées au projet.

Flux architectural cible :
1. Fichiers MarketSpecs -> Définit la microstructure (ex: IG Markets)
2. run_backtest.py (Bootstrap) -> Charge le Runner + Injecte la config
3. BacktraderRunner -> Encapsule les threads & le système de Ping-Pong

### Brique 1 : Centralisation des configurations de marché (`MarketSpecs`)
Pour que le framework calcule correctement les marges, l'exposition et les frais de transaction, il doit connaître les spécifications exactes du contrat négocié. Afin de garantir le typage fort, la détection d'erreurs à l'analyse statique (`run_lint.sh`) et l'autocomplétion par l'IDE, ces configurations seront codées en Python natif (via l'import du modèle `ContractSpecification` utilisant des types `Decimal`).

*   **Emplacement cible** : Création d'un package `aegis/config/market_specs/`.
*   **Fichier cible initial** : `ig_markets.py` (modélisant les conditions de trading réelles du courtier IG Markets pour le Forex et les Indices).
*   **Exigences techniques** : Utilisation stricte de chaînes de caractères converties en `Decimal` pour éliminer tout risque d'erreur sur les nombres à virgule flottante au cours des calculs de solde.

### Brique 2 : Isolation d'infrastructure (`BacktraderRunner`)
Afin de respecter le principe de responsabilité unique (SRP), la logique de gestion des barrières de synchronisation et des cycles de vie des threads ne doit pas apparaître dans l'espace utilisateur. Elle est encapsulée dans une classe dédiée.

*   **Emplacement cible** : `aegis/infra/backtrader/runner.py`.
*   **Responsabilités de la classe** :
    *   Instancier l'objet centralisé `bt.Cerebro`.
    *   Configurer le lecteur de fichiers CSV via `bt.feeds.GenericCSVData` en appliquant le bon formatage de date (*timestamp*).
    *   Injecter la stratégie miroir `BacktraderProxyStrategy` et lier le composant de communication `BacktraderBridge`.
    *   Initialiser l'`AegisExecutionEngine` en lui injectant le bot utilisateur, l'adaptateur de courtier simulé (`BacktraderBrokerAdapter`) et le flux de données synchronisé (`BacktraderMarketFeed`).
    *   Gérer l'amorçage, le suivi de santé et la fermeture propre des threads de simulation (`INFRA` et `MAIN`).

### Brique 3 : Script de Bootstrap utilisateur (`run_backtest.py`)
Ce script constitue le fichier `main` manquant du projet. Il s'agit du point d'entrée exécutable par l'utilisateur pour lancer un backtest.

*   **Emplacement cible** : À la racine du projet `./run_backtest.py`.
*   **Comportement attendu** : Le script importe la configuration du courtier sélectionné, instancie la stratégie développée par l'utilisateur, définit le chemin d'accès vers le fichier CSV de données historiques Forex et délègue l'exécution globale au `BacktraderRunner`.

## 3. Conclusion et Évolution Live

La mise en place de ce cahier des charges permet d'exploiter immédiatement l'excellent travail d'architecture déjà réalisé sur Aegis. En isolant la gestion multi-thread dans le `BacktraderRunner` et en adoptant une configuration de marché en Python natif, le framework gagne en robustesse, en clarté et reste fidèle à sa philosophie agnostique. 

Une fois ces briques installées, l'utilisateur pourra passer d'un backtest historique basé sur Backtrader à une exécution en direct sur une API de production (comme CCXT ou un courtier CFD) en modifiant uniquement les paramètres d'initialisation de son script d'amorçage, sans jamais impacter le code de ses algorithmes de trading.

## Annexe : Plan de Développement Analytique

Le déploiement des fonctionnalités est découpé en **3 Jalons majeurs**, subdivisés en sous-jalons techniques clairs pour un suivi de projet agile.

### Jalon 1 : Couche de Configuration de la Microstructure de Marché
*Objectif : Mettre en place les données fixes décrivant les spécifications des instruments financiers.*

*   **Sous-jalon 1.1 : Structure de répertoires**
    *   Créer le dossier `aegis/config/` et le sous-dossier `market_specs/`.
    *   Initialiser les fichiers `__init__.py` nécessaires pour exposer les futurs dictionnaires de configuration.
*   **Sous-jalon 1.2 : Implémentation du profil IG Markets**
    *   Créer le fichier `ig_markets.py`.
    *   Déclarer un dictionnaire global nommé `IG_MARKETS_SPECIFICATIONS`.
    *   Remplir le dictionnaire avec l'instanciation de `ContractSpecification` pour l'instrument `EURUSD` en utilisant des valeurs `Decimal` strictes (ex: `tick_size=Decimal('0.00001')`, `contract_multiplier=Decimal('100000')`).
*   **Sous-jalon 1.3 : Validation qualité de la configuration**
    *   Exécuter le script linter existant (`./run_lint.sh`) pour valider la conformité du code et du style PEP8.

### Jalon 2 : Développement du Framework d'Isolation (`BacktraderRunner`)
*Objectif : Encapsuler la gestion du parallélisme et la configuration de Cerebro dans un composant d'infrastructure réutilisable.*

*   **Sous-jalon 2.1 : Signature de la classe et Initialisation**
    *   Créer le fichier `aegis/infra/backtrader/runner.py`.
    *   Définir la classe `BacktraderRunner` avec un constructeur acceptant : `symbol: str`, `csv_filepath: str`, `contract_spec: ContractSpecification`, `bot: BaseBot`, et `initial_cash: float`.
*   **Sous-jalon 2.2 : Configuration du flux de données et de Cerebro**
    *   Intégrer le chargement des fichiers de prix via `bt.feeds.GenericCSVData` dans le runner.
    *   Ajouter le chaînage automatique du `BacktraderBridge` et de la stratégie miroir `BacktraderProxyStrategy`.
    *   Instancier l'orchestrateur de domaine `AegisExecutionEngine` en résolvant l'enregistrement du contrat dans le `ContractRegistry`.
*   **Sous-jalon 2.3 : Cycle de vie des threads**
    *   Développer la méthode `.run()` de la classe.
    *   Implémenter la création des deux threads nommés `INFRA` (Backtrader) et `MAIN` (Aegis Core).
    *   Ajouter les mécanismes de capture d'exceptions (blocs `try/except`) au sein des workers pour forcer l'arrêt du pont (`bridge.stop_simulation()`) si un thread plante.

### Jalon 3 : Script d'Amorçage et Validation Réelle
*Objectif : Assembler les composants pour exécuter une simulation complète de bout en bout sur des données réelles.*

*   **Sous-jalon 3.1 : Script d'entrée de production**
    *   Créer le fichier `./run_backtest.py` à la racine de l'espace de travail.
    *   Mettre en place la configuration du logger Python pour afficher le nom du thread émetteur dans la console `(%(threadName)s)`.
*   **Sous-jalon 3.2 : Intégration des données physiques**
    *   Créer un répertoire local `./data/` et y placer un fichier échantillon d'historique Forex au format CSV (`eurusd_1m_historical.csv`).
    *   Configurer le mapping des index du fichier CSV dans le chargeur de données du runner (Open, High, Low, Close, Volume).
*   **Sous-jalon 3.3 : Test d'exécution et de fermeture**
    *   Lancer la commande `python run_backtest.py`.
    *   Vérifier graphiquement dans la console le bon déroulement alterné des boucles d'évaluation (cycles de ping-pong).
    *   S'assurer que le script s'arrête proprement une fois la fin du fichier CSV atteinte via la levée de l'exception `StopIteration`.

