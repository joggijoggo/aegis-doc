# LE PARCOURS DU QUANT : PLAN GLOBAL ET FEUILLE DE ROUTE IMMUNITAIRE

## PARTIE 1 : LES FONDATIONS MATHÉMATIQUES ET STATISTIQUES

### 1. Les Mathématiques Linéaires (Prérequis 1)
*   **Vecteurs et Espaces de Marché**
    *   Norme et produit scalaire → *Calculer la distance géométrique et la similarité cosinus entre deux vecteurs de rendements d'actifs.*
    *   Combinaisons linéaires et indépendance → *Vérifier si une nouvelle stratégie apporte de l'alpha ou duplique des facteurs existants.*
    *   Base et dimension spatiale → *Réduire la dimensionnalité des indicateurs techniques pour éliminer la redondance des données.*
*   **Calcul Matriciel de Base**
    *   Définition des structures bidimensionnelles (Matrices) → *Organiser les données de marché en grilles lignes/colonnes (jours x actifs).*
    *   Opérations élémentaires sur le papier → *Apprendre la somme de matrices et la multiplication d'une matrice par un scalaire (poids).*
    *   Le produit Matrice-Vecteur et Matrice-Matrice → *Calculer le rendement historique d'un portefeuille complet ligne par colonne.*
    *   Transposition et trace → *Extraire et manipuler les structures symétriques sur le papier.*
*   **Algèbre Matricielle Avancée**
    *   Le Déterminant d'une matrice carrée → *Vérifier géométriquement si un système d'équations de marché est singulier ou s'effondre à zéro.*
    *   Le concept mathématique de l'Inversion ($X^{-1}$) → *Comprendre les conditions d'existence de l'inverse (Rang plein) pour résoudre des systèmes.*
    *   Les règles de dérivation vectorielle et matricielle → *Apprendre à dériver une forme linéaire ou quadratique par rapport à tout un vecteur de paramètres.*
*   **Formes Quadratiques et Convexité**
    *   Matrices définies positives → *S'assurer que la matrice de risque (covariance) permet de trouver un minimum global strict (forme en bol).*
    *   Analyse Convexe et Optimisation sous contraintes → *Le thésrème des Multiplicateurs de Lagrange sur le papier pour l'allocation de Markowitz.*
    *   Résolution analytique instantanée → *Développer des algorithmes d'optimisation pour rééquilibrer le portefeuille du bot.*
    *   Projection orthogonale (matrice chapeau) → *Isoler la tendance principale du marché en éliminant le bruit du carnet d'ordres.*

### 2. Les Fondations Statistiques (Prérequis 2)
*   **Les Opérateurs Aléatoires Conditionnels**
    *   Propriétés fondamentales de l'opérateur d'Espérance $\mathbb{E}(\cdot)$ et de Variance $\mathbb{V}(\cdot)$.
    *   L'Espérance Conditionnelle $\mathbb{E}[Y|X]$ et la Loi de l'Espérance Itérée → *Formuler la meilleure prédiction mathématique possible sachant l'information présente.*
*   **Statistiques Descriptives de Marché**
    *   Mesures de tendance centrale et de dispersion → *Calculer les moyennes et écarts-types pour fixer les Take-Profit et Stop-Loss.*
    *   Association linéaire → *Calculer la covariance et la corrélation de Pearson pour identifier des paires d'actifs liées.*
*   **Probabilités Fondamentales et Inférence**
    *   Fonctions de densité, Lois Échantillonnages (Normale, Student, Khi-deux, Fisher) → *Calculer la Value-at-Risk (VaR) et gérer les queues épaisses.*
    *   Théorie de l'Estimation → *Évaluer le biais, la convergence asymptotique et l'efficacité relative d'un indicateur sur le papier.*
    *   Inférence de Base et Tests d'hypothèses ($H_0$ vs $H_1$) → *Maîtriser la p-valeur et les erreurs de type I et II pour valider un signal d'achat.*

### 3. La Régression Linéaire Simple (Le départ)
*   **Modèle à Deux Variables**
    *   Équation de la population (PRF) vs Équation de l'échantillon (SRF) → *Distinguer la vérité théorique immuable de l'estimation glissante.*
    *   Définition du terme d'erreur ($\epsilon$) → *Modéliser le bruit de marché imprévisible que l'algorithme doit ignorer.*
*   **Mécanique et Propriétés des MCO**
    *   Minimisation de la somme des résidus carrés et dérivation des formules analytiques de la pente ($\beta_1$ / Bêta) et de la constante ($\beta_0$ / Alpha).
    *   Propriétés algébriques et ajustement du modèle → *Prouver la somme nulle des résidus et décoder le R-carré ($R^2$).*
    *   Interprétation des unités de mesure → *Traduire le modèle en ordres réels de passage de capital.*

## PARTIE 2 : LE MODÈLE LINÉAIRE MULTIPLE ET SPÉCIFICATION

### 4. La Régression Linéaire Multiple (Généralisation)
*   **Formulation Matricielle Complète**
    *   Écriture du modèle $Y = X\beta + u$ et Dérivation matricielle formelle de l'estimateur $\hat{\beta} = (X'X)^{-1}X'Y$ via les règles de dérivation matricielle.
    *   Interprétation "ceteris paribus" → *Isoler l'impact pur d'une variable macroéconomique en neutralisant toutes les autres.*
*   **Hypothèses de Gauss-Markov et Inférence**
    *   Validation rigoureuse des hypothèses (Linéarité, absence de colinéarité, exogénéité stricte $\mathbb{E}[u|X]=0$, homoscédasticité, non-autocorrélation).
    *   Preuve du non-biais de $\hat{\beta}$ et Théorème de Gauss-Markov (Propriété BLUE).
    *   Tests $t$, $F$ et ajustement du $R^2$ → *Éliminer les indicateurs inutiles et pénaliser le sur-apprentissage (overfitting) du bot.*

### 5. Spécification Moderne du Modèle Linéaire (Pratique)
*   **Variables Qualitatives et Non-Linéarités**
    *   Variables indicatrices binaires (Dummies), Piège de la variable indicatrice, et Effets d'interaction (pente variable).
    *   Modèles Log-Log (élasticités), Log-Lin, Lin-Log et Modèles polynomiaux (rendements marginaux décroissants).
*   **Sélection et Diagnostic**
    *   Critères d'information (AIC, BIC) et Test RESET de Ramsey.
    *   Traitement des valeurs aberrantes (outliers) → *Protéger le bot contre l'impact d'un Flash Crash.*

## PARTIE 3 : LES PATHOLOGIES DES DONNÉES ET L'ENDOGÉNÉITÉ

### 6. Les Pathologies des Erreurs (Économétrie Intermédiaire)
*   **Hétéroscédasticité et Autocorrélation**
    *   Conséquences de l'hétéroscédasticité, Tests de Breusch-Pagan / White, et Corrections (MCP et White robuste).
    *   Nature de l'autocorrélation temporelle, Tests de Durbin-Watson / Breusch-Godfrey, et Correction de Newey-West (HAC).
*   **Multicolinéarité Non-Parfaite**
    *   Gonflement de la variance des coefficients et calcul du Facteur d'Inflation de la Variance (VIF).

### 7. Endogénéité et Variables Instrumentales (Causalité 1)
*   **Sources et Théorie de l'Instrumentation**
    *   Biais de variable omise, erreurs de mesure et double causalité simultanée.
    *   Définition, pertinence et exogénéité stricte d'un instrument valide ($Z$).
*   **Méthodes d'Estimation**
    *   Estimateur VI, Moindres Carrés Doubles (2SLS), Statistique $F$ de première étape, et Tests de Durbin-Wu-Hausman / Sargan-Hansen.

## PARTIE 4 : MODÉLISATION DES PRIX, DU RISQUE ET DE LA MICROSTRUCTURE

### 8. Variables Dépendantes Limitées (Micro-économétrie 1)
*   **Modélisation de Choix Binaires**
    *   Modèle Linéaire de Probabilité (MLP), limites hors de l'intervalle [0,1] et hétéroscédasticité inhérente.
    *   Modèles Logit et Probit, Maximum de Vraisemblance (MV), Effets marginaux (AME), et Odds Ratios.
*   **Données Censurées et de Sélection**
    *   Modèle Tobit et Modèle de sélection en deux étapes de Heckman (Heckit).

### 9. Économétrie des Séries Temporelles Classiques (Le moteur temporel)
*   **Analyse Asymptotique et Haute Dimension**
    *   Comportement des sommes géométriques infinies et introduction aux limites de suites temporelles lorsque $T \to \infty$.
*   **Fondations et Modélisation ARMA**
    *   Stationnarité au sens faible, Opérateurs de retard, et Fonctions d'autocorrélation (FAC/FACP).
    *   Processus Autorégressifs (AR), Moyenne Mobile (MA), et Méthodologie de Box-Jenkins.
*   **Non-Stationnarité et Cointégration**
    *   Processus DS/TS, Régression spurieuse, Tests de racine unitaire (ADF, PP), Cointégration, VECM, Modèles VAR, et Causalité de Granger.

### 10. Modélisation de la Volatilité Dynamique (Risk Management)
*   **Modèles ARCH et GARCH**
    *   Volatility Clustering, modèles ARCH(p), GARCH(1,1) et application au calcul de la VaR dynamique et au Position Sizing.
*   **Extensions Asymétriques**
    *   Modèles EGARCH / TGARCH et capture de l'effet de levier sur les positions Short.

### 11. L'Économétrie des Hautes Fréquences et Microstructure (Données d'ordres)
*   **Processus Pointuels et Temporels**
    *   Processus de Hawkes (événements auto-excités dans le carnet d'ordres) et Modèles de Durée Conditionnelle Autorégressive (ACD).

## PARTIE 5 : STRUCTURES COMPLEXES ET INTELLIGENCE DU BOT

### 12. Économétrie des Données de Panel (Structures Multi-actifs)
*   Structure de Panel (individus $i$ dans le temps $t$), OLS Poolé, Modèles à Effets Fixes (FE / Transformation Within), Premières Différences (FD), Effets Aléatoires (RE / FGLS), et Test de Hausman.

### 13. Évaluation d'Impact et Politiques Publiques (Validation d'Alpha)
*   Modèle causal de Rubin (contrefactuel), Mesures ATE/ATT, Différence dans les Différences (DiD), Hypothèse des tendances parallèles, et Event Studies.

### 14. Les Changements de Régime et Non-Linéarités (Adaptabilité)
*   Modèles à Seuils (TAR / SETAR) et Modèles à Changement de Régime de Markov (Markov Switching).

### 15. L'Économétrie en Haute Dimension (Sélection d'indicateurs)
*   Régressions Régularisées (Ridge, Lasso, Elastic Net), Analyse en Composantes Principales (ACP), et Modèles de Facteurs Market Neutral.

## PARTIE 6 : SÉCURITÉ, ROBUSTESSE ET IMMUNITÉ CARDINALE

### 16. L'Économétrie Bayésienne (Mise à jour des croyances)
*   Inférence Bayésienne (Prior, Vraisemblance, Posterior), Algorithmes MCMC / Échantillonneur de Gibbs, Modèles BVAR, et Prior de Minnesota.

### 17. La Non-Linéarité des Rendements Extrêmes (Gestion des cygnes noirs)
*   Théorie des Valeurs Extrêmes (EVT / Lois GEV et GPD), Expected Shortfall (ES), Coupe-circuits (Circuit Breakers), et Modélisation par Copules.

### 18. L'Économétrie Non-Paramétrique (Flexibilité absolue)
*   Régression par Noyau (Nadaraya-Watson), choix de la fenêtre (Bandwidth), et Splines de lissage.

### 19. La Discontinuité et Seuils de Marché (Microstructure avancée)
*   Variable de forçage, Cutoff, Sharp RDD (Discontinuité nette), et Fuzzy RDD (Discontinuité floue).

### 20. L'Appariement Historique (Stratégies de reconnaissance de motifs)
*   Appariement (Matching), Score de Propension (PSM), Hypothèse d'indépendance conditionnelle (CIA), et Algorithmes d'appariement (Mahalanobis).

### 21. Les Biais Économétriques du Backtesting (Survie du bot)
*   Biais de Look-Ahead (protocoles de détection en T-1) et Biais de Survie (intégration des données d'entreprises radiées / Delisted Data).
