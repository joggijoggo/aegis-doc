### 🔬 Laboratoire Expérimental de la Partie 4 — Expérience 8 : Sensibilité de l'algorithme aux changements de régime (Crise vs. Calme)

L'objectif de cette expérience est de prouver que les formules de Politis-White calculent de manière autonome une probabilité de coupure $p$ adaptée au contexte de marché. Nous allons démontrer comment l'ordinateur modifie dynamiquement la taille moyenne de ses blocs sans aucune intervention humaine, uniquement en ajustant le point de rupture $k^*$ et le coefficient de force du signal $G_{\text{PW}}$.

---

### 1. Ce que l'on cherche à observer

Dans cette expérience, nous cherchons à observer la **capacité d'adaptation automatique** de l'architecture mathématique de Politis-White lorsqu'elle est confrontée à deux réalités économiques opposées.

Plus spécifiquement, nous voulons traquer l'évolution de trois variables pivots au sein de la formule de lissage :
*   Le point de rupture $k^*$ : se déplace-t-il vers le futur lorsque le marché est en crise ?
*   La force du signal $G_{\text{PW}}$ : augmente-t-elle proportionnellement à la persistance de la mémoire ?
*   L'espérance finale de la longueur de bloc $\mathbb{E}(L)$ : l'algorithme est-il capable de basculer de lui-même d'un modèle à blocs très longs (pour conserver l'information) à un modèle à blocs très courts (pour maximiser la diversité) ?

---

### 2. La Construction Méthodologique et Numérique des Échantillons

Pour isoler l'impact du comportement du marché, nous allons fabriquer deux environnements de volatilité radicalement opposés, chacun mesurant précisément $T = 1\ 000$ jours de cotation.

La matière première indispensable à l'ordinateur est un **générateur de bruit blanc normal**. À chaque pas de temps $t$, l'algorithme tire une valeur au hasard issue d'une loi normale centrée réduite (moyenne de 0, variance de 1), que l'on note $\epsilon_t \sim \mathcal{N}(0,1)$.

#### 📈 Environnement A : Le Marché en Crise (Processus AR(1) à Forte Persistance)
Ce vecteur simule un indice boursier en pleine tempête financière. Les chocs de volatilité y sont interconnectés : une forte secousse aujourd'hui engendre de violentes répliques sur plusieurs semaines.
*   **Formule de construction (Processus Autorégressif) :**
    $$Y_t = \mu + \phi \cdot Y_{t-1} + \sigma \cdot \epsilon_t$$
*   **Paramètres choisis :** Moyenne de fond $\mu = 0.05$, coefficient d'autocorrélation $\phi = 0.80$ (créant une inertie de 80% du jour précédent), et écart-type du choc $\sigma = 0.40$. Initialisation à $Y_0 = 0.05$.
*   **Génération chiffrée pas à pas (Chocs tirés : \(\epsilon_1 = +1.5\), \(\epsilon_2 = -0.1\), \(\epsilon_3 = -1.2\)) :**
    *   **Jour 1 :** $Y_1 = 0.05 + 0.80 \times 0.05 + 0.40 \times 1.5 = 0.05 + 0.04 + 0.60 = \mathbf{0.69}$
    *   **Jour 2 :** $Y_2 = 0.05 + 0.80 \times 0.69 + 0.40 \times (-0.1) = 0.05 + 0.552 - 0.04 = \mathbf{0.562}$ (Le marché reste haut par persistance).
    *   **Jour 3 :** $Y_3 = 0.05 + 0.80 \times 0.562 + 0.40 \times (-1.2) = 0.05 + 0.4496 - 0.48 = \mathbf{0.0196}$
*   **Variance calculée sur l'échantillon final ($\sigma_A^2$) :** $\sigma_A^2 = \mathbf{3.20}$

#### 📉 Environnement B : Le Marché Calme et Efficient (Processus i.i.d.)
Ce vecteur simule un marché idéal, totalement apaisé et liquide, où les prix intègrent instantanément l'information. Les variations d'un jour n'ont aucune influence sur le lendemain.
*   **Formule de construction :**
    $$Y_t = \mu + \sigma \cdot \epsilon_t$$
*   **Paramètres choisis :** Moyenne de fond $\mu = 0.05$, coefficient de mémoire $\phi = 0.00$ (absence de dépendance), et écart-type du choc $\sigma = 0.45$.
*   **Génération chiffrée pas à pas (Mêmes chocs tirés : \(\epsilon_1 = +1.5\), \(\epsilon_2 = -0.1\), \(\epsilon_3 = -1.2\)) :**
    *   **Jour 1 :** $Y_1 = 0.05 + 0.45 \times 1.5 = 0.05 + 0.675 = \mathbf{0.725}$
    *   **Jour 2 :** $Y_2 = 0.05 + 0.45 \times (-0.1) = 0.05 - 0.045 = \mathbf{0.005}$ (Le marché oublie instantanément le choc de la veille).
    *   **Jour 3 :** $Y_3 = 0.05 + 0.45 \times (-1.2) = 0.05 - 0.54 = \mathbf{-0.490}$
*   **Variance calculée sur l'échantillon final ($\sigma_B^2$) :** $\sigma_B^2 = \mathbf{0.20}$

---

### 3. Le Cœur de la Mesure : Le Seuil Critique Universel

La taille de l'échantillon étant identique dans les deux mondes ($T=1\ 000$), l'ordinateur commence par calculer l'unique ligne de tolérance au bruit qui servira d'arbitre pour filtrer les coefficients d'autocorrélation de Pearson ($\rho_k$) :

$$\text{Seuil}_{\text{PW}} = 2 \times \sqrt{\frac{\ln(1000)}{1000}} = 2 \times \sqrt{\frac{6.9077}{1000}} \approx \mathbf{0.1662}$$

---

### 4. Application Numérique Pas à Pas sur le Marché en Crise (Environnement A)

L'ordinateur déploie l'algorithme de Politis-White sur les données de la crise financière :

#### Étape A : Scan et détection du point de rupture $k^*$
L'ordinateur calcule les coefficients d'autocorrélation pour les décalages successifs :
*   $k=1 \implies \rho_1 = 0.65$ (Supérieur à $0.1662 \rightarrow$ Signal actif)
*   $k=2 \implies \rho_2 = 0.52$ (Supérieur à $0.1662 \rightarrow$ Signal actif)
*   $k=3 \implies \rho_3 = 0.44$ (Supérieur à $0.1662 \rightarrow$ Signal actif)
*   $k=4 \implies \rho_4 = 0.35$ (Supérieur à $0.1662 \rightarrow$ Signal actif)
*   $k=5 \implies \rho_5 = 0.28$ (Supérieur à $0.1662 \rightarrow$ Signal actif)
*   $k=6 \implies \rho_6 = 0.20$ (Supérieur à $0.1662 \rightarrow$ Signal actif)
*   $k=7 \implies \rho_7 = 0.12$ (Inférieur à $0.1662 \rightarrow$ **Coupure actée !**)

L'algorithme s'arrête instantanément au premier franchissement descendant :
$$\mathbf{k^* = 7}$$

#### Étape B : Calcul de la fenêtre globale $M$
Pour capturer les effets de traîne de la crise, l'ordinateur applique le multiplicateur de sécurité :
$$M = 2 \times 7 = \mathbf{14 \text{ jours}}$$

#### Étape C : Calcul de la force de la mémoire cumulée $G_{\text{PW}}$
L'ordinateur effectue la somme pondérée par le temps des 14 coefficients (en considérant que les jours 8 à 14 oscillent près de zéro et s'annulent) :
$$\sum_{k=1}^{14} k \cdot \rho_k = (1 \times 0.65) + (2 \times 0.52) + (3 \times 0.44) + (4 \times 0.35) + (5 \times 0.28) + (6 \times 0.20) + (7 \times 0.12)$$
$$\sum_{k=1}^{14} k \cdot \rho_k = 0.65 + 1.04 + 1.32 + 1.40 + 1.40 + 1.20 + 0.84 = 7.89$$

Le coefficient de structure $G_{\text{PW}}$ pour le marché en crise s'établit à :
$$G_{\text{PW}} = 2 \times 7.89 = \mathbf{15.78} \implies G_{\text{PW}}^2 = \mathbf{249.008}$$

#### Étape D : Extraction cubique de $p_{\text{optimal}}$
L'ordinateur injecte la variance stressée ($\sigma_A^2 = 3.20$) and le facteur d'échelle ($1000^{-1/3} = 0.10$) :
$$p_{\text{optimal}} = \left( \frac{3.20}{249.008} \right)^{1/3} \times 0.10 = (0.01285)^{1/3} \times 0.10 \approx 0.2342 \times 0.10 = \mathbf{0.0234}$$

---

### 5. Application Numérique Pas à Pas sur le Marché Calme (Environnement B)

L'ordinateur déploie les mêmes formules sur les données du marché efficient :

#### Étape A : Scan et détection du point de rupture $k^*$
L'ordinateur scanne les coefficients de Pearson :
*   $k=1 \implies \rho_1 = 0.04$ (Inférieur à $0.1662 \rightarrow$ **Coupure immédiate !**)

N'ayant détecté aucun signal au-dessus du bruit de fond dès le premier jour, l'algorithme s'arrête :
$$\mathbf{k^* = 1}$$

#### Étape B : Calcul de la fenêtre globale $M$
La fenêtre se rétracte à son minimum réglementaire :
$$M = 2 \times 1 = \mathbf{2 \text{ jours}}$$

#### Étape C : Calcul de la force de la mémoire cumulée $G_{\text{PW}}$
L'ordinateur calcule le signal sur cette fenêtre microscopique :
$$G_{\text{PW}} = 2 \times (1 \times \rho_1) = 2 \times 0.04 = \mathbf{0.08} \implies G_{\text{PW}}^2 = \mathbf{0.0064}$$

#### Étape D : Extraction cubique de $p_{\text{optimal}}$
L'ordinateur injecte la faible variance calme ($\sigma_B^2 = 0.20$) and le facteur d'échelle ($0.10$) :
$$p_{\text{optimal}} = \left( \frac{0.20}{0.0064} \right)^{1/3} \times 0.10 = (31.25)^{1/3} \times 0.10 \approx 3.1498 \times 0.10 = \mathbf{0.3150}$$

---

### 6. Ce que l'on interprète

L'analyse comparative de ces deux exécutions numériques nous permet d'interpréter le comportement et la pertinence de la méthode :

*   **Interprétation du Marché en Crise (Environnement A) :** La probabilité de coupure s'établit à **0.0234** (soit seulement 2.34 % de chances de briser le bloc à chaque étape). Par conséquent, la longueur moyenne des blocs s'allonge d'elle-même à $\mathbb{E}(L) = \frac{1}{0.0234} \approx \mathbf{42.7 \text{ jours}}$. L'algorithme a intercepté l'inertie de la crise financière : il ordonne de déployer de très grands blocs géométriques (environ 43 jours) car c'est le seul moyen de conserver la structure de dépendance et de ne pas sous-estimer le risque systémique persistant.
*   **Interprétation du Marché Calme (Environnement B) :** La probabilité de coupure grimpe à **0.3150** (31.50 % de chances d'arrêt). La longueur moyenne des blocs s'effondre à $\mathbb{E}(L) = \frac{1}{0.3150} \approx \mathbf{3.17 \text{ jours}}$. N'ayant trouvé aucune structure de mémoire dans le bruit blanc ($G_{\text{PW}} \to 0$), l'algorithme choisit de réduire ses blocs au minimum. Conserver de grands blocs fixes n'aurait servi qu'à copier-coller inutilement l'historique d'origine, détruisant la diversité des simulations. En resserrant les morceaux autour de 3 jours, l'ordinateur libère le pouvoir d'exploration du bootstrap et maximise la diversité (la variance) des scénarios générés.

**Conclusion du Laboratoire :** L'algorithme de Politis & White résout élégamment le dilemme biais-variance de manière dynamique. Il s'autorégule de façon optimale selon la structure interne de la série temporelle analysée.
