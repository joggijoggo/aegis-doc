# RAPPORT TECHNIQUE 2 : SPÉCIFICATIONS TECHNIQUES ET ALGORITHMIQUES DU PROJET 1 – LE CONSTRUCTEUR D'HISTOIRES (TICK ENGINE)

---

## PARTIE 1 : ENCADREMENT STRATÉGIQUE ET JUSTIFICATION DU LABORATOIRE IN-SAMPLE

**Statut du Document :** Spécifications d'Ingénierie de l'Infrastructure des Données (In-Sample 2019-2022)  
**Composant Logiciel :** Projet 1 (Couche Bas Niveau / Gestion de la Microstructure)  
**Données d'Entrée :** Fichiers Apache Parquet Quotidiens Forex Spot Level 1  

---

## Synthèse Managériale

Le Projet 1 (Tick Engine) constitue le socle microstructurel de toute notre infrastructure de recherche quantitative. Sa mission stratégique est d'annuler définitivement l'ordre chronologique de l'histoire réelle pour immuniser nos futures caractéristiques (Features) contre le surapprentissage. Pour ce faire, ce module orchestre la construction de **28 univers de marché alternatifs** en appliquant la méthodologie de la Validation Croisée Combinatoire Purgée (Combinatorial Purged Cross-Validation).

Sur le plan technique, l'ingénierie de ce module repose sur un mécanisme de virtualisation absolue en mémoire vive (RAM). Pour optimiser les ressources informatiques, les fichiers Apache Parquet journaliers originaux ne sont jamais clonés ni écrits physiquement sur disque. Le module génère une table d'indexation ultra-légère et applique, de manière récursive et à la volée, des opérateurs mathématiques de suture sur le flux de ticks. Ces opérateurs garantissent la suppression des sauts de prix artificiels tout en maintenant l'invariance absolue de la liquidité (Spread) et l'intégrité de l'activité (Volume). Le Projet 1 fournit ainsi aux projets en aval un serveur proxy de données virtuelles parfaites et prêtes pour l'échantillonneur.

---

## 1. Introduction

### 1.1. Contexte
Dans l'industrie du trading algorithmique, l'accès à des données historiques de haute fidélité est une condition nécessaire mais non suffisante pour concevoir des modèles robustes. Sur le marché du Forex Spot, les données de niveau 1 (Level 1) sont caractérisées par un flux asynchrone de ticks (Bid/Ask/Volume) arrivant à haute fréquence. La manipulation de ces séries temporelles massives nécessite une infrastructure de données capable de respecter la microstructure du marché (l'écart acheteur-vendeur et l'intensité des cotations) tout en pliant l'axe du temps et des prix aux exigences des protocoles de validation scientifique.

### 1.2. Motivation
La motivation fondamentale du Projet 1 is de saturer mathématiquement la période d'étude In-Sample (2019-2022) afin de simuler des scénarios de marché extrêmes et inédits. Si un modèle de données n'est testé que sur le fil linéaire de l'histoire réelle, il apprendra des biais spécifiques à cette chronologie. En mélangeant de grands blocs de données réelles (par exemple, en faisant succéder immédiatement une phase de krach à une phase de compression de volatilité), nous forçons le système à développer une résilience structurelle. La matière première de ce projet est constituée de fichiers Apache Parquet quotidiens, un format hautement optimisé pour les calculs vectorisés en finance quantitative.

### 1.3. Problématique
La problématique technique de ce module s'énonce ainsi :
> **« Comment réordonner de manière non chronologique des fichiers Parquet journaliers de ticks pour construire 28 univers virtuels continus, sans introduire de biais d'anachronisme, et sans générer de sauts de prix artificiels qui détruiraient la validité statistique des analyses en aval ? »**

---

## 2. Justification du Périmètre Temporel et Dimensionnement Mathématique

### 2.1. Justification Macroéconomique du Bloc In-Sample (2019-2022)
Le choix exclusif de la période allant du 1er janvier 2019 au 31 décembre 2022 comme laboratoire In-Sample repose sur la densité et la diversité des régimes de volatilité qui s'y sont succédé. Ce bloc de 4 ans offre un échantillon exhaustif des états possibles de la microstructure moderne de l'EUR/USD :
* **2019 (Compression extrême) :** Une année de léthargie historique caractérisée par une absence de tendance et un bruit de fond pur. Elle sert de base de calibrage pour tester la résistance de nos descripteurs face aux faux signaux.
* **2020 (Choc exogène de liquidité) :** L'éclatement de la pandémie de la COVID-19 en mars a provoqué une explosion de la volatilité et des anomalies massives de flux d'ordres. Elle offre le terrain parfait pour évaluer l'accélération de nos méthodes d'échantillonnage asynchrones.
* **2021 (Transition structurelle) :** Une année d'alternance complexe entre retour à la moyenne et amorce de tendances institutionnelles.
* **2022 (Divergence monétaire lourde) :** Marqué par le réveil de l'inflation globale, le resserrement agressif de la Réserve Fédérale américaine et le passage de l'EUR/USD sous la parité (0.95). Ce régime de tendance lourde dicté par des flux de capitaux directionnels est indispensable pour valider la robustesse macroéconomique de l'étude.

En accumulant ces quatre régimes dans notre base IS, le modèle se confronte à l'ensemble des dynamiques du Forex moderne. L'exclusion des années suivantes (2023-2026) garantit un sanctuaire Out-Of-Sample massif pour la validation de production à long terme.

### 2.2. Justification Mathématique du Partitionnement Semestriel (6 mois)
Le choix de diviser ces 4 ans en 8 blocs de 6 mois répond à des critères de microstructure et de cohérence statistique :
1. **Stabilité microstructurelle inter-journalière :** Le fait de couper les blocs de données aux frontières strictes des journées de trading garantit que le Projet 2 reçoit un flux fluide. Chaque transition inter-blocs est vécue par le système comme une ouverture de session quotidienne standard.
2. **Puissance et convergence statistique :** Le découpage combinatoire basé sur 8 blocs dont on extrait 2 blocs de test à chaque itération engendre précisément 28 histoires alternatives (C(8,2)). Un échantillon de 28 trajectoires de 3 ans fournit une puissance statistique idéale, proche du seuil critique des 30 individus requis par le Théorème Central Limite, permettant à nos futurs tests d'atteindre une convergence mathématique stable, sans jamais déceler l'artifice des jointures.

---

## PARTIE 2 : LOGIQUE DE CLÔTURE ET ALGORITHMES DE SUTURE VECTORISÉE

---

## 3. Algorithme de Partitionnement et Matrice Combinatoire (CPCV)

La période historique In-Sample (2019-2022) fait l'objet d'une sectorisation rigoureuse selon les paramètres validés à la section précédente :

1. **Contrainte Journalière Absolue :** Les coupures de blocs ne s'effectuent jamais au milieu d'une journée de trading. Elles sont calées à la seconde près sur l'heure de fermeture quotidienne officielle du marché Forex (22:00:00 GMT / 17:00 EST). Cela élimine les biais de rupture de saisonnalité intraday.
2. **Génération des Univers :** L'algorithme CPCV extrait récursivement toutes les combinaisons d'entraînement possibles constituées de l'assemblage désordonné de 6 blocs (3 ans de données cumulées au total), isolant les 2 blocs restants (1 an de données) pour servir d'évaluation interne au sein du Projet 3.

---

## 4. Les Algorithmes Vectorisés de Suture Microstructurelle

Lorsque deux blocs non consécutifs sont soudés en RAM, le point de raccordement subit des distorsions géométriques qu'il faut corriger récursivement d'un bloc à l'autre.

### 4.1. Algorithme de Réalignement des Prix Récursif (Back-Adjustment Chain)
Pour éviter l'apparition d'un saut de prix artificiel qui introduirait un outlier extrême destructeur pour les tests statistiques, le Projet 1 implémente un algorithme d'ajustement cumulatif et récursif.

L'algorithme maintient une variable d'état appelée l'Ajustement Cumulé (`cumulative_price_shift`) tout au long de l'assemblage de la trajectoire :
* Pour le premier bloc de l'histoire alternative : L'ajustement initial est nul (`cumulative_price_shift = 0.0`).
* Pour chaque transition entre un bloc précédent (A) et un bloc suivant (B), le module calcule le gap sur la colonne `price_mid`. Le prix de fin ajusté du bloc précédent est égal au dernier prix mid de A augmenté de l'ajustement cumulé courant. Le gap est égal à la différence entre ce prix de fin ajusté et le tout premier prix mid brut du bloc B.
* Ce gap est ajouté à la variable d'état : `cumulative_price_shift += gap`.
* L'intégralité de la colonne `price_mid` du bloc B lissé est alors translatée en RAM par cette nouvelle valeur d'ajustement cumulé.

### 4.2. Algorithme d'Invariance du Spread et Intégrité du Volume
* **Préservation de la Liquidité (Spread) :** Pour empêcher mathématiquement l'apparition de spreads négatifs ou d'arbitrages artificiels, l'ajustement calculé sur le prix moyen doit être appliqué de façon strictement identique aux cours acheteurs et vendeurs. Pour chaque tick t du bloc translaté, les prix ajustés deviennent :
  `bidprice_ajusté = bidprice_brut + cumulative_price_shift`
  `askprice_ajusté = askprice_brut + cumulative_price_shift`
  Par soustraction, le spread ajusté reste rigoureusement identique au spread original historique.
* **Intégrité de l'Activité (Volume) :** Les colonnes `bidvolume`, `askvolume` et `volume` (Tick Count) mesurent l'intensité des flux et non une altitude de prix. Elles ne subissent aucun ajustement. Leurs valeurs numériques brutes sont transmises intactes.

### 4.3. Algorithme de Translation Temporelle (Shift Temporel)
Les structures de données en aval exigent un index temporel strictement croissant et monotone.
* L'algorithme calcule l'écart temporel exact entre le timestamp de fin du bloc précédent et le premier timestamp brut du bloc suivant.
* Ce delta temporel est additionné à l'ensemble des timestamps du bloc entrant, augmenté d'une constante de microstructure minimale (1 milliseconde) pour matérialiser la transition physique de la suture quotidienne sans créer de doublon temporel.

---

## PARTIE 3 : ARCHITECTURE LOGICIELLE, CONTRATS D'INTERFACE ET PERSPECTIVES

---

## 5. Architecture Informatique : Virtualisation par Indexation RAM

Afin d'éliminer les goulets d'étranglement de bande passante I/O et d'éviter le stockage redondant de fichiers massifs sur disque, le Projet 1 implémente un pattern de virtualisation pure.

1. **Chargement Unique en RAM :** Les fichiers Apache Parquet de la période In-Sample (2019-2022) sont lus une seule fois et stockés dans un tableau mémoire continu indexé via `Polars`.
2. **La Table des Métadonnées (Metadata Map) :** Le module génère un dictionnaire centralisé léger. Pour chacun des 28 univers alternatifs, ce dictionnaire contient l'ordre exact d'assemblage des blocs, les pointeurs de lignes physiques de début et de fin dans la RAM, ainsi que les coefficients précalculés d'ajustement de prix et de temps pour chaque segment.
3. **Streaming à la Volée :** Lors de l'exécution, le Projet 1 agit comme un itérateur de flux (Lazy Evaluation). Il extrait séquentiellement les lignes de ticks de la RAM d'origine, applique instantanément les décalages de prix et de temps requis par le bloc courant, et envoie directement le tick ajusté au Projet 2 sans aucune persistance sur disque.

---

## 6. Spécifications des Interfaces d'Entrée / Sortie (Contrat API)

### 6.1. Format requis en Entrée (Schéma des Parquets Quotidiens)
Le module exige des fichiers Apache Parquet journaliers structurés selon le schéma strict suivant :
`[timestamp, bidprice, askprice, bidvolume, askvolume, price_mid, spread, volume]`

### 6.2. Format garanti en Sortie (Interface de Flux P1 ➔ P2)
Le flux de ticks ajustés émis vers le Projet 2 reproduit exactement le schéma d'entrée, avec les garanties microstructurelles suivantes :
* `timestamp` : Axe temporel fictif, monotone, strictement croissant, unique.
* `bidprice` / `askprice` / `price_mid` : Translatés récursivement selon l'altitude de l'univers virtuel actuel.
* `spread` : Strictement égal au spread original (Invariant de liquidité).
* `volume` / `bidvolume` / `askvolume` : Identiques aux invariants d'activité historiques d'origine.

---

## 7. Pistes Futures

* **Parallélisation GPU de l'Ajustement à la Volée :** Implémenter les opérateurs de décalage cumulatif via des noyaux de calcul parallélisés (Cuda / OpenCL) si l'infrastructure doit traiter simultanément un panier volumineux de paires de devises en temps réel.
* **Génération d'Univers Hybrides via GANs :** Explorer à long terme l'utilisation de réseaux de neurones génératifs antagonistes (GANs) entraînés sur les 28 trajectoires de ticks pour synthétiser des micro-mouvements de prix encore jamais observés, enrichissant davantage la base d'évaluation.

---

## 8. Conclusion

Le Projet 1 remplit son rôle d'infrastructure de bas niveau en fournissant un environnement de simulation multi-univers mathématiquement cohérent. Grâce à la combinaison de la CPCV journalière, de la chaîne d'ajustement récursive et de la virtualisation en RAM, le Tick Engine fournit un flux d'une fluidité parfaite. Le Projet 3 pourra ainsi auditer la robustesse des configurations dans l'aveuglement le plus complet, sans qu'aucune contrainte glissante artificielle n'ait été imposée en amont. Le flux de ticks ainsi purifié et ajusté est prêt à être consommé par le Projet 2.

