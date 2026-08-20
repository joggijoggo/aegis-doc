# RAPPORT 1 : CADRE STRATÉGIQUE, DIRECTEUR ET SYNOPTIQUE DE L'ARCHITECTURE GLOBALE

## PARTIE 1 : SYNTHÈSE MANAGÉRIALE ET ENCADREMENT SCIENTIFIQUE

**Statut du Document :** Cadre Directeur de Recherche et d'Industrialisation (In-Sample 2019-2022)  
**Domaine :** Ingénierie des Données Financières & Stabilité Multi-Univers du *Feature Space*  
**Actifs de Référence Principaux :** Forex Spot (EUR/USD, GBP/JPY, AUD/NZD) – Données Ticks Level 1  

---

## Synthèse Managériale

La majorité des stratégies de trading systématique et des modèles de Machine Learning échouent lors de leur déploiement en production en raison du **p-hacking de second niveau**. Ce biais méthodologique se produit lorsque le chercheur, après avoir constaté la dégradation des performances sur les données futures (*Out-Of-Sample*), revient itérativement modifier ses indicateurs, ses transformations ou ses fenêtres temporelles d'apprentissage jusqu'à ce que les résultats soient acceptables. À cet instant, les données futures sont définitivement contaminées et le modèle perd toute capacité de généralisation réelle.

Ce projet de recherche introduit une rupture scientifique majeure en appliquant un principe de sanctuarisation absolue. Les données futures (2023-2026) sont totalement exclues de la phase de recherche pour être définitivement écartées de l'environnement de validation actuel. À l'inverse, la période historique **In-Sample (2019-2022)** est saturée mathématiquement. En fragmentant ces 4 années en blocs interchangeables de 6 mois et en les recombinant de manière non chronologique, nous créons **28 univers de marché alternatifs**. 

Pour survivre et être qualifiée, une configuration de paramètres ne peut plus simplement "avoir fonctionné dans le passé réel" : elle doit prouver sa stabilité distributionnelle (Loi Normale) et son immunité face à la dérive (*Feature Drift*) à travers ces 28 scénarios déstructurés. L'ensemble de ce processus industriel est soutenu par une architecture logicielle découpée en 3 modules autonomes et étanches, communiquant par des flux de données standardisés et optimisés en mémoire vive.

---

## 1. Introduction

### 1.1. Contexte
Dans le cadre de l'ingénierie quantitative appliquée au marché des changes (Forex), la representation classique des cours sous forme de barres temporelles fixes (minutes, heures, jours) reste la norme de l'industrie de détail. Pourtant, d'un point de vue microstructurel, le temps chronologique est une unité purement exogène. Le Forex est un marché de gré à gré (OTC) décentralisé, ouvert 24h/24, caractérisé par une saisonnalité intra-journalière extrême (léthargie de la session asiatique opposée à la haute densité des sessions de Londres et New York) et une absence de volume officiel centralisé (Level 2/3). Travailler sur des données temporelles revient à échantillonner un flux hautement irrégulier avec une horloge rigide, ce qui détruit les propriétés statistiques de la donnée financière.

### 1.2. Motivation
Les données financières brutes ainsi échantillonnées souffrent de non-stationnarité, d'hétéroscédasticité (variance fluctuante) et de queues de distribution épaisses (*fat tails*). Or, la quasi-totalité des frameworks d'apprentissage statistique et d'optimisation mathématique présupposent des rendements distribués selon une loi normale et stationnaire. L'échantillonnage asynchrone (Range, Renko, Imbalance, Run Bars) résout ce problème en ralentissant la collecte de données lorsque le marché est inactif et en l'accélérant lors des flux d'informations, agissant comme un filtre de bruit naturel. La motivation de cette étude est d'établir scientifiquement les règles de transformation et les configurations de paramètres qui permettent de forcer la donnée microstructurelle brute à converger vers une distribution gaussienne stable.

### 1.3. Problématique
La question centrale de cette recherche se formule ainsi :
> **« Dans quelle mesure l’exploitation intensive et combinatoire de la donnée In-Sample (2019-2022) au travers de 28 univers virtuels ajustés permet-elle d'identifier et de valider mathématiquement un espace de caractéristiques stationnaire, normalisé et immunisé contre le Feature Drift, éliminant ainsi le p-hacking de second niveau avant tout contact avec les données futures ? »**

---

## PARTIE 2 : LE PARADIGME SOC ET L'ANALYSE MODULAIRE

---

## 2. Le Paradigme de la Séparation des Préoccupations (SoC)

Pour garantir la transparence académique, éviter les fuites de données d'un module à l'autre et optimiser les temps de calcul sur des volumes massifs de ticks, le pipeline quantitatif est structuré en trois projets logiciels autonomes. Chaque projet dispose d'une responsabilité unique et immuable.

*   **PROJET 1 : LE CONSTRUCTEUR D'HISTOIRES (TICK ENGINE)**
    Responsabilité : Chargement des parquets de ticks journaliers, partitionnement CPCV, application des algorithmes de suture virtuelle en RAM.
*   **PROJET 2 : LE CONSTRUCTEUR DE DATASET (BAR ENGINE)**
    Responsabilité : Réception du flux de ticks ajustés, échantillonnage asynchrone selon une grille d'hyperparamètres externes, clôture et émission des barres brutes augmentées.
*   **PROJET 3 : LE LABORATOIRE SCIENTIFIQUE (EVAL SUITE)**
    Responsabilité : Réception des barres brutes, extraction des features paramétrées, application des transformations statistiques, calcul des tests de normalité et de dérive.

---

## 3. Cartographie des Interactions et Responsabilités Modulaires

### 3.1. PROJET 1 : Le Moteur Microstructurel et Géométrique (Tick Engine)
Le Projet 1 gère l'infrastructure et la plomberie de bas niveau des données. Sa mission est d'annuler la chronologie réelle pour fabriquer les 28 trajectoires de marché de 3 ans.
*   **Intégrité Microstructurelle :** Les découpages se font strictement aux frontières journalières (fermeture du Forex le vendredi à 22:00:00 GMT), respectant la signature horaire des sessions.
*   **Algorithme de Suture Cumulatif :** Il applique à la volée en mémoire un opérateur de *Back-Adjustment* récursif (Delta P) sur les prix Bid et Ask pour effacer les gaps de raccordement tout en maintenant l'invariance du spread. Les volumes originaux de transactions (*Tick Count*) restent strictement inchangés. Un opérateur de translation temporelle (Delta T) ré-indexe les horodatages de manière monotone croissante.
*   **Virtualisation :** Aucune donnée n'est écrite sur disque. L'assemblage se fait virtuellement en RAM par pointeurs d'index.

### 3.2. PROJET 2 : Le Compresseur Structurel Passif (Bar Engine)
Le Projet 2 reçoit un flux de ticks bruts continus, fluides et ajustés, sans savoir s'ils proviennent de l'histoire réelle ou d'un univers combinatoire.
*   **Neutralité Scientifique :** C'est un exécuteur passif. Il applique les conditions de clôture (Temps, Volume, Prix, Information) dictées par une grille de paramètres externes (W, k).
*   **Gestion de l'Overshooting :** Si un tick unique apporte un volume d'activité qui dépasse le seuil de déclenchement de la barre, il est entièrement absorbé pour respecter la microstructure de l'ordre d'origine. Le compteur de la barre suivante est réinitialisé à zéro sans report de surplus ni fractionnement de tick.
*   **Livrable :** Il émet des structures de barres OHLCV brutes et augmentées des métriques d'activité (volume d'achat/vente cumulé, spread moyen).

### 3.3. PROJET 3 : Le Laboratoire d'Apprentissage et d'Évaluation (Eval Suite)
Le Projet 3 est l'arbitre statistique suprême de l'étude. Tout son travail d'évaluation est concentré sur les segments de test internes à la période In-Sample.
*   **Liberté de Modélisation :** Il prend les barres brutes, extrait les caractéristiques (Rendements, Volatilité de Garman-Klass, Entropie) et teste de manière paramétrable l'impact des transformations de normalisation (Z-Score glissant, différenciation fractionnaire).
*   **Sanction Statistique :** Il mesure la gaussianité via le test de Jarque-Bera et quantifie la dérive aux points de suture via le *Population Stability Index* (PSI) et la distance de Kolmogorov-Smirnov.
*   **Contrôle de la Multiplicité :** Il applique la correction de Bonferroni ou le *Deflated Sharpe Ratio* basés sur la taille de la grille de paramètres explorée pour éliminer le biais de *data snooping*.

---

## PARTIE 3 : INTERFACES, PISTES FUTURES ET CONCLUSION

---

## 4. Spécification des Contrats d'Interfaces (API Internes)

Pour préserver l'étanchéité absolue entre les phases de développement, les structures de données aux frontières des projets ont été figées contractuellement :

1.  **Interface P1 ➔ P2 (Flux de Ticks Ajustés) :**  
    Colonnes : `[timestamp, bidprice, askprice, bidvolume, askvolume, price_mid, spread, volume]`  
    *Contraintes : Index temporel strictement croissant, spreads originaux préservés, volumes bruts intacts.*
2.  **Interface P2 ➔ P3 (Flux de Barres Brutes et Paramétrées) :**  
    Colonnes : `[virtual_timestamp, parameter_config_id, open, high, low, close, total_ticks, total_bidvolume, total_askvolume, average_spread]`  
    *Contraintes : Aucune modification statistique ou indicator technique calculé. Données rattachées à l'identifiant unique de la configuration.*

---

## 5. Pistes Futures

*   **Extension Multi-Actifs et Crypto-Devises :** Transposer ce cadre de validation combinatoire sur des marchés centralisés (Futures) ou hautement fragmentés (Crypto-devises Spot) en adaptant la sous-famille des barres de Valeur Notionnelle.
*   **Intégration de l'Apprentissage par Renforcement (RL) :** Utiliser les 28 univers alternatifs du Projet 1 comme un simulateur d'environnement (*Gym Environment*) pour entraîner des agents de Deep Reinforcement Learning dans des conditions de stress-test intensives.
*   **Différenciation Fractionnaire Dynamique :** Explorer dans le Projet 3 l'implémentation d'un algorithme de différenciation fractionnaire dont l'ordre de dérivation (d) s'ajuste dynamiquement pour conserver un niveau de mémoire maximum tout en validant la stationnarité.

---

## 6. Conclusion

L'approche holistique formalisée dans ce rapport général pose des fondations architecturales inattaquables pour la recherche quantitative sur le Forex. En transformant le laboratoire In-Sample en un environnement de simulation multi-univers à haute sévérité statistique, ce protocole force l'identification de caractéristiques robustes par nature. Le système n'est plus dépendant d'un alginement de planètes chronologique favorable dans l'histoire réelle. Une fois ce cadre directeur validé, l'analyse détaillée de l'infrastructure de bas niveau (Projet 1) peut être initiée.

