# CHAPITRE 2 : LE CALCUL MATRICIEL DE BASE

## 1. Morphologie d'une Matrice de Marché

En algèbre linéaire, une **matrice** est un tableau bidimensionnel de nombres réels ordonnés en lignes et en colonnes. Pour un chercheur quantitatif (*Quant*), c'est l'outil ultime pour fusionner tout un univers d'actifs au sein d'un seul bloc mathématique.

Soit $X$ une matrice de marché de dimension $(T \times K)$. Dans notre cadre de travail, cette grille se décompose selon deux axes physiques immuables :
*   **L'axe horizontal (Les lignes $t$) représente le Temps :** De la ligne $1$ à la ligne $T$ (jours, bougies, millisecondes).
*   **L'axe vertical (Les colonnes $k$) représente les Actifs :** De la colonne $1$ à la colonne $K$ (actions, cryptomonnaies, matières premières).

L'élément situé à l'intersection de la ligne $i$ et de la colonne $j$ est noté $X_{i,j}$.

$$\text{Soit } X = \begin{bmatrix} 1 & 3 \\ 2 & 1 \end{bmatrix}$$

Ici, $X$ est une matrice de dimension $(2 \times 2)$ : $2$ jours d'historique pour $2$ actifs.
*   La colonne 1 ($\begin{bmatrix} 1 \\ 2 \end{bmatrix}$) est le vecteur rendement de l'**Action A** ($+1\%$ le Jour 1, $+2\%$ le Jour 2).
*   La colonne 2 ($\begin{bmatrix} 3 \\ 1 \end{bmatrix}$) est le vecteur rendement de l'**Action B** ($+3\%$ le Jour 1, $+1\%$ le Jour 2).
*   L'élément $X_{1,2} = 3$ indique le rendement spécifique de l'Action B au Jour 1.

---

## 2. L'Addition Matricielle (Le cumul de flux)

L'addition de deux matrices consiste à fusionner deux tableaux de données case par case.
Soient deux matrices $A$ et $B$ de dimensions identiques. La matrice résultante $C = A + B$ est définie par :
$$C_{i,j} = A_{i,j} + B_{i,j}$$

### 📝 Application Numérique
Soit le flux de rendement de la banque A ($A$) et celui de la banque B ($B$) :
$$A = \begin{bmatrix} 1 & 3 \\ 2 & 1 \end{bmatrix}, \quad B = \begin{bmatrix} 0 & 2 \\ -1 & 4 \end{bmatrix}$$

$$A + B = \begin{bmatrix} 1+0 & 3+2 \\ 2+(-1) & 1+4 \end{bmatrix} = \begin{bmatrix} 1 & 5 \\ 1 & 5 \end{bmatrix}$$

### ⚠️ La Contrainte de Conformité
L'addition ou la soustraction est **strictement impossible** si les deux matrices n'ont pas exactement les mêmes dimensions. Une matrice $(2 \times 2)$ ne peut pas s'additionner avec une matrice $(3 \times 2)$, car la troisième ligne de données se retrouverait sans case homologue pour opérer le calcul, déclenchant un rejet structurel du système.

---

## 3. La Multiplication par un Scalaire (Le levier financier)

Multiplier une matrice par un scalaire (un nombre réel isolé $k$) revient à appliquer ce coefficient multiplicateur à chaque composante du tableau.
$$(kX)_{i,j} = k \times X_{i,j}$$

### 📝 Application Numérique
Si le bot applique un levier financier multiplicative de $k = 3$ sur notre matrice de marché $X$ :
$$3X = 3 \times \begin{bmatrix} 1 & 3 \\ 2 & 1 \end{bmatrix} = \begin{bmatrix} 3\times1 & 3\times3 \\ 3\times2 & 3\times1 \end{bmatrix} = \begin{bmatrix} 3 & 9 \\ 6 & 3 \end{bmatrix}$$

---

## 4. Le Produit Matriciel (L'agrégation Ligne par Colonne)

C'est l'opération maîtresse du calcul matriciel. Multiplier deux matrices ne consiste pas à multiplier les cases homologues, mais à effectuer une succession de **produits scalaires**.

### A. Règle Algébrique Fondamentale
Soit le produit $R = XW$. La case située à la ligne $i$ et à la colonne $j$ de la matrice finale $R$ est le produit scalaire entre la **ligne $i$ de la matrice de gauche ($X$)** et la **colonne $j$ de la matrice de droite ($W$)** :
$$R_{i,j} = \sum_{k=1}^K X_{i,k} W_{k,j}$$

### 📝 Application Numérique (Matrice-Vecteur)
Soit la matrice de marché $X$ et le vecteur colonne d'allocation de capital $\mathbf{w} = \begin{bmatrix} 2 \\ 4 \end{bmatrix}$ (dimension $2 \times 1$) :
$$\mathbf{r}_p = X\mathbf{w} = \begin{bmatrix} 1 & 3 \\ 2 & 1 \end{bmatrix} \times \begin{bmatrix} 2 \\ 4 \end{bmatrix}$$

*   **Ligne 1 $\cdot$ Colonne unique :** $(1 \times 2) + (3 \times 4) = 2 + 12 = \mathbf{14}$ (Rendement Portefeuille Jour 1)
*   **Ligne 2 $\cdot$ Colonne unique :** $(2 \times 2) + (1 \times 4) = 4 + 4 = \mathbf{8}$ (Rendement Portefeuille Jour 2)

$$\mathbf{r}_p = \begin{bmatrix} 14 \\ 8 \end{bmatrix}$$

### B. La Condition d'Emboîtement des Dimensions
Le produit $A \times B$ est mathématiquement valide si et seulement si **le nombre de colonnes de la matrice de gauche ($A$) est strictement égal au nombre de lignes de la matrice de droite ($B$)**.

$$\text{Dimension de } A : (T \times \mathbf{K}) \quad \times \quad \text{Dimension de } B : (\mathbf{K} \times N) \implies \text{Résultat } : (T \times N)$$

Si les dimensions internes ($\mathbf{K}$) ne concordent pas, les vecteurs manipulés lors du produit scalaire n'ont pas la même taille, bloquant l'exécution du calcul.

### C. La Non-Commutativité Absolue
Contrairement aux nombres réels, le produit matriciel dépend strictement de l'ordre des facteurs. Mettre le Marché à gauche ou à droite change la nature physique de l'opération.
$$X \times W \neq W \times X$$

*   $X\mathbf{w}$ applique le rendement du marché à tes poids de capital (cohérence temporelle).
*   $\mathbf{w}X$ est géométriquement invalide et économiquement aberrant (mélange croisé de rendements et de poids à des dates différentes).

---

## 5. La Transposition ($X^t$) et l'effet miroir

La transposition est une opération géométrique qui fait pivoter une matrice autour de sa diagonale principale. Les lignes deviennent des colonnes et les colonnes deviennent des lignes.
$$(X^t)_{i,j} = X_{j,i}$$

### 📝 Application Numérique
$$X = \begin{bmatrix} 1 & 3 \\ 2 & 1 \end{bmatrix} \implies X^t = \begin{bmatrix} 1 & 2 \\ 3 & 1 \end{bmatrix}$$

### ⚠️ Règle de Retournement des Produits
Lorsque l'on transpose le produit de deux blocs matriciels, l'ordre des facteurs doit obligatoirement être inversé pour préserver la compatibilité des dimensions :
$$(XW)^t = W^t X^t$$

---

## 6. La Trace ($\text{Tr}$) d'une Matrice

La **Trace** est un opérateur réservé exclusivement aux matrices carrées (même nombre de lignes et de colonnes). Elle est la somme de tous les éléments situés sur la diagonale principale (du haut-gauche au bas-droite) :
$$\text{Tr}(X) = \sum_{i=1}^K X_{i,i}$$

### 📝 Application Numérique
$$\text{Tr}(X) = \text{Tr}\left(\begin{bmatrix} \mathbf{1} & 3 \\ 2 & \mathbf{1} \end{bmatrix}\right) = 1 + 1 = 2$$

### 💎 Propriétés Clés de la Trace
1.  **Invariance par transposition :** La trace d'un tableau est rigoureusement identique à la trace de son miroir.
    $$\text{Tr}(X) = \text{Tr}(X^t)$$
2.  **Linéarité :** La trace d'une somme de matrices est égale à la somme de leurs traces individuelles.
    $$\text{Tr}(A + B) = \text{Tr}(A) + \text{Tr}(B)$$
