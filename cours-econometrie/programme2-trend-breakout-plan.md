### Plan de Vol Détaillé : Programme 2 (Capture de Tendance & Rupture)

---

#### Bloc 1 : Introduction à la Récursivité & Filtre de Kalman Scalaire (1D)

##### 1.1 : Le Biais de Phase des Indicateurs Classiques & L'Alternative Récursive
*   **Objectif / Concept :** Preuve mathématique du retard temporel (*lag*) et du déphasage des moyennes mobiles (SMA, EMA) lors des retournements de tendance. Introduction à la cinétique de la mise à jour récursive : comment corriger une estimation à chaque tick sans stocker d'historique passif.
*   **Fil rouge Trading / Bot :** Codage d'un estimateur de prix récursif à mémoire dynamique. Le robot choisit de mettre à jour la valeur "vraie" d'un actif en éliminant le bruit des ticks parasites.

##### 1.2 : Le Filtre de Kalman Scalaire (1D) — La Boucle Prédiction / Correction
*   **Objectif / Concept :** Formalisation de la boucle fermée de Kalman en une dimension. Équation de transition (la dérive lente du prix) et Équation de mesure (le flux du courtier). Dérivation des équations simplifiées du Gain de Kalman ($K_t$) et de la variance de l'erreur ($P_t$).
*   **Fil rouge Trading / Bot :** Implémentation de la classe `ScalarKalmanEngine`. Le robot exécute son premier cycle complet `.predict()` et `.update()` à chaque tick. L'utilisateur observe visuellement l'extraction d'une tendance fluide dénuée de tout retard temporel lors des pivots.

##### 1.3 : Métrique d'Audit : Le Test de Blancheur de l'Innovation (Ljung-Box)
*   **Objectif / Concept :** Si le filtre de Kalman extrait tout le signal disponible, l'erreur résiduelle (l'innovation) doit être un **bruit blanc pur** sans aucune mémoire temporelle. Si le bruit résiduel est autocorrélé, le filtre est mal calibré.
*   **Fil rouge Trading / Bot :** Intégration du module `InnovationAuditor`. Le robot applique un test statistique de Ljung-Box sur ses propres résidus. Si le test rejette la blancheur, le robot lève une alerte d'infrastructure, détectant que sa grille de lecture passe à côté d'un mouvement réel.

##### 1.4 : Mini-Projet 1 — Le Traqueur de Prix 1D de Précision
*   **Cahier des charges :** Développement d'un script complet qui applique le filtre scalaire sur l'EUR/USD. Le robot doit extraire la tendance et exécuter l'audit de blancheur de l'innovation. Le test de validation compare la réactivité du filtre face à une EMA.

---

#### Bloc 2 : Modélisation Cinétique Scalaire & Calibrage Adaptatif (2D Scalaire)

##### 2.1 : Décomposition Cinétique : Position, Vitesse et Accélération du Prix
*   **Objectif / Concept :** Modéliser un prix chronologique comme un objet physique en mouvement doté d'une vitesse (le taux de variation ou *momentum* statistique) et d'une accélération. Dérivation des équations de transition couplées sous forme scalaire.
*   **Fil rouge Trading / Bot :** Développement du module `KineticTrendModeler`. Le robot apprend à projeter la position future du prix en extrapolant sa vitesse de propagation actuelle.

##### 2.2 : Robustesse Métrologique : L'Estimation Adaptative des Variances ($Q$ et $R$)
*   **Objectif / Concept :** Rompre avec le dogme des bruits fixes. Utilisation de la technique de *Covariance Matching* : le robot analyse la variance empirique de ses innovations récentes pour réajuster en temps réel la variance de processus $Q$ et la variance de mesure $R$.
*   **Fil rouge Trading / Bot :** Implémentation du noyau d'auto-calibrage `AdaptiveNoiseTuner`. Si le carnet d'ordres devient chaotique (période de news), le robot recalcule instantanément la hausse de $R$, ce qui écrase son Gain de Kalman pour immuniser sa ligne de tendance contre les faux signaux d'accélération.

##### 2.3 : Mini-Projet 2 — L'Indicateur de Vitesse Pure Auto-Ajusté
*   **Cahier des charges :** Conception d'un double filtre de Kalman scalaire interconnecté adaptatif : le premier estime la position, le second extrait la vitesse pure en adaptant ses bruits. Le robot génère un oscillateur de vitesse sans aucun retard temporel, validé par la détection des points d'inflexion exacts du marché.

#### Bloc 3 : Généralisation Matricielle & Enveloppes Élastiques

##### 3.1 : Le Formalisme de l'Espace d'États Vectoriel & Filtre de Kalman Matriciel
*   **Objectif / Concept :** Projeter le robot dans l'algèbre linéaire. Compactage des équations cinétiques et adaptatives du Bloc 2 au sein des vecteurs d'états et matrices de transition. Algorithme de propagation et de correction vectorielle standard.
*   **Fil rouge Trading / Bot :** Codage du cœur logiciel `MatrixKalmanEngine` en pur NumPy. À chaque tick, le robot met à jour simultanément son estimation du niveau de prix et de la vitesse de la tendance sous forme matricielle.

##### 3.2 : Métrique de Robustesse : La Trace de la Covariance de l'Erreur ($P_t$)
*   **Objectif / Concept :** La matrice $P_t$ quantifie la zone d'ombre probabiliste de l'estimation vectorielle. Si le marché subit un choc de volatilité imprévu, la variance $P_t$ explose, signalant une perte de contrôle métrologique.
*   **Fil rouge Trading / Bot :** Implémentation de la jauge `CovarianceSafetyGate`. Le robot calcule la trace (somme des éléments diagonaux) de la matrice $P_t$. Si ce compteur dépasse un seuil de sécurité critique, le robot coupe instantanément l'engagement de capital.

##### 3.3 : Enveloppes Élastiques de Kalman & Trailing Stop Statistique
*   **Objectif / Concept :** Utilisation de la variance d'erreur de position issue de la diagonale de $P_t$ pour tracer un canal de confiance entourant la tendance. Les bandes mesurent la précision actuelle du modèle. La frontière du canal sert de trailing stop asymétrique dynamique.
*   **Fil rouge Trading / Bot :** Implémentation du module `StatisticalTrailingStop`. Le robot réajuste sa barrière de protection à chaque tick. Si la tendance est certifiée par le filtre, le stop verrouille agressivement les gains. Si le flou augmente, le stop s'élargit pour laisser respirer la position.

##### 3.4 : Mini-Projet 3 — Le Robot de Breakout Cinétique Matriciel
*   **Cahier des charges :** Développement de la classe de production `BivariateKalmanFilter`. Le robot doit ouvrir un ordre d'achat lorsque le prix brut casse la bande supérieure matricielle de Kalman et gérer sa sortie via le trailing stop statistique. La conformité du code est validée par un test unitaire confrontant les sorties du bot aux matrices de covariance du package standard `pykalman`.

#### Bloc 4 : Classification des Régimes par les Modèles de Markov Cachés

##### 4.1 : Le Formalisme de Markov Caché (HMM) & Matrice de Transition
*   **Objectif / Concept :** Modéliser l'asymétrie structurelle des marchés. Postuler que le prix est gouverné par un état invisible qui bascule entre un Régime 1 (Compression stationnaire bruitée) et un Régime 2 (Expansion directionnelle explosive). Définition des probabilités d'émission et de transition.
*   **Fil rouge Trading / Bot :** Implémentation de la structure `MarkovRegimeSpecification`. Le robot apprend à formaliser mathématiquement le basculement cyclique des marchés financiers.

##### 4.2 : L'Algorithme de Viterbi et le Décodage Temporel
*   **Objectif / Concept :** Résolution du problème de décodage. Comprendre comment l'algorithme de Viterbi remonte le fil de l'histoire pour identifier la séquence d'états cachés la plus probable ayant généré le graphique observé.
*   **Fil rouge Trading / Bot :** Codage du décodeur de régime `ViterbiDecoder`. À chaque clôture de bougie, le robot recalcule l'état actuel du marché pour confirmer si la transition vers le régime directionnel est statistiquement validée.

##### 4.3 : Métrique de Robustesse : L'Entropie de Classification (Score de Certitude)
*   **Objectif / Concept :** Quantifier le flou du décodage markovien. Si les probabilités d'émission sont floues (ex: 51% Régime 1, 49% Régime 2), le modèle est indécis. Le robot doit calculer l'Entropie Relative pour mesurer la netteté de sa vision des cycles.
*   **Fil rouge Trading / Bot :** Implémentation du filtre `ClassificationEntropyShield`. Le robot calcule le score d'entropie en direct. Si la classification est jugée trop incertaine (entropie élevée), le robot gèle ses autorisations de trading, neutralisant le risque de sur-trading au milieu d'une zone de pivot indécise.

##### 4.4 : Mini-Projet 4 — Le Sélecteur de Régime Automatique
*   **Cahier des charges :** Assemblage du module HMM et du filtre d'entropie. Le robot doit analyser un historique de prix et colorer l'arrière-plan du graphique selon le régime détecté. Le script doit prouver sa robustesse en bloquant les ordres d'achat du Programme 2 si le breakout a lieu au milieu d'un régime classifié comme stationnaire instable.

#### Bloc 5 : État de l'Art & Backtest Animé Multi-Panneaux

##### 5.1 : Les Limites du Linéaire et Introduction au Filtre de Kalman Étendu (EKF)
*   **Objectif / Concept :** Comprendre pourquoi le filtre de Kalman standard échoue lors des krachs ou des crises de liquidité où les dynamiques deviennent brutalement non-linéaires. Introduction au Filtre de Kalman Étendu (EKF) via la linéarisation locale par la matrice Jacobienne.
*   **Fil rouge Trading / Bot :** Écriture du module `ExtendedKalmanKernel`. Le robot dérive la matrice Jacobienne en direct pour adapter ses projections à des fonctions d'évolution quadratiques ou exponentielles lors des accélérations de panique de marché.

##### 5.2 : La Régression à Commutation Markovienne (*Markov Switching Regression*)
*   **Objectif / Concept :** Intégrer le changement de comportement des coefficients. Comprendre comment modéliser un système où ce ne sont pas seulement les états qui changent, mais la pente et l'intercepte eux-mêmes qui commutent instantanément d'une valeur à une autre selon le régime décodé par le HMM.
*   **Fil rouge Trading / Bot :** Implémentation du moteur `MarkovSwitchingRegressionEngine`. Le robot calcule deux structures de coefficients parallèles et bascule de l'une à l'autre en fonction du flux macroéconomique.

##### 5.3 : Mini-Projet 5 — Le Tableau de Bord Animé à 5 Panneaux du Programme 2
*   **Cahier des charges :** C'est le grand chef-d'œuvre de validation du Programme 2. L'utilisateur développe un framework de backtest glissant complet sur 500 bougies de l'USD/JPY ou du Nasdaq, adossé à un dashboard rafraîchi en temps réel à 30 FPS via `FuncAnimation(blit=True)`.
*   **Spécification du Dashboard Synchrone à 5 Panneaux :**
    *   *Panneau Principal Supérieur (Double échelle twinx) :* Évolution chronologique brute de l'actif dépendant Y et de l'indicateur macroéconomique directeur X.
    *   *Quadrant Inférieur 1 :* Le prix brut superposé à la tendance cinétique pure extraite par le Filtre de Kalman Matriciel ou l'EKF.
    *   *Quadrant Inférieur 2 :* L'écartement élastique des bandes de protection de Kalman et la trajectoire du trailing stop statistique.
    *   *Quadrant Inférieur 3 :* L'historique du diagnostic HMM et du bouclier d'entropie (Arrière-plan coloré : Vert net = Régime Tendance validé, Rouge net = Régime Stationnaire validé, Gris = Zone d'entropie instable, trading bloqué).
    *   *Quadrant Inférieur 4 :* La courbe de capitaux comparée : Modèle de momentum classique (EMA avec retard) versus Modèle de Kalman/HMM/EKF de précision, mettant en évidence la sanctuarisation absolue du capital en zone de bruit.

---

### Intégration Finale du Programme 2

*   **The State-Space Gateway :** Le programme se clôture par la livraison d'un squelette logiciel complet à trous (`# YOUR_CODE_HERE`) englobant l'intégralité des classes du module, documenté exclusivement en anglais scientifique.
*   **Passerelle Industrielle :** Guide de conformité décrivant brièvement comment appeler ces diagnostics de pointe via les packages standards `statsmodels.tsa.regime_switching.markov_regression` et `pykalman`.

