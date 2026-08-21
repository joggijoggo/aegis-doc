### 🔬 Laboratoire Expérimental de la Partie 2 — Expérience 5 : Quantifier l'impact de la taille du bloc (Le dilemme biais-variance)

L'objectif de cette expérience est de mesurer numériquement comment la taille du bloc fixe ($b$) altère la qualité des simulations. Nous allons démontrer qu'un bloc trop petit échoue à restituer la mémoire réelle de la volatilité (biais), tandis qu'un bloc trop grand fige la simulation et détruit la diversité statistique (variance).

---

### 1. La Construction Méthodologique de l'Expérience

Pour cette expérience, nous réutilisons la série réelle de la volatilité au carré ($R_t^2$) développée lors de l'Expérience 1 sur l'indice S&P 500 ($T = 1\ 250$ jours). Nous savons, grâce à notre point de référence initial, que l'autocorrélation de Pearson réelle d'ordre 1 de cette série est de $\rho_1 = 0.22$.

Nous allons tester trois configurations géométriques distinctes de l'algorithme de Bootstrap Circulaire par Blocs (CBB) pour mettre en évidence la sensibilité des résultats à ce paramètre :
*   **Configuration A (Blocs ultra-courts) :** $b = 2$ jours.
*   **Configuration B (Blocs intermédiaires) :** $b = 20$ jours.
*   **Configuration C (Blocs ultra-longs) :** $b = 400$ jours.

---

### 2. L'Indicateur du Biais : La Mémoire Moyenne Simulée ($\bar{\rho}_1^*$)

#### Définition et construction mathématique :
Le premier objectif de notre protocole est de mesurer la quantité de mémoire que l'algorithme est capable de reproduire. À chaque simulation $m$ (où $m$ varie de $1$ à $M$), l'ordinateur génère une série rééchantillonnée $Y^{*(m)}$ et calcule son propre coefficient d'autocorrélation d'ordre 1, noté $\rho_1^{*(m)}$, via la formule classique de Pearson.

L'indicateur final est la moyenne arithmétique de ces coefficients sur l'ensemble des simulations :

$$\bar{\rho}_1 = \frac{1}{M} \sum_{m=1}^M \rho_1^{*(m)}$$

#### Motivation économétrique du Biais :
Cet indicateur mesure la fidélité temporelle du modèle. Le biais statistique est défini par l'écart absolu entre la moyenne des simulations et la réalité historique :

$$\text{Biais} = \vert \bar{\rho}_1^* - \rho_1 \vert$$

Si l'algorithme coupe les blocs trop fréquemment, il détruit les structures de corrélation : $\bar{\rho}_1^*$ va tendre vers $0$, générant un biais élevé. Un bon paramètre $b$ doit minimiser ce biais en se rapprochant de la cible de $0.22$.

#### 🔢 Application numérique du Biais :
Imaginons que nous limitons notre étude à seulement $M = 3$ simulations pour la configuration A ($b=2$) :
*   Simulation 1 : L'ordinateur calcule un coefficient $\rho_1^{*(1)} = 0.01$
*   Simulation 2 : L'ordinateur calcule un coefficient $\rho_1^{*(2)} = -0.02$
*   Simulation 3 : L'ordinateur calcule un coefficient $\rho_1^{*(3)} = 0.04$

Calcul de l'indicateur :
$$\bar{\rho}_1^* = \frac{0.01 + (-0.02) + 0.04}{3} = \frac{0.03}{3} = \mathbf{0.01}$$

Calcul du Biais final pour cette configuration :
$$\text{Biais} = \vert 0.01 - 0.22 \vert = \mathbf{0.21}$$

#### 🎯 Justification de l'interprétation numérique :
Un résultat de $\bar{\rho}_1^* = 0.01$ (très proche de zéro) confirme que **le modèle détruit la mémoire**.

**Pourquoi ?** Parce qu'un bloc de taille $b=2$ ne contient qu'une seule transition chronologique réelle (du jour 1 au jour 2). Dès le jour suivant, l'algorithme saute vers un autre bloc tiré au hasard, ce qui crée une rupture artificielle de la mémoire. Sur une série simulée de 1 250 jours, on subit ainsi environ 625 sauts aléatoires. Ces déconnexions massives forcent la covariance à s'annuler, rapprochant mathématiquement le coefficient de Pearson de zéro. Le biais de sous-estimation du risque est ainsi maximal ($0.21$).

### 3. L'Indicateur de la Variance : La Dispersion des Moyennes de Volatilité ($\sigma_{\bar{Y}^*}$)

#### Définition et construction mathématique :
Le second objectif est de mesurer la diversité macroscopique des scénarios générés. Pour chaque série simulée $m$, l'ordinateur calcule d'abord sa moyenne de volatilité globale, notée $\bar{Y}^{*(m)} = \frac{1}{T} \sum_{t=1}^T Y_{t}^{*(m)}$.

Une fois les $M$ simulations effectuées, nous obtenons une liste de $M$ moyennes. L'indicateur de variance correspond à l'écart-type (la dispersion) de cette liste de moyennes par rapport à leur propre méta-moyenne (notée $\bar{\bar{Y}}^*$) :

$$\sigma_{\bar{Y}^*} = \sqrt{\frac{1}{M} \sum_{m=1}^M \left( \bar{Y}^{*(m)} - \bar{\bar{Y}}^* \right)^2}$$

#### Motivation économétrique de la Variance :
Le but du bootstrap est de créer des mondes parallèles crédibles mais différents. Cet indicateur $\sigma_{\bar{Y}^*}$ mesure la capacité de l'algorithme à faire varier la volatilité moyenne d'une simulation à l'autre. Si cet indicateur s'effondre vers $0$, toutes les simulations affichent la même volatilité moyenne : le bootstrap ne produit plus de diversité.

#### 🔢 Application numérique de la Variance :
Analysons ce qui se produit avec la configuration à blocs ultra-longs ($b=400$) sur $M=3$ simulations. Supposons que la vraie moyenne historique de la volatilité soit de $5.00\%$.
*   Simulation 1 : La moyenne globale de cette série s'établit à $\bar{Y}^{*(1)} = 5.01\%$
*   Simulation 2 : La moyenne globale de cette série s'établit à $\bar{Y}^{*(2)} = 4.99\%$
*   Simulation 3 : La moyenne globale de cette série s'établit à $\bar{Y}^{*(3)} = 5.00\%$

Calcul de la méta-moyenne :
$$\bar{\bar{Y}}^* = \frac{5.01 + 4.99 + 5.00}{3} = \mathbf{5.00\%}$$

Calcul de la dispersion (variance empirique) :
*   Écart 1 : $(5.01 - 5.00)^2 = 0.0001$
*   Écart 2 : $(4.99 - 5.00)^2 = 0.0001$
*   Écart 3 : $(5.00 - 5.00)^2 = 0.0000$

Moyenne des écarts carrés : $\frac{0.0001 + 0.0001 + 0}{3} = 0.0000666$
L'écart-type final $\sigma_{\bar{Y}^*}$ vaut : $\sqrt{0.0000666} \approx \mathbf{0.008\%}$.

#### 🎯 Justification de l'interprétation numérique :
Un écart-type de seulement $0.008\%$ (proche de zéro) signifie que **la variance s'est éteinte**.

**Pourquoi ?** Pour combler une simulation de 1 250 jours avec des morceaux de taille $b=400$, l'ordinateur n'a besoin de piger que 3 ou 4 blocs au total. Le nombre de combinaisons d'assemblage possibles s'effondre de manière dramatique. Les séries simulées ne sont pas de "nouveaux mondes alternatifs", mais de simples copier-coller de l'historique réel décalés de quelques jours. Chaque simulation réplique donc presque fidèlement la même distribution macroscopique et la même moyenne globale que la série historique d'origine. L'algorithme est figé et perd son pouvoir d'exploration statistique.

---

### 4. La Synthèse Algébrique : Le Compromis de l'Erreur Quadratique Moyenne (MSE)

#### Le concept d'optimalité :
En statistique, la qualité globale d'un estimateur combinant ces deux forces contraires est mesurée par l'**Erreur Quadratique Moyenne** (MSE pour *Mean Squared Error*). Pour l'estimation de notre structure de mémoire, elle se formalise ainsi :

$$\text{MSE} = \text{Biais}^2 + \text{Variance}_{\rho_1^*} = \left( \bar{\rho}_1^* - \rho_1 \right)^2 + \sigma^2_{\rho_1^*}$$

Où $\sigma^2_{\rho_1^*}$ représente la variance propre du coefficient d'autocorrélation d'une simulation à l'autre.

#### Pourquoi ces deux indicateurs sont en conflit géométrique :
Ce volet conceptuel montre que l'on ne peut pas optimiser un indicator sans dégrader l'autre. Si vous augmentez la taille du bloc pour réduire le Biais (conserver la mémoire), vous réduisez mécaniquement le nombre de blocs distincts combinables dans l'urne. Les tirages deviennent prisonniers de grands pans d'historique figés, ce qui fait chuter la diversité des scénarios. Trouver la taille de bloc optimale revient à chercher le point géométrique exact où la courbe de la MSE atteint son niveau le plus bas.

#### 🔢 Application numérique de la MSE :
Comparons mathématiquement nos configurations à l'aide de leur MSE théorique calculée sur la mémoire :

*   **Cas A (Blocs de 2 jours) :** Le modèle détruit la mémoire ($\bar{\rho}_1^* = 0.01$ au lieu de $0.22$), mais comme il mélange beaucoup les cartes, les coefficients varient peu d'une simulation à l'autre ($\sigma^2_{\rho_1^*} = 0.002$).
    $$\text{MSE}_A = (0.01 - 0.22)^2 + 0.002 = (-0.21)^2 + 0.002 = 0.0441 + 0.002 = \mathbf{0.0461}$$

*   **Cas B (Blocs de 20 jours) :** Le modèle capture bien la mémoire ($\bar{\rho}_1^* = 0.20$ au lieu de $0.22$). L'indépendance relative des 62 blocs de taille 20 laisse une variance modérée entre les simulations ($\sigma^2_{\rho_1^*} = 0.005$).
    $$\text{MSE}_B = (0.20 - 0.22)^2 + 0.005 = (-0.02)^2 + 0.005 = 0.0004 + 0.005 = \mathbf{0.0054}$$

*   **Cas C (Blocs de 400 jours) :** Le modèle ne fait aucun compromis sur la mémoire ($\bar{\rho}_1^* = 0.22$ au lieu de $0.22$). En revanche, la pauvreté des combinaisons fait exploser l'instabilité de l'estimateur de corrélation d'une réplication à l'autre ($\sigma^2_{\rho_1^*} = 0.038$).
    $$\text{MSE}_C = (0.22 - 0.22)^2 + 0.038 = (0)^2 + 0.038 = \mathbf{0.0380}$$

*Verdict mathématique :** La configuration B ($b=20$) présente la MSE the plus faible ($0.0054$). C'est le réglage statistiquement optimal parmi les trois choix de blocs fixes testés.

---

### 5. Le Protocole Algorithmique d'Exécution

Pour chacune des trois configurations ($b = 2$, $b = 20$, $b = 400$) :
1.  Initialisez deux vecteurs de stockage vides de taille $M = 10\ 000$.
2.  Lancez une boucle de 10 000 réplications. À chaque itération $m$, générez une série bootstrapée CBB de taille $1\ 250$ jours.
3.  Calculez le coefficient de Pearson $\rho_1^{*(m)}$ sur la série générée et stockez-le dans le premier vecteur.
4.  Calculez la moyenne globale $\bar{Y}^{*(m)}$ de la série générée et stockez-la dans le second vecteur.
5.  À la sortie de la boucle, calculez la moyenne du premier vecteur ($\bar{\rho}_1^*$) et l'écart-type du second vecteur ($\sigma_{\bar{Y}^*}$).

---

### 6. Règle de Décision face au Modèle Idéal

Un modèle de simulation parfait doit valider simultanément les deux critères : un biais proche de $0$ (donc $\bar{\rho}_1^* \approx 0.22$) et une variance $\sigma_{\bar{Y}^*}$ statistiquement de l'ordre de grandeur attendu (supérieure à $0.03$) pour garantir le pouvoir d'exploration du bootstrap.

---

### 7. Ce que vous allez observer

Après l'exécution des calculs sur les données réelles du S&P 500, vous allez observer un basculement radical des indicateurs :
*   **Pour $b = 2$ :** La mémoire simulée s'effondre à $\bar{\rho}_1^* \approx \mathbf{0.02}$. Le biais est massif ($0.20$). En revanche, la dispersion des moyennes est très large : $\sigma_{\bar{Y}^*} \approx \mathbf{0.08}$.
*   **Pour $b = 20$ :** L'équilibre est atteint. La mémoire est restaurée à $\bar{\rho}_1^* \approx \mathbf{0.20}$ (biais négligeable de $0.02$). La dispersion reste robuste : $\sigma_{\bar{Y}^*} \approx \mathbf{0.05}$.
*   **Pour $b = 400$ :** La mémoire est idéalement capturée à $\bar{\rho}_1^* = \mathbf{0.22}$ (biais nul). Mais la diversité s'éteint complètement : $\sigma_{\bar{Y}^*} \approx \mathbf{0.003}$.

---

### 8. Analyse des Résultats

L'analyse de ces chiffres apporte la preuve empirique du dilemme biais-variance imposé par la rigidité des blocs fixes.

Avec des blocs de 2 jours ($b=2$), les coupures incessantes détruisent le lien dynamique de la volatilité, provoquant un biais de sous-estimation du risque. À l'inverse, avec des blocs de 400 jours ($b=400$), l'ordinateur n'a besoin de piger que 3 gros morceaux pour combler les 1 250 jours de simulation. Le renouvellement des scénarios devient impossible, la variance s'effondre, et toutes les simulations ne font que reproduire la moyenne historique exacte.

Le modèle à blocs fixes condamne l'analyste à procéder par tâtonnements pour trouver un compromis précaire (comme $b=20$). C'est cette impossibilité d'ajuster dynamiquement la taille des blocs qui justifie le saut conceptuel vers le Bootstrap Stationnaire, où la loi géométrique automatisera ce lissage.
