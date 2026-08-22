### Plan de Développement Itératif : Le Moteur de Backtest Animé

---

#### Sprint 1 : Le Noyau Temporel Glissant & Optimisation OLS

*   **Spécifications Théoriques & Historiques :** L'analogie de la flèche du temps en programmation financière. Origine historique des fenêtres glissantes (*rolling windows*) pour capter l'instabilité des paramètres économiques à travers le temps.
*   **Développement Informatique (ENL) :**
    *   Création de la boucle principale du backtest (parcours du jeu de données de $t = 60$ à $500$).
    *   Mise en œuvre du sous-échantillonnage vectoriel (extraction instantanée sans boucle `for` des sous-tableaux `x_window` et `y_window`).
    *   Calcul à passe unique des coefficients optimaux $\hat{b}_t$ (Hedge Ratio) et $\hat{a}_t$ (Anchor).
*   **Contrôle de Qualité (Unit Test) :** Écriture du guardrail algébrique vérifiant les conditions normales MCO sur la fenêtre ($\sum e_i = 0$). Le pipeline crashe explicitement si l'orthogonalité est violée.

---

#### Sprint 2 : Le Pipeline d'Audit Probabiliste & Cascade de Sécurité

*   **Spécifications Théoriques & Historiques :** Justification de l'asymétrie des seuils industriels. Le risque de ruine de l'Erreur de Type I (la barrière à 5% de Student) versus le frottement mécanique du marché (le seuil de 15% pour le R² et l'ancrage).
*   **Développement Informatique (ENL) :**
    *   Implémentation de la fragmentation de la variance ($\text{TSS} = \text{ESS} + \text{RSS}$) et du calcul du $R^2$ glissant.
    *   Calcul des erreurs standards des paramètres ($\hat{\sigma}_{\hat{b}}$, $\hat{\sigma}_{\hat{a}}$) et interrogation de la fonction quantile de la loi de Student via `scipy.stats.t.ppf`.
    *   Câblage de la cascade séquentielle des trois interrupteurs automatiques (*circuit breakers*).
*   **Contrôle de Qualité (Unit Test) :** Injection d'une phase de marché chaotique synthétique pour vérifier que le jeton d'autorisation de trading bascule instantanément à `False` dès qu'un seuil est violé.

---

#### Sprint 3 : Le Z-Score Dynamique & La Logique de Backtest

*   **Spécifications Théoriques & Historiques :** La physique de l'entonnoir probabiliste. Comment l'écart au centre de gravité historique amplifie l'erreur de prévision locale sous forme de parabole. Origine historique de la validation par backtesting.
*   **Développement Informatique (ENL) :**
    *   Calcul de la variance de prévision locale en un point au temps $t$ : $\sigma_{\hat{y}_t}^2$.
    *   Formulation du **Z-Score Ajusté Dynamique** ($Z_{\text{dyn}}$) amorti par l'incertitude de la droite.
    *   Écriture de la machine à états de la logique de trading : gestion des conditions d'entrée hyperboliques ($\lvert Z_{\text{dyn}} \rvert \ge 2.00$), des conditions de sortie à l'équilibre ($\lvert Z_{\text{dyn}} \rvert \le 0.15$), et calcul du cumul des pips nets.
*   **Contrôle de Qualité (Unit Test) :** Comparaison mathématique entre le Z-Score naïf et le Z-Score ajusté lors d'une fausse extension de prix pour valider la neutralisation du signal fantôme.

---

#### Sprint 4 : Le Tableau de Bord Graphique Haute Performance & Spécification des 5 Panneaux

*   **Spécifications Théoriques & Historiques :** L'histoire de la visualisation des risques en salle de marché. Les contraintes de la programmation asynchrone et de l'optimisation mémoire du rendu graphique multi-axes.
*   **Développement Informatique (ENL) :**
    *   Initialisation d'une grille asymétrique `GridSpec` à 5 panneaux synchronisés (structure $3 \times 2$).
    *   Configuration du **Panneau Principal Supérieur** (s'étalant sur toute la largeur des deux colonnes sur la première ligne) sur double échelle (`ax.twinx()`) pour tracer l'évolution chronologique brute de l'EUR/USD (Axe gauche $Y_1$) et du GBP/USD (Axe droite $Y_2$).
    *   Configuration des 4 quadrants inférieurs (lignes 2 et 3) : Nuage de points + Droite MCO dynamique, Tunnel hyperbolique du spread, Écran des interrupteurs $R^2$/$t$-test, et Courbe de capitaux comparée.
    *   Mise en œuvre du moteur d'animation `matplotlib.animation.FuncAnimation` avec le protocole de rendu sélectif (`blit=True`) en manipulant exclusivement les objets `Line2D` via la méthode `.set_data()`.
*   **Contrôle de Qualité (Unit Test) :** Lancement global du simulateur animé. Validation visuelle de la synchronisation temporelle : l'utilisateur observe les prix bruts diverger ou converger sur le grand panneau du dessus, expliquant instantanément l'évasement des bandes hyperboliques sur le quadrant du spread et le déclenchement des ordres sur la courbe d'équité.

---

### Intégration Finale : Exercice à Trous Universel & Conclusion du Bloc 1

*   **The Multivariate Gateway :** Le projet se clôture par un squelette de code complet à trous (`# YOUR_CODE_HERE`) englobant l'intégralité des 4 Sprints, documenté exclusivement en anglais scientifique.
*   **Conclusion :** Bilan de la certification bivariée du robot et ouverture vers le **Bloc 2 (Algèbre Matricielle)**, présenté comme l'unique outil de compression syntaxique pour automatiser ce pipeline de précision sur des portefeuilles multi-actifs à haute dimension.
