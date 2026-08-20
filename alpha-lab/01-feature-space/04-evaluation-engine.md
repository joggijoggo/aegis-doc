# RAPPORT TECHNIQUE GLOBAL : PROJET 3 (EVALUATION & LEARNING SUITE)
## PARTIE 1 : FONDATIONS MICROSTRUCTURELLES ET PROBLÉMATIQUE BIDIRECTIONNELLE (SECTION 1/2)

**Statut du Document :** Manuel Directeur de Recherche et Spécifications Spatio-Temporelles  
**Période d'Étude :** Laboratoire In-Sample (1er Janvier 2019 - 31 Décembre 2022)  
**Composant Logiciel :** Projet 3 (Couche Haute / Intelligence Analytique)  
**Données d'Entrée :** Matrice Totale à 18 Dimensions de Barres Brutes (provenant du Projet 2)  

---

## Synthèse Managériale

Le **Projet 3 (Evaluation & Learning Suite)** incarne l'autorité scientifique supérieure et l'arbitre statistique final de notre écosystème de recherche quantitative. Placé en aval du constructeur d'histoires alternatives (Projet 1) et du moteur de compression passif (Projet 2), ce module a pour responsabilité unique d'auditer et de qualifier l'espace des caractéristiques candidates (Feature Space). Son objectif industriel est d'isoler des règles de comportement invariantes, dotées d'une convergence gaussienne et d'une stationnarité parfaites, capables d'affronter les futures ruptures de régimes du marché sans jamais contracter de biais de surapprentissage (backtest overfitting).

Pour éradiquer définitivement le **p-hacking de second niveau** — le processus par lequel un chercheur contamine les données futures en modifiant itérativement ses descripteurs après avoir constaté leur dégradation sur l'Out-Of-Sample —, le Projet 3 est soumis à un principe de sanctuarisation et d'aveuglement absolu. Il ignore totalement l'artifice géométrique et l'origine des 28 univers virtuels générés en RAM par le Projet 1. Il consomme chaque trajectoire synthétique de 3 ans comme une série temporelle réelle, unifiée, continue et fluide.

---

## 1. Introduction, Contexte et Problématique Bidirectionnelle

### 1.1. Le Contexte Microstructurel du Forex Spot
Le marché des changes de gré à gré (Forex Spot) est un environnement asynchrone, fragmenté et décentralisé. Contrairement aux marchés d'actions ou de contrats à terme centralisés (Futures), il ne possède ni volume officiel global, ni carnet d'ordres unifié (Level 2/3). Les données de Niveau 1 (Level 1) traitées par notre infrastructure se matérialisent par un flux hautement irrégulier de cotations (quotes) Bid/Ask émises par des réseaux de courtiers et de banques partenaires. 

L'échantillonnage de ce flux par une horloge chronologique rigide (ex: barres de 5 minutes) détruit les propriétés statistiques de la donnée en mélangeant des phases de léthargie extrême (session asiatique) et des pics d'activité massifs (sessions de Londres et New York intercalées).

### 1.2. La Motivation Scientifique : Briser l'Hétéroscédasticité
Les séries temporelles classiques souffrent structurellement de trois pathologies statistiques majeures qui invalident la quasi-totalité des modèles prédictifs standards : la non-stationnarité de la moyenne, l'hétéroscédasticité (grappes de volatilité ou volatility clustering) et la présence de queues de distribution épaisses (fat tails) matérialisées par un Kurtosis largement supérieur à 3. 

L'échantillonnage asynchrone paramétrique exécuté par le Projet 2 (Volume, Notional, Imbalance Bars) résout ce problème en avant-première en adaptant la vitesse de capture à l'intensité réelle du marché. La motivation du Projet 3 est de piloter ce processus de manière probabiliste afin de déterminer sous quelles conditions de granularité la donnée est forcée de converger vers une distribution stable.

### 1.3. La Problématique Double et Bidirectionnelle
La problématique d'ingénierie du Projet 3 se formalise par une double équation stochastique résolue de manière simultanée :
* **Le Sens Direct (Approche Ingénieur) :** Une structure d'échantillonnage étant fixée au niveau du Projet 2 (ex : une Volume Bar de taille V), quelle est la profondeur de mémoire d'apprentissage (T) et la fenêtre de transformation interne (W) minimales requises au niveau du Projet 3 pour stabiliser statistiquement le signal ?
* **Le Sens Inverse (Approche Chercheur) :** Une contrainte métier étant imposée (ex : "Je souhaite impérativement utiliser le rendement logarithmique pur"), quelle est la meilleure famille de barres asynchrones (Activité, Géométrie ou Information) et quel horizon de granularité (Scalping, Intraday ou Swing) maximisent la gaussianité et l'immunité face au glissement distributionnel (Feature Drift) ?

---

# RAPPORT TECHNIQUE GLOBAL : PROJET 3 (EVALUATION & LEARNING SUITE)
## PARTIE 1 : DUAL-PRICING ET TRANSFORMATION IDENTITAIRE (SECTION 2/2)

---

## 2. Évolution de l'Architecture des Prix : Du Mid-Price au Dual-Pricing

### 2.1. Le Choix Initial et sa Critique Microstructurelle
Aux prémices du projet, l'interface de données entre le moteur d'échantillonnage (Projet 2) et le laboratoire (Projet 3) reposait sur le format classique de l'OHLCV calculé sur le prix moyen abstrait (`price_mid`), adjointe d'une simple métrique moyenne d'asymétrie de la liquidité (`average_spread`).

L'analyse quantitative approfondie a révélé que ce choix initial représentait une dégradation destructive de l'information de Niveau 1 :
1. Sur le Forex Spot, le prix unique est une abstraction mathématique ; le marché n'existe que par la dualité permanente entre le cours acheteur (`Bid`) et le cours vendeur (`Ask`). 
2. Une moyenne arithmétique du spread (`average_spread`) efface la signature des pics d'écartement éphémères mais violents qui surviennent à l'intérieur d'une barre lors des chocs de liquidité (annonces macroéconomiques, bascules journalières).
3. Un modèle de simulation d'ordres réels (Projet 4) serait incapable de tester proprement des exécutions au marché ou des ordres limites, car les vraies barrières physiques d'achat (Ask) et de vente (Bid) étaient fusionnées.

### 2.2. Le Raffinement Final : La Matrice Totale à 18 Dimensions
Pour éliminer tout biais d'anachronisme et conserver l'intégralité du carnet Level 1, l'interface de sortie du Projet 2 a été restructurée pour livrer une structure non dégradée à 18 dimensions, dédoublant entièrement l'OHLCV et isolant les extrêmes de friction :
* **Géométrie du Prix Moyen (4 dimensions) :** `open_mid`, `high_mid`, `low_mid`, `close_mid`
* **Barrière Vendeuse Réelle (4 dimensions) :** `open_bid`, `high_bid`, `low_bid`, `close_bid`
* **Barrière Acheteuse Réelle (4 dimensions) :** `open_ask`, `high_ask`, `low_ask`, `close_ask`
* **Friction Intra-Barre (3 dimensions) :** `min_spread`, `max_spread`, `avg_spread`
* **Masse de Flux Brute (3 dimensions) :** `tick_count`, `total_volume`, `total_bidvolume`, `total_askvolume`

L'inclusion du `avg_spread` de confort, combinée à `min_spread` et `max_spread`, permet au Projet 3 de quantifier la *densité temporelle* du risque de liquidité. Le module distingue ainsi instantanément une barre ultra-liquide ayant subi un élargissement de spread d'une milliseconde sur son dernier tick (`min` = 1 pip, `max` = 10 pips, `avg` = 1.01 pip) d'une barre totalement bloquée dans une crise de contrepartie prolongée (`min` = 1 pip, `max` = 10 pips, `avg` = 9.85 pips).

---

## 3. Évolution des Transformations : L'Émergence de la Part Identitaire

### 3.1. Le Choix Initial et sa Critique Mathématique
La première version du pipeline imposait l'application systématique et standardisée de filtres de normalisation (tels que le Z-Score glissant ou la différenciation fractionnaire) sur toutes les caractéristiques candidates extraites par le Projet 3.

L'audit statistique du Feature Space a démontré que ce choix souffrait d'un biais directif lourd. Certaines caractéristiques microstructurelles avancées sont, par construction interne et de par leur nature mathématique intrinsèque, **déjà stationnaires et stables sous leur forme brute**. 
* L'Indice d'Asymétrie de Pression des Flux (Order Flow Imbalance Ratio - OFI), défini par le rapport volumétrique \(\frac{\text{BidVol} - \text{AskVol}}{\text{BidVol} + \text{AskVol}}\), est une variable structurellement bornée sur l'intervalle \([-1, 1]\).
* L'Entropie de Shannon appliquée à la distribution des signes des ticks à l'intérieur d'une barre est une mesure intrinsèquement confinée sur l'intervalle \([0, \ln(2)]\).

Appliquer un Z-Score glissant sur ces variables détruirait leur signification physique, effacerait la mémoire de leurs bornes naturelles et introduirait une instabilité numérique (division par un écart-type local proche de zéro en période de compression).

### 3.2. Le Raffinement Final : Le Triple Pipeline de Traitement
Le sous-module de transformation du Projet 3 a été restructuré pour intégrer obligatoirement trois branches comparatives :
1. **La Transformation Identitaire (I) :** La caractéristique brute est conservée intacte. Elle sert de benchmark de contrôle n°1 pour évaluer si l'échantillonneur asynchrone du Projet 2 (le paramètre V) suffit à lui seul à stabiliser la distribution.
2. **Le Z-Score Glissant (W) :** Centrage et réduction de la valeur vis-à-vis de sa moyenne (\(\mu\)) et de son écart-type (\(\sigma\)) calculés exclusivement sur une fenêtre glissante exprimée en *nombre de barres* (W), représentant la définition interne de la feature.
3. **La Différenciation Fractionnaire (d) :** Opérateur de dérivation d'ordre non entier permettant d'éliminer la racine unitaire tout en préservant le niveau maximum de mémoire historique à long terme.

### 3.3. Application Numérique du Pipeline d'Extraction (Le Rendement Logarithmique)
Pour matérialiser ce segment, observons le calcul d'une feature candidate courante : le **Rendement Logarithmique Pur (\(r_t\))** au pas t de l'Univers Alternatif n°14.

1. Réception des données brutes du Projet 2 :
* `close_mid` enregistré à la barre précédente (t-1) = **1.1000**
* `close_mid` enregistré à la barre actuelle (t) = **1.1011**
* Le Projet 3 extrait la feature brute via la branche Identitaire (I) :
  \[r_t = \ln\left(\frac{1.1011}{1.1000}\right) = \mathbf{+0.001000} \text{ (soit un décalage de +10 pips)}\]

2. Évaluation parallèle sur la branche Z-Score Glissant (Fidélité de feature W = 100 barres) :
L'historique des 100 barres passées au sein de cet univers affiche les constantes locales suivantes (calculées de manière vectorisée en RAM) :
* Moyenne locale des rendements (\(\mu\)) = **0.000100** (légère dérive haussière sur la fenêtre)
* Écart-type local des rendements (\(\sigma\)) = **0.000600** (régime de volatilité modérée)

Le module de traitement calcule la valeur standardisée finale de la feature au pas t :
\[Z_t = \frac{r_t - \mu}{\sigma} = \frac{0.001000 - 0.000100}{0.000600} = \frac{0.000900}{0.000600} = \mathbf{+1.500000}\]

La valeur brute de +10 pips est convertie en une coordonnée stochastique pure : le prix actuel s'écarte de +1,5 écart-type par rapport à sa dynamique récente. La série de ces coordonnées est transmise au moteur de validation pour l'optimisation.

---


# RAPPORT TECHNIQUE GLOBAL : PROJET 3 (EVALUATION & LEARNING SUITE)
## PARTIE 2 : STRUCTURE DE LA GRILLE PARAMÉTRIQUE LOGARITHMIQUE DISSYMÉTRIQUE (SECTION 1/2)

---

## 4. Structure de la Grille de Recherche : Pas Logarithmiques et Dissymétrie

### 4.1. Justification du Pas Logarithmique sur l'Axe V
L'axe V matérialise le seuil d'accumulation et de déclenchement mécanique de la barre gérée par le Projet 2 (qu'il s'agisse de barres de temps, de ticks, de volume fractionnaire ou de déséquilibre de flux). Pour explorer cet espace, le Projet 3 s'interdit d'utiliser un pas de progression linéaire constant. Il applique obligatoirement une progression géométrique ou logarithmique.

Cette contrainte repose sur deux réalités scientifiques incontestables :
1. **La Loi d'Échelle Microstructurelle (Sensibilité Proportionnelle) :** La sensibilité des séries financières aux changements de paramètres est fractale. Passer d'une taille de barre de V = 1 000 lots à V = 2 000 lots double la masse d'activité requise, modifiant de manière drastique la granularité des données et la forme de la distribution. À l'inverse, passer de V = 50 000 lots à V = 51 000 lots n'induit qu'un décalage mineur de 2 %, statistiquement invisible. Une grille linéaire sur-analyserait inutilement les régimes de basse fréquence (les grandes barres) au détriment des zones critiques de haute fréquence (les petites barres).
2. **L'Orthogonalité des Configurations (Élimination de la Redondance) :** Tester des paramètres linéaires ultra-proches génère des séries temporelles de barres corrélées à plus de 99,9 %. Si l'une de ces configurations passe les filtres statistiques par pure coïncidence locale, elle induit un biais de sélection. Le pas logarithmique force les jalons de la grille à être distincts, éradiquant la redondance informationnelle qui alimente le surapprentissage.

### 4.2. Justification de la Dissymétrie de la Grille et du Budget Maximum (M ≤ 100)
La grille bidimensionnelle construite par le laboratoire est volontairement asymétrique. Elle alloue une haute résolution à l'axe V (le paramètre d'infrastructure du Projet 2) et une basse résolution à l'axe W (la fenêtre de standardisation interne).

Cette dissymétrie est dictée par la nature des opérateurs :
* L'axe V est un opérateur géométrique non linéaire lourd qui transforme la structure intrinsèque de la donnée brute.
* L'axe W applique un opérateur de lissage statistique standardisé dont l'effet amortisseur est hautement linéaire et redondant sur des horizons proches.

Le budget total de la grille est strictement borné à M = V × W ≤ 100 configurations uniques. Si le chercheur s'autorisait une grille gigantesque, la probabilité mathématique de découvrir une configuration performante par pure chance stochastique tendrait vers 1. Brider la grille à un maximum de 100 combinaisons orthogonales permet de préserver la puissance du test statistique de protection (la correction de Bonferroni) sans paralyser la capacité de détection du laboratoire.


---

# RAPPORT TECHNIQUE GLOBAL : PROJET 3 (EVALUATION & LEARNING SUITE)
## PARTIE 2 : ECOSYSTÈME DU WALK-FORWARD GLISSANT INTRA-HISTOIRE (SECTION 2/2)

---

## 5. Le Pipeline de Walk-Forward Interne (Training Optimization)

Pour chacun des 28 univers alternatifs, le Projet 3 isole le sous-ensemble d'entraînement constitué des 6 blocs de données (soit 3 ans d'histoire permutée). Il y déploie un protocole d'optimisation glissante pas-à-pas (Walk-Forward Optimization - WFO).

### 5.1. Le Principe du Glissement Temporel Imbriqué
L'algorithme parcourt les 3 ans d'entraînement de manière purement chronologique selon un pas fixe (ex : incréments mensuels). À chaque étape t de la marche en avant, l'algorithme teste de manière comparative trois profondeurs distinctes de fenêtres d'apprentissage historiques, exprimées en temps macroéconomique (notées T, la mémoire d'apprentissage) :
* **T₁ (Horizon Court) :** Une fenêtre glissante de 3 mois de données (capturant la dynamique microstructurelle vive et les chocs de liquidité récents).
* **T₂ (Horizon Moyen) :** Une fenêtre glissante de 6 mois de données (capturant les cycles de volatilité inter-journaliers).
* **T₃ (Horizon Long) :** Une fenêtre glissante de 12 mois de données (garantissant la stabilité des tendances macroéconomiques lourdes).

Le système n'élit pas une seule fenêtre exclusive. Il évalue la qualité de la distribution sur le bloc de test interne qui suit immédiatement le segment d'apprentissage. L'algorithme combine ensuite les scores de performance des trois horizons par une agrégation linéaire. Ce protocole force le modèle à identifier un paramètre capable d'hybrider la réactivité des mémoires courtes et la robustesse des mémoires longues.

### 5.2. Batterie de Tests Éliminatoires Locaux
Lors du parcours du Walk-Forward, la série temporelle issue de chaque configuration (V, W) sur la fenêtre d'apprentissage T doit impérativement surmonter trois verrous mathématiques éliminatoires :

#### 1. Test de Stationnarité (Dickey-Fuller Augmenté - ADF)
Le module valide le retour à la moyenne de la caractéristique et l'absence de racine unitaire pour s'assurer que le signal ne dérive pas vers l'infini :
\[\Delta X_t = \alpha X_{t-1} + \sum_{i=1}^{p} \beta_i \Delta X_{t-i} + \epsilon_t\]
*Contrainte stricte :* Rejet de l'hypothèse nulle d'instabilité avec une p-value < 0.01.

#### 2. Test de Forme Distributionnelle (Jarque-Bera - JB)
Le module quantifie la proximité géométrique de la distribution par rapport à une distribution gaussienne parfaite en évaluant conjointement le Skewness (S, moment d'ordre 3) et le Kurtosis (K, moment d'ordre 4) :
\[JB = \frac{n}{6} \left( S^2 + \frac{1}{4}(K - 3)^2 \right)\]
*Contrainte stricte :* Non-rejet de l'hypothèse de normalité (p-value > alpha_ajusté).

#### 3. Test d'Homoscédasticité (Test de Levene)
Le module s'assure que la variance de la caractéristique reste constante et homogène d'un sous-segment à l'autre au sein de la fenêtre d'apprentissage, validant ainsi la capacité de l'échantillonneur asynchrone à absorber les pics de volatilité.

### 5.3. Application Numérique du Walk-Forward Interne
Observons le comportement du moteur de calcul à un pas t donné de la marche en avant au sein du sous-ensemble d'entraînement de l'Univers n°14, pour une fenêtre de standardisation fixée à W = 44 barres. L'algorithme évalue l'impact du paramètre V à l'entrée de son pas de test interne :

* **Configuration A : V = 1 000 lots (Horizon rapide / Haute granularité)**  
  Le Projet 2 génère un flux massif de barres très fines. La fenêtre d'apprentissage contient une densité importante de points de données, mais reste confinée à un historique très récent et nerveux.  
  *Résultats des tests :* Le test ADF confirme une stationnarité excellente car les données mutent à haute fréquence (Score_ADF = 0.99). En revanche, la distribution est polluée par le bruit de microstructure intra-journalier, générant un Kurtosis extrême (K = 5.4). Le test de Jarque-Bera rejette violemment la normalité (Score_JB = 0.12).  
  *Moyenne combinée des performances :* (0.99 + 0.12) / 2 = **0.555**

* **Configuration B : V = 5 000 lots (Horizon moyen / Échelle microstructurelle)**  
  Le Projet 2 filtre le bruit de basse microstructure grâce à des barres plus épaisses, permettant d'englober les transitions de sessions quotidiennes.  
  *Résultats des tests :* Le test ADF valide la stationnarité de la série (Score_ADF = 0.94). La forme de la distribution converge de manière spectaculaire vers une courbe de Gauss parfaite : le Skewness s'établit à -0.05 et le Kurtosis se stabilise à 3.15. La p-value de Jarque-Bera émise est excellente (Score_JB = 0.88).  
  *Moyenne combinée des performances :* (0.94 + 0.88) / 2 = **0.910**

Le laboratoire enregistre le score de chaque configuration pour ce pas temporel et fait progresser le Walk-Forward d'un mois. À la fin de la trajectoire d'entraînement, le système dispose d'une matrice complète de scores d'adaptation, prête pour la phase de sélection par plateau.

---


# RAPPORT TECHNIQUE GLOBAL : PROJET 3 (EVALUATION & LEARNING SUITE)
## PARTIE 3 : CRITÈRES DE SÉLECTION PAR PLATEAU DE STABILITÉ LOCALE (SECTION 1/2)

---

## 6. Évolution des Critères de Sélection : Du Maximum Local au Plateau de Stabilité

### 6.1. Le Choix Initial et sa Critique Mathématique
Dans les premières phases de conception de notre protocole, le critère d'élection du paramètre optimal au sein de la fenêtre d'apprentissage consistait à localiser de manière automatique la valeur de V qui maximisait de manière absolue la p-value du test de Jarque-Bera.

L'analyse statistique de la topologie des surfaces de réponse a mis en évidence le danger conceptuel de cette approche :
* Sélectionner la valeur crête (le "Max") d'une distribution de p-values introduit un biais d'overfitting localisé majeur. 
* En microstructure de marché, les sommets de performance isolés (les "pics") sont des artefacts stochastiques provoqués par le hasard d'un alignement géométrique des ticks sur une période donnée.
* Si la liquidité réelle décale ses flux de quelques fractions de lots le jour suivant, le pic s'effondre et la caractéristique dérive. Un paramètre de production doit être choisi pour sa tolérance au déplacement, et non pour sa performance absolue à un instant figé.

### 6.2. Le Raffinement Final : La Sélection par l'Entrée du Plateau
Pour forcer l'identification d'une structure invariante, l'algorithme de sélection du Projet 3 a été modifié pour rejeter les pics de performance étroits et rechercher des **plateaux de robustesse homogènes**.

L'algorithme se structure autour d'une logique en trois temps :
1. **Le Filtrage de Flottaison :** Le module isole l'ensemble des candidats V dont la p-value de Jarque-Bera est strictement supérieure au seuil critique ajusté par la correction de Bonferroni. Tout paramètre sous la ligne de flottaison statistique est éliminé.
2. **Le Calcul du Gradient Local :** Pour les points restants, l'algorithme calcule la pente (la dérivée première $\frac{dp}{dV}$) et la courbure (la dérivée seconde $\frac{d^2p}{dV^2}$) de la courbe de p-value.
3. **La Localisation du Point d'Inflexion Inférieur :** Le Projet 3 sélectionne **la valeur de V la plus petite possible** située au point d'inflexion marquant l'entrée d'un plateau plat (une zone où la p-value reste stable et où la variance locale de cette p-value est quasi nulle).

Ce raffinement garantit un double bénéfice : il assure une immunité mathématique face aux micro-déplacements de liquidité (le paramètre est assis au milieu d'une zone stable), tout en préservant la plus haute granularité informative (en retenant le V minimal, on conserve les barres les plus fines possibles, maximisant le nombre de points disponibles pour les modèles en aval).


---


# RAPPORT TECHNIQUE GLOBAL : PROJET 3 (EVALUATION & LEARNING SUITE)
## PARTIE 3 : RÉGULARISATION PAR LA VOLATILITÉ LOGARITHMIQUE COMPRESSÉE (SECTION 2/2)

---

## 7. Évolution de la Fonction d'Objectif : Intégration de la Volatilité Compressée

### 7.1. Le Choix Initial et sa Critique Statistique
Pour consolider les enveloppes minimales stables $V^*_t$ identifiées à chaque étape t du Walk-Forward et en extraire la configuration unique à projeter sur le Test Out-Of-Sample de l'univers, la méthode intermédiaire consistait à appliquer une moyenne pondérée par la volatilité brute du marché ($\sigma_t$).

L'analyse de la distribution de la volatilité a invalidé cette approche :
* Sur le Forex Spot, la volatilité ne progresse pas sur une échelle linéaire ; elle suit une loi de puissance à queues épaisses (*Pareto-like*). 
* Lors d'un événement macroéconomique majeur ou d'une crise de liquidité aiguë, la volatilité brute explose de manière géométrique, multipliant sa valeur par un facteur 10, 20 ou 50 par rapport à son bruit de fond habituel.
* Si l'on injecte cette volatilité brute dans le calcul des poids ($w_t = \sigma_t / \sum \sigma_i$), un choc exogène de deux semaines va capter à lui seul plus de 60 % de la puissance de décision sur un historique d'apprentissage de 3 ans.

Le modèle souffrait d'un effet "trou noir" : il sur-dimensionnait le paramètre final $V^*$ pour se protéger d'une crise transitoire, ce qui stérilisait totalement la donnée (génération de barres géantes à perte massive de caractéristiques et de granularité) pendant les 95 % de temps où le marché était calme.

### 7.2. Le Raffinement Final : La Compression Logarithmique Positive
Normaliser la volatilité par un Z-score classique est mathématiquement proscrit, car cela génère des valeurs négatives pour les périodes sous la moyenne, détruisant la cohérence d'une moyenne pondérée d'infrastructures. La solution d'ingénierie retenue par le Projet 3 applique une **transformation de compression logarithmique non linéaire** sur la volatilité brute (mesurée via la variance intrinsèque intra-barre de Garman-Klass) :
$$\text{Score}_t = 10 + \ln(\sigma_{GK, t})$$
$$w_t = \frac{\text{Score}_t}{\sum_{i=1}^{T} \text{Score}_i}$$
$$V^*_{\text{final}} = \sum_{t=1}^{T} (V^*_t \times w_t)$$

Cette formulation bride mathématiquement l'impact des crises aiguës en ramenant les lois de puissance à des rapports d'échelle gérables. Le krach conserve son rôle de signal de risque et surélève modérément le paramètre pour sécuriser le système, mais il perd son pouvoir de neutralisation statistique du comportement normal du marché.

### 7.3. Application Numérique de la Fonction d'Objectif Régularisée
Observons le comportement du moteur d'agrégation au sein du bloc d'entraînement de l'Univers n°14, scindé pour la démonstration en 4 périodes temporelles distinctes.

1. Constatations des Enveloppes Minimales et de la Volatilité Brute :
* **Période 1 (Calme Plat) :** Volatilité $\sigma_1 = 0.0005$. Le point d'inflexion stable de convergence de la feature est bas : $V^*_1 = \mathbf{2\,000\text{ lots}}$.
* **Période 2 (Tension standard) :** Volatilité $\sigma_2 = 0.0015$. Le seuil de convergence grimpe à $V^*_2 = \mathbf{6\,000\text{ lots}}$.
* **Période 3 (Krach Systémique) :** Volatilité $\sigma_3 = 0.0050$ (Explosion de facteur 10). L'enveloppe de sécurité exige des barres larges : $V^*_3 = \mathbf{20\,000\text{ lots}}$.
* **Période 4 (Régime Directionnel) :** Volatilité $\sigma_4 = 0.0020$. Le point d'inflexion s'établit à $V^*_4 = \mathbf{8\,000\text{ lots}}$.

2. Effet de l'Atténuation Logarithmique sur l'Allocation des Poids :
Si nous avions utilisé la volatilité brute, la masse globale aurait été de 0.0090, attribuant un poids de **55,56 %** au seul mois de krach ($0.0050 / 0.0090$), tandis que la période calme n'aurait pesé que **5,56 %**. 

Appliquons le filtre de compression logarithmique positive de notre spécification :
* $\text{Score}_1 = 10 + \ln(0.0005) = 10 - 7.60 = \mathbf{2.40} \implies w_1 = 2.40 / 14.39 = \mathbf{16.68\%}$
* $\text{Score}_2 = 10 + \ln(0.0015) = 10 - 6.50 = \mathbf{3.50} \implies w_2 = 3.50 / 14.39 = \mathbf{24.32\%}$
* $\text{Score}_3 = 10 + \ln(0.0050) = 10 - 5.30 = \mathbf{4.70} \implies w_3 = 4.70 / 14.39 = \mathbf{32.66\%}$
* $\text{Score}_4 = 10 + \ln(0.0020) = 10 - 6.21 = \mathbf{3.79} \implies w_4 = 3.79 / 14.39 = \mathbf{26.34\%}$
* *Masse totale cumulée :* $\sum \text{Score} = \mathbf{14.39}$

Le krach est bridé à un poids de **32,66 %**, redonnant une voix statistique légitime (**16,68 %**) à la période de calme.

3. Calcul du Paramètre Élu Final ($V^*_{\text{final}}$) :
$$V^*_{\text{final}} = (2\,000 \times 0.1668) + (6\,000 \times 0.2432) + (20\,000 \times 0.3266) + (8\,000 \times 0.2634)$$
$$V^*_{\text{final}} = 333.60 + 1\,459.20 + 6\,532.00 + 2\,107.20 = \mathbf{10\,432\text{ lots}}$$

L'algorithme retient la valeur théorique de 10 432 lots. Le compromis d'ingénierie est validé : le paramètre s'est surélevé pour garantir la normalité de la feature face aux crises futures ($10.4k > 4k$), mais a refusé la paranoïa de se figer sur le maximum absolu d'outlier ($10.4k < 20k$). Le pipeline conserve sa réactivité.

---

# RAPPORT TECHNIQUE GLOBAL : PROJET 3 (EVALUATION & LEARNING SUITE)
## PARTIE 4 : CRASH-TEST OUT-OF-SAMPLE ÉTANCHE ET MÉTA-ÉLECTION PAR PÉNALISATION PSI (SECTION 1/2)

---

## 8. Le Protocole de Crash-Test sur les Blocs de Test (Testing Out-Of-Sample)

Le triplet de paramètres optimal ($V^*_i, W^*_i, T^*_i$) étant élu, figé et verrouillé à la fin de la phase d'optimisation glissante sur l'ensemble d'entraînement de l'Univers Alternatif courant, le Projet 3 bascule instantanément sur sa phase la plus critique : l'évaluation en aveugle sur son segment **Out-Of-Sample (OOS) propre**.

### 8.1. Imperméabilité et Aveuglement Technologique
Ce segment est constitué des **2 blocs de 6 mois restants** (soit 1 an de données de trading continu pures) qui ont été rigoureusement mis de côté par le Projet 1 pour cet univers spécifique. 
* **Contrainte de Non-Ajustement :** Le module instancie la configuration élue de manière purement passive. Aucun ré-apprentissage, aucun ajustement et aucune modification de paramètre n'est tolérée sur ce segment.
* **La Sanction de la Dérive :** Si le couple optimal ($V^*_i, W^*_i$) était le fruit d'un surapprentissage localisé sur les blocs d'entraînement, la structure de la distribution s'effondrera immédiatement lors de la confrontation avec ce "futur virtuel" étanche.

### 8.2. La Métrique Arbitre : Le Population Stability Index (PSI)
Pour quantifier de manière non paramétrique la dérive de la caractéristique (*Feature Drift*) entre l'environnement d'apprentissage (Train) et l'environnement de validation (Test OOS), le Laboratoire utilise le **Population Stability Index (PSI)**. 

Le spectre de la feature est segmenté en $k$ seaux statistiques orthogonaux (*buckets*). Le PSI mesure l'entropie de glissement relative des fréquences observées face aux fréquences attendues :
$$PSI_i = \sum_{j=1}^{k} \left( \% \text{Actual}_j - \% \text{Expected}_j \right) \times \ln\left(\frac{\% \text{Actual}_j}{\% \text{Expected}_j}\right)$$
Où $\% \text{Expected}_j$ représente la fréquence de la caractéristique dans le seau $j$ sur l'ensemble d'entraînement, et $\% \text{Actual}_j$ la fréquence observée sur le bloc de Test OOS.

---

## 9. La Méta-Élection Continue et la Pénalisation par le PSI

### 9.1. Le Principe de Neutralité et le Rejet du Choc Volatiel Transversal
Pour la sélection finale du triplet de paramètres unifié destiné à la production, toute pondération basée sur la volatilité globale à l'échelle des univers est formellement proscrite. L'échec est structurellement neutre : une dérive de distribution peut tout aussi bien frapper un univers calme en phase de compression qu'un univers de panique. Seul le verdict objectif du PSI dicte la pénalisation.

### 9.2. L'Algorithme de Vote Démocratique par la Performance
Chaque univers $i$ (de 1 à 28) dispose d'une seule voix. Sa force lors de l'arbitrage transversal est dictée par une fonction de pénalisation exponentielle indexée sur son propre score PSI obtenu lors du crash-test individuel :
$$\text{Facteur Performance}_i = \exp\left( - \lambda \times PSI_i \right)$$

Le Poids de Confiance Réduit ($W_{\text{confiance}, i}$) attribué au choix de chaque univers s'obtient en normalisant ce facteur sur la masse globale du conclave :
$$W_{\text{confiance}, i} = \frac{\text{Facteur Performance}_i}{\sum_{j=1}^{28} \text{Facteur Performance}_j}$$

Où $\lambda$ est le coefficient de sévérité algorithmique fixé à 10. Les valeurs méta-optimales continues sont extraites par le produit matriciel des choix locaux et de ce vecteur de confiance :
$$V_{\text{méta}} = \sum_{i=1}^{28} \left( V^*_i \times W_{\text{confiance}, i} \right)$$
$$W_{\text{méta}} = \sum_{i=1}^{28} \left( W^*_i \times W_{\text{confiance}, i} \right)$$

---

# RAPPORT TECHNIQUE GLOBAL : PROJET 3 (EVALUATION & LEARNING SUITE)
## PARTIE 4 : VECTORS SÉCURISÉS ET VERDICT CRASH-TEST GLOBAL (SECTION 2/2)

---

### 9.3. Application Numérique : Construction Étanche des Valeurs Méta

Pour comprendre la cinématique de cette méta-élection, simulons les résultats obtenus par nos 28 univers lors de leurs crash-tests individuels. Notre grille d'exploration d'infrastructure pour $V$ comportait les jalons fixes [4k lots, 8k lots, 16k lots] et l'axe de fidélité $W$ comportait [20 barres, 60 barres, 100 barres].

#### 1. Recueil des Voix du Conclave :
* **Groupe 1 (20 Univers Robustes) :** Ont validé leur test OOS avec un $PSI = \mathbf{0.02}$. Ils proposent tous le couple optimal : $V^* = 4\,000 \text{ lots}$ et $W^* = 40 \text{ barres}$.
* **Groupe 2 (5 Univers Intermédiaires) :** Ont validé leur test OOS avec un $PSI = \mathbf{0.05}$. Ils proposent tous le couple optimal : $V^* = 8\,000 \text{ lots}$ et $W^* = 60 \text{ barres}$.
* **Groupe 3 (3 Univers Récalcitrants) :** Ont échoué à leur test OOS individuel avec un $PSI = \mathbf{0.25}$. Ils proposent tous le couple : $V^* = 8\,000 \text{ lots}$ et $W^* = 60 \text{ barres}$.

#### 2. Calcul des Facteurs de Performance Individuels (Pénalisation Exponentielle $\lambda = 10$) :
* Pour un univers du Groupe 1 : $\exp(-10 \times 0.02) = \exp(-0.2) = \mathbf{0.8187}$
* Pour un univers du Groupe 2 : $\exp(-10 \times 0.05) = \exp(-0.5) = \mathbf{0.6065}$
* Pour un univers du Groupe 3 : $\exp(-10 \times 0.25) = \exp(-2.5) = \mathbf{0.0820}$ *(Note : l'échec subit une décote immédiate de 90 % de sa force politique).*

#### 3. Calcul de la Masse des Facteurs (Dénominateur Commun) :
$$\sum \text{Facteurs} = (20 \times 0.8187) + (5 \times 0.6065) + (3 \times 0.0820) = 16.374 + 3.0325 + 0.246 = \mathbf{19.6525}$$

#### 4. Calcul des Poids de Confiance Relatifs ($W_{\text{confiance}}$) :
* Poids global accordé au Groupe 1 : $16.374 / 19.6525 = \mathbf{83.32\%}$
* Poids global accordé au Groupe 2 : $3.0325 / 19.6525 = \mathbf{15.43\%}$
* Poids global accordé au Groupe 3 : $0.246 / 19.6525 = \mathbf{1.25\%}$

#### 5. Synthèse Mathématique Continue des Valeurs Méta :
Appliquons ce vecteur de confiance aux propositions physiques du conclave :
$$V_{\text{méta}} = (4\,000 \times 0.8332) + (8\,000 \times 0.1543) + (8\,000 \times 0.0125) = \mathbf{4\,667.20\text{ lots}}$$
$$W_{\text{méta}} = (40 \times 0.8332) + (60 \times 0.1543) + (60 \times 0.0125) = \mathbf{43.336\text{ barres}}$$

#### 6. Rupture de la Grille Fixe et Arrondi Logique :
Le Projet 3 s'interdit d'interpoler ou de rejeter ces valeurs exactes vers les jalons d'origine (4k ou 8k). Il conserve la quintessence du vote continu et procède à un arrondi logique et propre :
* $V_{\text{méta}}$ unifié et arrêté = **4 700 lots**
* $W_{\text{méta}}$ unifié et arrêté = **44 barres**
* $T_{\text{méta}}$ (la rolling window temporelle glissante d'apprentissage) s'établit fermement à **6 mois**.

---

## 10. Le Grand Crash-Test Final "In-Sample" (L'Étape C)

Ce triplet n'appartenant à aucun jeu de données précalculé, le Projet 3 bascule en mode dynamique pour valider l'universalité de sa règle :
1. **Génération Synchrone :** Le Projet 3 ordonne au Bar Engine (Projet 2) de reconstruire physiquement en RAM la série temporelle exacte et sur-mesure pour un seuil fixe de **4 700 lots** sur l'intégralité de la période 2019-2022 à travers les 28 univers.
2. **Calcul de la Feature :** Le Projet 3 applique la transformation de Z-Score glissant sur la fenêtre stricte de **44 barres**, en rafraîchissant dynamiquement ses paramètres d'apprentissage ($\mu, \sigma$) sur un horizon glissant de **6 mois ($T_{\text{méta}}$)**.
3. **Audit OOS Transversal :** L'algorithme isole les 28 portions de Test Out-Of-Sample (la dernière année de données de chaque univers) et y calcule les 28 nouveaux scores PSI.

#### Évaluation Numérique du PSI sur l'OOS de l'Univers n°14 :
L'espace des Z-scores généré à 4 700 lots est segmenté en 4 seaux cardinaux pour vérifier sa stabilité distributionnelle :
* *Seau 1 ($Z < -2$) :* Fréquence Attendue (E) = 5.0% | Fréquence Observée (A) = 5.2%  
  $\text{Morceau}_1 = (0.052 - 0.050) \times \ln(0.052 / 0.050) = \mathbf{0.000078}$
* *Seau 2 ($-2 \le Z < 0$) :* Fréquence Attendue (E) = 45.0% | Fréquence Observée (A) = 43.1%  
  $\text{Morceau}_2 = (0.431 - 0.450) \times \ln(0.431 / 0.450) = \mathbf{0.000819}$
* *Seau 3 ($0 \le Z < +2$) :* Fréquence Attendue (E) = 45.0% | Fréquence Observée (A) = 46.5%  
  $\text{Morceau}_3 = (0.465 - 0.450) \times \ln(0.465 / 0.450) = \mathbf{0.000492}$
* *Seau 4 ($Z \ge +2$) :* Fréquence Attendue (E) = 5.0% | Fréquence Observée (A) = 5.2%  
  $\text{Morceau}_4 = (0.052 - 0.050) \times \ln(0.052 / 0.050) = \mathbf{0.000078}$

$$PSI_{\text{total Univers 14}} = 0.000078 + 0.000819 + 0.000492 + 0.000078 = \mathbf{0.001467}$$

#### Le Verdict d'Acceptation Globale :
Le score $PSI$ sur l'Univers 14 est parfait ($0.0014 < 0.10$). L'algorithme calcule la moyenne arithmétique de ces scores à travers les 28 portions de Test pures. Si ce $PSI_{\text{moyen transversal}}$ reste strictly contenu sous la barrière des **0.10**, la caractéristique `log_return` est **définitivement validée, qualifiée et déclarée immortelle face au drift**.

---

## 11. Conclusion du Manuel de Recherche In-Sample

Le Projet 3 achève avec une rigueur absolue le cycle d'ingénierie statistique de la période In-Sample. En transformant la recherche de paramètres en une quête continue de la coordonnée unifiée $(V, W, T)$ dictée exclusivement par la pénalisation empirique du PSI, le système bâtit un rempart inattaquable contre la chance et le surapprentissage. 

Les caractéristiques candidates qui ressortent officiellement qualifiées de ce double filtre d'aveuglement ont prouvé leur capacité géométrique à lisser le bruit microstructure l et leur vitesse d'apprentissage à absorber les chocs macroéconomiques. Les fondations de l'infrastructure de recherche sont désormais scellées. Le Cahier des Charges théorique est clos. Le pipeline est prêt à être transposé sous forme de code opérationnel.

