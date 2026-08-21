## Bloc 1 : Les Fondations Intuitives & Le Modèle Linéaire Simple
Ce bloc pose l'imagerie mentale, géométrique et probabiliste sans la lourdeur des matrices. Il introduit la méthode d'estimation sur deux variables (X et Y) pour comprendre la mécanique fondamentale avant de généraliser.

### 1. Spécification & Intuition Géométrique (Modèle à Deux Variables)
* Comprendre la notion de "distance" entre des données réelles et une droite théorique. Écriture scalaire du modèle simple : y = a + bx + e.
* Fil rouge Trading : Comment tracer la "droite parfaite" au milieu d'un nuage de points représentant le prix de deux devises fortement liées (ex: EUR/USD et GBP/USD) pour repérer visuellement les écarts de prix.

### 2. Formalisme Scalaire & Minimisation (Les Sommes)
* Dérivation de l'estimateur des Moindres Carrés Ordinaires (MCO) par le calcul différentiel classique (sans matrice). Minimisation de la somme des résidus au carré. Propriété algébrique fondamentale : la droite passe par le point moyen (X_bar, Y_bar).
* Fil rouge Trading : Calculer manuellement la sensibilité (le coefficient de corrélation b) d'une devise par rapport à une autre et isoler la constante (a), le point d'ancrage de notre spread.

### 3. Intuition Probabiliste & Inférence Fondamentale
* Qu'est-ce qu'une distribution ? Comprendre la Loi Normale comme la représentation graphique des variations quotidiennes d'un marché stable. Introduction aux résidus et aux hypothèses statistiques de base (erreur centrée, variance constante).
* Fil rouge Trading : Différencier une variation de prix qui relève du hasard (bruit de marché) d'un mouvement anormal qui indique le début d'une vraie tendance (signal). Comprendre pourquoi une moyenne mobile est en retard et pourquoi l'analyse des résidus est plus fine.

### Mini-Projet 1 : Le Cahier des Charges de la Régression Bivariée
* Fil rouge Trading : Concevoir un script conceptuel d'arbitrage de deux devises. Vous devez définir graphiquement les critères géométriques d'un écart de prix anormal à l'aide des résidus scalaires, modéliser leur distribution sous l'hypothèse de bruit blanc, et lister les limites d'un indicateur technique classique (type RSI) face à cette droite de régression.

## Bloc 2 : Le Paradigme Linéaire Global (Transition Matricielle & Modèle Multiple)
Ce bloc transpose le modèle simple dans l'espace de l'algèbre linéaire et généralise la régression à N variables explicatives simultanées.

### 1. Fondations Théoriques & Spécification Matricielle
* Traduction du modèle simple sous forme vectorielle, puis écriture du modèle multiple généralisé : Y = Xb + e. Analyse exhaustive des hypothèses de Gauss-Markov sous forme matricielle et démonstration du théorème BLUE.
* Fil rouge Trading : Spécification du modèle d'évaluation des actifs (CAPM multi-indices). Vous cherchez à expliquer les mouvements d'une de vos devises en fonction de plusieurs indices macroéconomiques simultanés (Dollar Index, prix des matières premières, différentiels de taux).

### 2. Formalisme Mathématique & Estimation Globale
* Dérivation matricielle du vecteur d'estimation MCO : b_hat = (X^T X)^(-1) X^T Y. Approche géométrique par les matrices de projection orthogonale (matrice chapeau H et matrice résiduelle M). Décomposition de la variance (Théorème de Cochran).
* Fil rouge Trading : Algorithme d'isolation de l'Alpha (a) et des Betas (b_i). Les mathématiques des projections servent ici à projeter le prix de la devise sur le sous-espace des indices pour calculer l'Alpha structurel, qui représente votre avantage statistique théorique.

### 3. Inférence & Validation Statistique en Haute Dimension
* Construction des tests d'hypothèses linéaires multivariés : statistique de Student (t-test) pour les restrictions individuelles et statistique de Fisher (F-test) pour les restrictions conjointes. Analyse de la variance résiduelle non biaisée s^2 et du R^2 ajusté.
* Fil rouge Trading : Validation des signaux. Les tests d'hypothèses servent à prouver statistiquement si l'un de vos indices macroéconomiques a un impact réel et persistant sur le prix de la devise (t-test), ou si la combinaison globale de vos indicateurs apporte une vraie information (F-test) ou n'est que du bruit de données.

### Mini-Projet 2 : Le Filtre à Alpha Multi-Indicateurs
* Fil rouge Trading : Formaliser un modèle d'évaluation de la juste valeur d'une devise (ex: EUR/USD). Dériver sous forme matricielle les conditions du premier ordre pour obtenir le vecteur b_hat, projeter les prix sur la matrice chapeau H pour isoler l'Alpha structurel, et valider par des tests de Student et de Fisher quels indicateurs macroéconomiques sont mathématiquement significatifs pour expliquer le prix.

## Bloc 3 : Pathologies des Données & Variables Qualitatives
Ce bloc lève les hypothèses restrictives du modèle linéaire pour adapter les formules mathématiques à la réalité chaotique des séries financières.

### 1. Fondations Théoriques & Spécification
* Introduction de la non-sphéricité des erreurs (hétéroscédasticité) et de la non-linéarité. Modélisation des choix qualitatifs : spécification des modèles à variable dépendante binaire (Logit et Probit via l'indice latent).
* Fil rouge Trading : Intégration des anomalies de marché. Les marchés financiers violent les règles classiques (la volatilité change tout le temps, les chocs s'accumulent). On introduit aussi des variables binaires pour capturer l'impact des fenêtres temporelles (ex: ouverture de la session de New York = 1, sinon 0).

### 2. Formalisme Mathématique & Estimation
* Dérivation de l'estimateur des Moindres Carrés Généraux (MCG) pour corriger la structure des erreurs : b_MCG = (X^T Omega^(-1) X)^(-1) X^T Omega^(-1) Y. Principes de l'estimation par Maximum de Vraisemblance (MV) pour les modèles Logit/Probit. Théorie asymptotique (consistance et normalité asymptotique via les estimateurs robustes de White).
* Fil rouge Trading : Robustesse algorithmique. Vous apprenez à modifier mathématiquement les calculs du modèle pour que vos estimations de tendance ne deviennent pas biaisées ou totalement "aveugles" lors des pics de volatilité.

### 3. Inférence & Validation Statistique
* Tests d'hétéroscédasticité (Breusch-Pagan, White). Inférence sous MV (test du Ratio de Vraisemblance, test de Wald). Métriques de classification pour choix qualitatifs : Pseudo-R^2, table de confusion, et analyse de la courbe ROC/AUC.
* Fil rouge Trading : Prédire un retournement de tendance. Les modèles Logit/Probit servent à calculer une métrique exacte : "Probabilité mathématique que le marché se retourne dans la prochaine heure (0 à 100%)". La courbe ROC valide si cette prédiction est statistiquement fiable pour déclencher un ordre.

### Mini-Projet 3 : L'Algorithme de Classification des Chocs de Volatilité
* Fil rouge Trading : Construire un modèle probabiliste de retournement de marché. Modéliser une variable binaire Y_t (1 si retournement de tendance, 0 sinon). Formaliser l'estimation par Maximum de Vraisemblance du modèle Probit, corriger la matrice de variance-covariance Omega pour intégrer l'hétéroscédasticité des sessions de trading, et valider le pouvoir prédictif du modèle via la fonction mathématique de la courbe ROC.

## Bloc 4 : Dynamique Temporelle & Systèmes Équationnels
Ce bloc est dédié à l'étude des données chronologiques, de la dépendance temporelle (les autorégresseurs) et de la modélisation mathématique du risque.

### 1. Fondations Théoriques & Spécification
* Processus stochastiques : concepts de stationnarité et d'ergodicité. Spécification des processus univariés : structures Autorégressives AR(p), Moyennes Mobiles MA(q) et structures mixtes ARMA/ARIMA. Modélisation de la variance conditionnelle temporelle via les structures ARCH et GARCH.
* Fil rouge Trading : Coder la mémoire des prix. C'est ici que vous introduisez les autorégresseurs. Vous apprenez à modéliser le prix d'une devise à l'instant t uniquement basé sur ses propres bougies fermées passées (H1/H4). Le modèle GARCH sert à modéliser et anticiper les cycles de forte volatilité du marché.

### 2. Formalisme Mathématique & Estimation
* Équations de Yule-Walker pour la détermination des paramètres des processus AR. Conditions de stationnarité via les racines des polynômes de retard. Écriture de la fonction de vraisemblance conditionnelle pour un modèle GARCH(1,1).
* Fil rouge Trading : Calcul de la persistance de tendance. Vous devez résoudre les équations pour savoir si la tendance passée s'amortit rapidement ou si elle a une forte persistance. Le calcul GARCH estime mathématiquement la distribution probable du prix à la prochaine bougie.

### 3. Inférence & Validation Statistique
* Analyse des fonctions d'autocorrélation (FAC/FACP). Critères d'information pour la sélection d'ordre (AIC/BIC). Tests de racine unitaire (Dickey-Fuller Augmenté, Phillips-Perron) pour valider la stationnarité.
* Fil rouge Trading : Moteur de décision du Bot. L'AIC/BIC détermine automatiquement combien de bougies passées (lags) le bot doit analyser pour être optimal sans surapprendre. Le calcul GARCH dicte mathématiquement le Position Sizing : si le risque calculé augmente, le bot réduit la taille du trade pour protéger le capital.

### Mini-Projet 4 : Le Noyau Prédictif ARMA-GARCH avec Gestion du Risque
* Fil rouge Trading : Concevoir le moteur mathématique principal d'un bot de Day-Trading. Écrire le système d'équations d'un modèle conjoint ARMA(p,q)-GARCH(1,1). Utiliser le critère BIC pour déterminer analytiquement le nombre optimal de retards, résoudre les conditions de stationnarité sur les polynômes de retard, et formuler l'équation de Position Sizing qui réduit la taille des positions dès que la variance conditionnelle GARCH augmente.

## Bloc 5 : Données de Structure, Cointégration & Haute Dimension
Ce bloc aborde les structures multi-actifs, le problème de la causalité statistique, et le traitement des modèles contenant un très grand nombre de variables.

### 1. Fondations Théoriques & Spécification
* Le problème de l'endogénéité (simultanéité, variables omises). Théorie de la cointégration de Granger. Économétrie en haute dimension : le problème du surapprentissage (overfitting) lorsque le nombre de régresseurs augmente fortement.
* Fil rouge Trading : Co-mouvements et sélection d'indicateurs. En Day/Swing trading, les devises bougent en paniers liés. On cherche des relations de long terme. On cherche également à utiliser des dizaines d'indicateurs techniques simultanés sans saturer le modèle.

### 2. Formalisme Mathématique & Estimation
* Méthode des Variables Instrumentales (VI) et Doubles Moindres Carrés (D2MC). Spécification du Modèle Vectoriel à Correction d'Erreur (VECM). Estimation régularisée (pénalisation de la fonction de perte MCO) : formulation mathématique du Lasso (norme L1), du Ridge (norme L2), et de l'ElasticNet.
* Fil rouge Trading : Stratégie d'Arbitrage Statistique & Filtrage. Vous utilisez les équations du VECM pour concevoir une stratégie de Pairs Trading. Le modèle calcule l'écart d'équilibre (spread) entre deux actifs cointégrés. Le Lasso sert de filtre : il élimine le bruit en forçant à zéro les coefficients de tous les indicateurs techniques inutiles pour ne garder que ceux qui ont un vrai pouvoir prédictif instantané.

### 3. Inférence & Validation Statistique
* Tests de cointégration (approche d'Engle-Granger, procédure de Johansen). Test de causalité de Granger. Évaluation en haute dimension : validation croisée (K-fold), analyse de l'erreur quadratique moyenne hors-échantillon (Out-of-sample MSE) et tests de stabilité (CUSUM).
* Fil rouge Trading : Backtesting mathématique. Avant toute exécution, les tests de Johansen valident si la stratégie d'arbitrage repose sur une vraie relation mathématique stable ou sur une simple corrélation éphémère. L'analyse du MSE hors-échantillon simule mathématiquement la robustesse du bot face à des données de marché totalement inédites.

### Mini-Projet 5 : L'Architecture Finale de l'Arbitrage Statistique Régularisé
* Fil rouge Trading : Modéliser un système de trading de type Pairs Trading sur un panier de devises majeures. Appliquer la procédure de Johansen pour prouver la cointégration à long terme entre les actifs, spécifier la matrice de transition du modèle VECM pour capturer la vitesse de retour à la moyenne du spread, et injecter une pénalisation Lasso pour filtrer un grand nombre d'indicateurs macroéconomiques afin de valider la stabilité du modèle hors-échantillon (Out-of-sample MSE).
