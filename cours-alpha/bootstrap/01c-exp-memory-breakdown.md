### 🔬 Laboratoire Expérimental de la Partie 1 — Expérience 2 : Constater le problème

L'objectif de cette expérience est de quantifier précisément la destruction de la mémoire temporelle lorsque l'on applique l'algorithme classique d'Efron (1979) sur nos données dépendantes.

---

### 1. Le Principe du Rééchantillonnage d'Efron

Le bootstrap classique fonctionne selon le principe du tirage uniforme avec remise.

*   **La notion "avec remise" :** L'ordinateur pioche un élément, note sa valeur, puis le remet immédiatement dans l'urne. Cela signifie qu'au cours de la simulation d'une nouvelle série de taille $T$, un même jour de bourse d'origine peut être pioché plusieurs fois, tandis que d'autres jours ne seront jamais sélectionnés.
*   **La probabilité de sélection :** Chaque jour de l'historique initial possède exactement la même probabilité d'être choisi à chaque tirage, soit :

$$P = \frac{1}{T}$$

---

### 2. La Méthodologie Algorithmique et Construction Numérique

Pour construire la série bootstrapée (que l'on notera $Y^*_{\text{Efron}}$), l'ordinateur procède à une vectorisation aléatoire :

1. Il génère une liste d'indices de position temporelle aléatoires. Pour cela, il utilise une fonction pseudo-aléatoire qui génère des nombres entiers compris de manière uniforme entre $1$ et $T$ (dans notre cas, entre $1$ et $1\ 250$).
2. Supposons que pour les 5 premiers tirages, l'ordinateur sorte la suite d'entiers suivante : $[412, \ 89, \ 412, \ 1102, \ 5]$.
3. Il extrait les valeurs correspondantes dans notre vecteur de volatilité réelle $R_t^2$ mesuré à l'Expérience 1. Le vecteur simulé commencera ainsi par la valeur du jour 412, puis du jour 89, puis à nouveau du jour 412, etc., jusqu'à accumuler précisément $1\ 250$ observations.

---

### 3. Le Choix de la Métrique de Validation ($\rho_1^*$)

La métrique de validation reste rigoureusement identique à l'Expérience 1 : le **coefficient de corrélation linéaire de Pearson** appliqué avec un décalage de $k=1$ jour.

Cependant, nous appliquons cette fois-ci la formule sur les données rééchantillonnées :

$$\rho_1^* = \frac{\sum_{t=2}^T (Y^*_{\text{Efron}, t} - \bar{Y}^*)(Y^*_{\text{Efron}, t-1} - \bar{Y}^*)}{\sum_{t=1}^T (Y^*_{\text{Efron}, t} - \bar{Y}^*)^2}$$

*   **Motivation de la métrique :** En reprenant exactement la même formule statistique, nous isolons une seule variable : l'impact du mélange. Si le bootstrap d'Efron est adapté aux séries temporelles, le coefficient $\rho_1^*$ doit être statistiquement proche du coefficient réel $\rho_1$ calculé à l'Expérience 1 (qui était d'environ $0.22$).

---

### 4. Application Numérique Théorique

Pour illustrer mathématiquement pourquoi le coefficient s'effondre, regardons ce qui se passe au niveau de la covariance (le numérateur de Pearson) lors du décalage d'un jour.

Dans la série simulée, le jour $t$ (provenant par exemple du jour réel 412) se retrouve aligné avec le jour $t-1$ (provenant du jour réel 89).

Comme le tirage de l'indice 412 et de l'indice 89 a été fait de manière totalement indépendante par l'ordinateur, l'écart du point 412 par rapport à sa moyenne n'a aucune relation linéaire avec l'écart du point 89. Lors de la somme des produits des écarts, les valeurs positives et négatives vont s'annuler mutuellement. Le numérateur de la formule tend mathématiquement vers $0$.

---

### 5. Ce que vous allez observer

En calculant le coefficient de Pearson $\rho_1^*$ sur la série générée par le bootstrap d'Efron, vous allez observer que :

* Le coefficient d'autocorrélation s'effondre pour atteindre une valeur quasi-nulle, par exemple $\rho_1^* = \mathbf{0.0012}$.
* Peu importe le nombre de fois que vous relancez le bootstrap d'Efron, le résultat oscillera toujours de manière infime autour de $0$, ne dépassant jamais les seuils de bruit de fond de $\pm 0.055$.

---

### 6. Analyse des Résultats

La perte de mémoire est totale. En comparant le résultat initial ($\rho_1 \approx 0.22$) et le résultat bootstrapé ($\rho_1^* \approx 0.00$), on mesure l'erreur structurelle de la méthode d'Efron.

Puisque les tirages individuels détruisent les couples chronologiques, l'algorithme génère un monde artificiel où la volatilité de demain est totalement indépendante de celle d'aujourd'hui. Utiliser cette méthode pour simuler des trajectoires financières conduirait à sous-estimer massivement la persistance des risques et la durée des crises, prouvant de manière définitive le besoin de conserver des blocs de données consécutives.
