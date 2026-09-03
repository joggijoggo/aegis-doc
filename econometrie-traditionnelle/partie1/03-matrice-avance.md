# BLOC-NOTE DU QUANT : LE DÉTERMINANT D'UNE MATRICE CARRÉE

## 1. Genèse algébrique et définition
Le déterminant d'une matrice carrée $A \in \mathbb{R}^{n \times n}$ est l'unique scalaire, noté $\det(A)$ ou $|A|$, qui détermine de manière univoque l'existence et l'unicité de la solution d'un système d'équations linéaires.

Historiquement, il émerge du besoin d'isoler une variable par élimination successive des autres inconnues.

*   **En dimension 1 :** L'équation scalaire $ax = b$ admet une solution unique $x = \frac{b}{a}$ si et seulement si $a \neq 0$.
*   **En dimension 2 :** Le système de deux équations à deux inconnues :
    $$\begin{cases} a x_1 + b x_2 = e \\ c x_1 + d x_2 = f \end{cases}$$
    se résout par élimination de $x_2$, ce qui isole le terme $(ad - bc)x_1 = ed - fb$. L'existence de la solution dépend du terme $(ad - bc)$. Si on range ces coefficients dans une matrice $A = \begin{pmatrix} a & b \\ c & d \end{pmatrix}$, ce terme clé est son déterminant :
    $$\det(A) = ad - bc$$

## 2. Intuition géométrique et Effondrement de l'espace
Géométriquement, le déterminant est le **facteur de modification des hyper-volumes** induit par l'opérateur linéaire associé à la matrice.

*   **Dimension 2 :** $\det(A)$ correspond à l'aire du parallélogramme formé par les vecteurs colonnes de $A$.
*   **Dimension 3 :** $\det(A)$ correspond au volume du parallélépipède formé par les trois vecteurs colonnes de $A$.
*   **Dimension n :** $\det(A)$ correspond à l'hyper-volume de l'hyperparallélépipède formé par les $n$ vecteurs colonnes dans $\mathbb{R}^n$.

### La Singularité
Lorsque $\det(A) = 0$, la structure géométrique subit un **effondrement dimensionnel**.
L'hyper-volume devient nul car au moins une des dimensions d'information est une réplication linéaire des autres. Les vecteurs colonnes ne s'écartent pas dans toutes les dimensions de l'espace ; l'information s'aplatit sur un sous-espace de dimension strictement inférieure à $n$. La matrice est dite **singulière**.

## 3. Formule générale et Récursivité (Développement de Laplace)
Pour une matrice de dimension supérieure, le calcul se généralise de manière récursive en réduisant la taille du problème d'un ordre à chaque étape.

Soit une matrice $A = (a_{ij}) \in \mathbb{R}^{n \times n}$. Le développement suivant la ligne $i$ est donné par :
$$\det(A) = \sum_{j=1}^{n} (-1)^{i+j} a_{ij} \det(M_{ij})$$

*   $M_{ij}$ est la sous-matrice (ou mineur) de dimension $(n-1) \times (n-1)$ obtenue en supprimant la ligne $i$ et la colonne $j$.
*   $(-1)^{i+j}$ est le facteur de signe alterné dicté par la position du coefficient, découlant directement des inversions de signes lors de la mise au dénominateur commun des variables éliminées.

Pour une matrice 3 × 3 développée sur sa première ligne ($i=1$) :
$$\det \begin{pmatrix} a & b & c \\ d & e & f \\ g & h & i \end{pmatrix} = a(ei - hf) - b(di - gf) + c(dh - eg)$$

## 4. Propriétés algébriques fondamentales pour l'économétrie

Soient $A$ et $B$ deux matrices carrées de dimension $n \times n$, et $\lambda$ un scalaire réel :

1.  **Invariance par transposition :**
    $$\det(A^t) = \det(A)$$
2.  **Linéarité face à l'homothétie :**
    $$\det(\lambda A) = \lambda^n \det(A)$$
3.  **Théorème de multiplicativité :**
    $$\det(AB) = \det(A) \times \det(B)$$
4.  **Déterminant de l'inverse :**
    $$\det(A^{-1}) = \frac{1}{\det(A)} \quad (\text{si } \det(A) \neq 0)$$

# BLOC-NOTE DU QUANT : L'INVERSION MATRICIELLE ET LE RANG PLEIN

## 1. Définitions fondatrices
L'inversion matricielle est l'analogue de la division scalaire. Elle permet d'isoler un vecteur d'inconnues dans un système d'équations du type $X\beta = Y$. La division directe par un tableau n'existant pas, on utilise la matrice inverse pour "nettoyer" le système.

*   **Matrice Identité ($I_n$) :** Élément neutre de la multiplication matricielle. C'est une matrice carrée de dimension $n \times n$ contenant des $1$ sur sa diagonale principale et des $0$ partout ailleurs.
    $$I_2 = \begin{pmatrix} 1 & 0 \\ 0 & 1 \end{pmatrix}$$
*   **Matrice Inverse ($X^{-1}$) :** Soit une matrice carrée $X \in \mathbb{R}^{n \times n}$. S'il existe une matrice $X^{-1}$ telle que :
    $$X X^{-1} = X^{-1} X = I_n$$
    alors $X$ est dite inversible ou régulière.

## 2. La condition de Rang Plein (Full Rank)
Une matrice carrée $X$ de dimension $n \times n$ n'admet une inverse que si et seulement si elle est de **rang plein**, noté $\text{rg}(X) = n$.

*   **Le Rang :** Nombre maximal de lignes ou de colonnes linéairement indépendantes.
*   **Lien avec le déterminant :** Une matrice est de rang plein si et seulement si son déterminant est strictement non nul ($\det(X) \neq 0$). Si $\det(X) = 0$, la formule $\det(X^{-1}) = \frac{1}{\det(X)}$ implique une division par zéro impossible : l'inverse n'existe pas.

## 3. Le cas des données réelles (Matrices rectangulaires)
En économétrie, la matrice des données $X$ est rectangulaire (dimension $n \times k$ avec $n$ observations et $k$ variables, $n > k$). Elle ne possède pas de déterminant ni d'inverse directe.

Pour résoudre le modèle $Y = X\beta + \epsilon$, on prémultiplie le système par la transposée $X^t$ (dimension $k \times n$) :
$$X^t X \beta = X^t Y$$
Le produit $X^t X$ est une matrice **carrée de dimension $k \times k$** (matrice d'information). Elle est inversible sous condition de rang plein colonne de la matrice initiale $X$ (absence de colinéarité parfaite entre les variables de marché).

## 4. Zoom Algorithmique : La Comatrice $\text{Com}(X)$
La comatrice (ou matrice des cofacteurs) est le cœur algébrique de l'inversion générale. Elle convertit chaque élément d'une matrice en sa "valeur de survie systémique" par rapport au reste du tableau.

Pour une matrice carrée $X$ de taille $M \times M$, la comatrice $\text{Com}(X)$ est une nouvelle matrice de même dimension $M \times M$. Chaque coefficient de coordonnée $(i, j)$ de la comatrice est un scalaire appelé **cofacteur**, défini par la formule rigoureuse :
$$\text{Co}(x_{ij}) = (-1)^{i+j} \cdot \det(M_{ij})$$

Le calcul se décompose en deux briques strictes :
1.  **Le Mineur ($\det(M_{ij})$) :** C'est le déterminant de la sous-matrice de dimension $(M-1) \times (M-1)$ obtenue en rayant virtuellement la ligne $i$ et la colonne $j$ de la matrice d'origine.
2.  **Le Damier des Signes ($(-1)^{i+j}$) :** C'est le facteur correcteur d'orientation. Il applique un signe $+$ si la somme de la ligne et de la colonne est paire, et un signe $-$ si elle est impaire. Visuellement, cela forme un damier immuable :
    $$\begin{pmatrix} + & - & + & \dots \\ - & + & - & \dots \\ + & - & + & \dots \\ \vdots & \vdots & \vdots & \ddots \end{pmatrix}$$

## 5. Propriétés algébriques détaillées de l'Inverse
Soient $A$ et $B$ deux matrices carrées inversibles de même taille, et $\lambda$ un scalaire réel non nul :

*   **L'inverse d'un produit (Ordre inversé) :**
    $$(AB)^{-1} = B^{-1} A^{-1}$$
    *Démonstration :* $(AB)(B^{-1}A^{-1}) = A(BB^{-1})A^{-1} = A I A^{-1} = A A^{-1} = I$. L'inversion d'un produit de matrices exige d'inverser l'ordre des facteurs, par analogie avec le fait de devoir retirer ses chaussures avant ses chaussettes lors de l'opération inverse de l'habillage.
*   **Permutabilité avec la transposition :**
    $$(A^t)^{-1} = (A^{-1})^t$$
    Prendre l'inverse d'une matrice qui a été transposée produit le même résultat algébrique que de transposer une matrice préalablement inversée. Les deux opérateurs commutent.
*   **Conservation de la symétrie :**
    $$\text{Si } A^t = A, \text{ alors } (A^{-1})^t = A^{-1}$$
    Si la matrice d'origine est un miroir parfait par rapport à sa diagonale principale, sa structure inverse conserve cette propriété géométrique. C'est le cas systématique de la matrice d'information $X^t X$.
*   **Homothétie face à un scalaire :**
    $$(\lambda A)^{-1} = \frac{1}{\lambda} A^{-1}$$
    Multiplier l'ensemble des coefficients d'une matrice par un facteur $\lambda$ se traduit par la multiplication de tous les éléments de sa matrice inverse par le scalaire inverse $\frac{1}{\lambda}$.

## 6. Méthodes de calcul de l'inverse
### Cas d'une matrice 2 × 2
$$X = \begin{pmatrix} a & b \\ c & d \end{pmatrix} \implies X^{-1} = \frac{1}{ad - bc} \begin{pmatrix} d & -b \\ -c & a \end{pmatrix}$$

### Cas général M × M (Théorème d'Inversion Globale)
$$X^{-1} = \frac{1}{\det(X)} \cdot \text{Com}(X)^t$$

**Algorithme de résolution sur le papier :**
1.  Calculer le déterminant global $\det(X)$. S'il est nul, stopper le calcul (matrice singulière).
2.  Construire la comatrice $\text{Com}(X)$ en calculant les $M^2$ cofacteurs individuels (Mineur $\times$ son signe de damier).
3.  Transposer la comatrice ($\text{Com}(X)^t$) en redressant chaque ligne sous forme de colonne.
4.  Diviser chaque coefficient du tableau résultant par le scalaire $\det(X)$.

# BLOC-NOTE DU QUANT : L'ESSENTIEL DE LA DÉRIVATION VECTORIELLE ET MATRICIELLE

## 1. Morphologie des Formes Linéaires et Quadratiques

### A. La Forme Linéaire : Le score pondéré
En algèbre matricielle, une **forme** désigne une fonction dont l'intrant est un vecteur et l'extrant un scalaire (un nombre isolé). La forme linéaire est l'équivalent multidimensionnel de la droite d'équation $f(x) = ax$. Elle modélise une agrégation pure, sans puissances ni interactions.

*   **Écriture analytique :** Pour un système à $M$ variables, elle réalise la somme pondérée suivante :
    $$f(x_1, x_2, \dots, x_M) = a_1 x_1 + a_2 x_2 + \dots + a_M x_M$$
*   **Écriture matricielle :** Elle se condense par le produit scalaire d'un vecteur colonne de constantes $a \in \mathbb{R}^M$ et d'un vecteur colonne de variables $x \in \mathbb{R}^M$ :
    $$f(x) = a^t x = \begin{pmatrix} a_1 & a_2 & \dots & a_M \end{pmatrix} \begin{pmatrix} x_1 \\ x_2 \\ \vdots \\ x_M \end{pmatrix}$$
*   **Propriété de symétrie scalaire :** Le résultat étant un nombre isolé, le produit est égal à sa propre transposée, ce qui permet d'inverser l'ordre des facteurs sans modifier la fonction : $a^t x = (a^t x)^t = x^t a$.

### B. La Forme Quadratique : La mesure du risque et des courbes
La forme quadratique est l'extension multidimensionnelle de la parabole $g(x) = ax^2$. Elle capture les variations au second degré ainsi que les interdépendances entre les variables de marché.

*   **Écriture analytique :** Elle intègre les carrés de chaque variable et les produits croisés de toutes les paires possibles :
    $$g(x_1, x_2) = A_{11}x_1^2 + (A_{12} + A_{21})x_1x_2 + A_{22}x_2^2$$
*   **Écriture matricielle :** Elle s'exprime sous la forme d'un produit en sandwich structurel autour d'une matrice carrée centrale $A \in \mathbb{R}^{M \times M}$ :
    $$g(x) = x^t A x$$
*   **Rôle de dispatching de la matrice centrale :**
    *   La **diagonale principale** ($A_{ii}$) stocke les coefficients affectés aux carrés purs ($x_i^2$).
    *   Les **éléments hors-diagonale** ($A_{ij}$ et $A_{ji}$) stockent les forces des interactions croisées ($x_i x_j$).
*   **Le principe de symétrisation obligatoire :** Puisque $x_i x_j = x_j x_i$, l'interaction globale est la somme des deux cases symétriques. Si une matrice $A$ n'est pas symétrique, on peut la remplacer par sa composante symétrique pure $A_{\text{sym}} = \frac{A + A^t}{2}$ sans modifier la valeur de la fonction, car $x^t A x = x^t \left(\frac{A + A^t}{2}\right) x$. En finance quantitative, la matrice centrale est presque systématiquement symétrique (comme la matrice de variance-covariance).

#### Anatomie et Propriétés de la Matrice Centrale $A$

Dans l'étude des formes quadratiques $x^t A x$, la matrice centrale $A \in \mathbb{R}^{M \times M}$ ne sert pas uniquement de réceptacle pour les coefficients. Sa structure algébrique dicte la géométrie de la fonction et définit les interactions systémiques des variables.

##### A. La Décomposition Symétrique / Asymétrique
Toute matrice carrée $A$ peut être décomposée de manière unique en la somme d'une matrice symétrique $A_{\text{s}}$ et d'une matrice antisymétrique $A_{\text{as}}$ (où $A_{\text{as}}^t = -A_{\text{as}}$) :
$$A = A_{\text{s}} + A_{\text{as}} \quad \text{avec} \quad A_{\text{s}} = \frac{A + A^t}{2} \quad \text{et} \quad A_{\text{as}} = \frac{A - A^t}{2}$$

*   **L'annulation antisymétrique :** Si l'on injecte la partie antisymétrique dans une forme quadratique, le résultat est systématiquement nul pour tout vecteur $x$ :
    $$x^t A_{\text{as}} x = 0$$
    *Démonstration :* Le produit $x^t A_{\text{as}} x$ étant un scalaire, il est égal à sa propre transposée : $x^t A_{\text{as}} x = (x^t A_{\text{as}} x)^t = x^t A_{\text{as}}^t x$. Comme $A_{\text{as}}^t = -A_{\text{as}}$, on obtient $x^t A_{\text{as}} x = -x^t A_{\text{as}} x$, ce qui impose rigoureusement $2 (x^t A_{\text{as}} x) = 0 \implies x^t A_{\text{as}} x = 0$.
*   **Conclusion fondamentale :** Seul le comportement de la partie symétrique $A_{\text{s}}$ influence la valeur d'une forme quadratique. C'est pourquoi, sans perte de généralité, les quants considèrent toujours que la matrice centrale $A$ est symétrique ($A = A^t$).

##### B. Représentation spectrale et Signatures de marché
La morphologie de la matrice $A$ détermine la courbure de l'espace de la fonction. En tant que matrice réelle et symétrique, le théorème spectral garantit que $A$ est diagonalisable dans une base orthogonale. Ses valeurs propres ($\lambda_i$) décrivent les pentes de la fonction le long de ses axes principaux :

*   **Matrice Définie Positive ($A \succ 0$) :** Toutes les valeurs propres de $A$ sont strictement supérieures à zéro. Pour tout vecteur $x \neq 0_M$, la forme quadratique est strictement positive ($x^t A x > 0$). La fonction dessine une cuvette parfaite (parabolique concave vers le haut).
*   **Matrice Définie Négative ($A \prec 0$) :** Toutes les valeurs propres de $A$ sont strictement inférieures à zéro. Pour tout vecteur $x \neq 0_M$, $x^t A x < 0$. La fonction dessine un dôme (parabolique convexe vers le bas).
*   **Matrice Indéfinie :** La matrice possède à la fois des valeurs propres positives et négatives. La fonction dessine une selle de cheval (col). Le signe de la forme quadratique dépend de la direction du vecteur $x$.

##### C. La Matrice de Variance-Covariance : Le prototype financier
En économétrie et en gestion de portefeuille (modèle de Markowitz), la matrice centrale $A$ prend presque toujours la forme de la matrice de variance-covariance, notée $\Sigma$.
*   Les éléments diagonaux $\Sigma_{ii}$ correspondent aux variances des actifs (leur risque intrinsèque). Ils sont mathématiquement contraints d'être positifs.
*   Les éléments hors-diagonale $\Sigma_{ij}$ correspondent aux covariances croisées.
*   Puisque la covariance entre l'actif $i$ et l'actif $j$ est identique à la covariance entre $j$ et $i$, la relation $\Sigma_{ij} = \Sigma_{ji}$ est naturellement vérifiée : la matrice de marché $\Sigma$ est structurellement symétrique.


---

## 2. Le Concept Analytique du Gradient

Dériver une fonction par rapport à tout un vecteur $x$ consiste à observer comment la fonction réagit face à une micro-variation de chacune de ses composantes, indépendamment les unes des autres.

Le **gradient** (noté $\nabla_x$ ou $\frac{\partial}{\partial x}$) regroupe l'ensemble des dérivées partielles premières au sein d'un unique vecteur colonne de dimension $M \times 1$.

$$\frac{\partial f}{\partial x} = \begin{pmatrix} \frac{\partial f}{\partial x_1} \\ \frac{\partial f}{\partial x_2} \\ \vdots \\ \frac{\partial f}{\partial x_M} \end{pmatrix}$$

Le gradient pointe mathématiquement dans la direction de la plus forte pente ascendante de la fonction au point donné. L'annulation du gradient ($\frac{\partial f}{\partial x} = 0_M$) constitue la condition nécessaire du premier ordre pour localiser un extremum (un minimum ou un maximum de marché).

---

## 3. Les Règles Fondamentales de Dérivation Matricielle

Par analogie directe avec les règles de dérivation scalaires apprises au lycée, l'analyse matricielle synthétise le calcul différentiel en trois formules globales :

### 📜 Règle 1 : Dérivation d'une Forme Linéaire
La dérivée d'une ligne droite est sa pente. Le gradient d'une forme linéaire par rapport au vecteur de variables extrait purement le vecteur de ses constantes.
$$\frac{\partial (a^t x)}{\partial x} = a$$
$$\frac{\partial (x^t a)}{\partial x} = a$$

### 📜 Règle 2 : Dérivation d'une Forme Quadratique Symétrique
Lorsque la matrice centrale $A$ est parfaitement symétrique ($A^t = A$), la dérivation fait descendre la puissance 2 au début du bloc matriciel, préservant la structure linéaire globale. C'est le calque exact de la formule scalaire $\frac{d(ax^2)}{dx} = 2ax$.
$$\frac{\partial (x^t A x)}{\partial x} = 2Ax$$

### 📜 Règle 3 : Dérivation d'une Forme Quadratique Générale (Non-Symétrique)
Si la matrice $A$ présente des coefficients asymétriques hors de sa diagonale, la dérivation doit tenir compte de la contribution distincte de la ligne et de la colonne de chaque élément. Le gradient se généralise sous la forme suivante :
$$\frac{\partial (x^t A x)}{\partial x} = (A + A^t)x$$
*(Note : Si $A$ est symétrique, $A + A^t = A + A = 2A$, ce qui prouve la cohérence parfaite de la règle 2).*

---

## 4. Démonstration Algébrique Formelle (Cas de dimension 2)

Pour prouver rigoureusement la règle de la forme quadratique symétrique sans parachuter le résultat, développons entièrement l'opérateur en dimension 2.

Soit la matrice symétrique $A = \begin{pmatrix} a & b \\ b & c \end{pmatrix}$ et le vecteur $x = \begin{pmatrix} x_1 \\ x_2 \end{pmatrix}$.

### Étape A : Expansion de la forme quadratique
Calculons le produit en sandwich $g(x) = x^t A x$ :
$$g(x) = \begin{pmatrix} x_1 & x_2 \end{pmatrix} \begin{pmatrix} a & b \\ b & c \end{pmatrix} \begin{pmatrix} x_1 \\ x_2 \end{pmatrix}$$
$$g(x) = \begin{pmatrix} x_1 & x_2 \end{pmatrix} \begin{pmatrix} a x_1 + b x_2 \\ b x_1 + c x_2 \end{pmatrix}$$
$$g(x) = x_1(a x_1 + b x_2) + x_2(b x_1 + c x_2) = a x_1^2 + 2b x_1 x_2 + c x_2^2$$

### Étape B : Calcul des dérivées partielles du Gradient
1.  **Dérivée par rapport à $x_1$ (en traitant $x_2$ comme une constante) :**
    $$\frac{\partial g}{\partial x_1} = 2a x_1 + 2b x_2 = 2(a x_1 + b x_2)$$
2.  **Dérivée par rapport à $x_2$ (en traitant $x_1$ comme une constante) :**
    $$\frac{\partial g}{\partial x_2} = 2b x_1 + 2c x_2 = 2(b x_1 + c x_2)$$

### Étape C : Reconstruction vectorielle
Rongeons ces deux composantes au sein du vecteur du gradient :
$$\frac{\partial g}{\partial x} = \begin{pmatrix} \frac{\partial g}{\partial x_1} \\ \frac{\partial g}{\partial x_2} \end{pmatrix} = \begin{pmatrix} 2(a x_1 + b x_2) \\ 2(b x_1 + c x_2) \end{pmatrix} = 2 \begin{pmatrix} a & b \\ b & c \end{pmatrix} \begin{pmatrix} x_1 \\ x_2 \end{pmatrix} = 2Ax$$
**La propriété $\frac{\partial (x^t A x)}{\partial x} = 2Ax$ est formellement démontrée.**

## 6. Propriétés Opératoires du Gradient Matriciel

Soient $a$ et $b$ deux vecteurs colonnes de constantes, $A$ et $B$ deux matrices carrées de dimensions compatibles, $X$ une matrice de données de constantes, $\lambda$ un scalaire réel, et $x$ le vecteur colonne des variables.

### A. Linéarité de l'opérateur Gradient
Le gradient est un opérateur différentiel linéaire. Il respecte la distributivité sur l'addition et l'homothétie par rapport à un scalaire :
*   **Distributivité :**
    $$\frac{\partial [f(x) + g(x)]}{\partial x} = \frac{\partial f(x)}{\partial x} + \frac{\partial g(x)}{\partial x}$$
*   **Homothétie :**
    $$\frac{\partial [\lambda f(x)]}{\partial x} = \lambda \frac{\partial f(x)}{\partial x}$$

### B. Propriétés appliquées aux Formes Linéaires Généralisées
*   **Somme de formes linéaires :**
    $$\frac{\partial (a^t x + b^t x)}{\partial x} = a + b$$
*   **Forme linéaire avec matrice de passage :** Si la variable $x$ est transformée par une matrice fixe $X$ avant d'être projetée sur le vecteur de constantes $a$, la fonction s'écrit $f(x) = a^t X x$. Son gradient extrait la structure par la gauche :
    $$\frac{\partial (a^t X x)}{\partial x} = X^t a$$

### C. Dérivation de la structure d'erreur (Le cœur des MCO)
En économétrie, lors de la minimisation de la somme des carrés des résidus, on rencontre systématiquement la forme quadratique des écarts $g(x) = (X x)^t (X x)$.
Par application de la règle de retournement de la transposition ($(Xx)^t = x^t X^t$), l'expression se réécrit en sandwich autour de la matrice d'information : $g(x) = x^t (X^t X) x$.

Puisque le produit $X^t X$ est structuralement symétrique ($(X^t X)^t = X^t (X^t)^t = X^t X$), l'application de la Règle 2 donne :
$$\frac{\partial [x^t (X^t X) x]}{\partial x} = 2 (X^t X) x$$

### D. Dérivation des termes croisés mixtes
Si le modèle comporte une interaction asymétrique entre un vecteur fixe $a$ et la variable $x$ à travers une matrice $A$, sous la forme $f(x) = a^t A x$ :
$$\frac{\partial (a^t A x)}{\partial x} = A^t a$$
Si la matrice $A$ est symétrique ($A^t = A$), la formule se simplifie directement :
$$\frac{\partial (a^t A x)}{\partial x} = A a$$

# BLOC-NOTE DU QUANT : CONVEXITÉ MATRICIELLE ET OPTIMISATION SOUS CONTRAINTE

## 1. Caractérisation de la Convexité en Haute Dimension

Pour valider qu'un extremum est un minimum unique, la fonction de coût doit être strictement convexe (en forme de cuvette). En dimension 1, cette condition exige que la dérivée seconde soit positive ($f''(x) \ge 0$). En dimension supérieure, cette notion est portée par la **matrice Hessienne**.

Soit la forme quadratique générale $g(x) = x^t A x$ :
*   **Le Gradient (Dérivée première) :**
    $$\frac{\partial g}{\partial x} = 2Ax$$
*   **La Hessienne $H$ (Dérivée seconde) :**
    $$\frac{\partial^2 g}{\partial x^2} = \frac{\partial (2Ax)}{\partial x} = 2A$$

La positivité de la dérivée seconde ne pouvant pas se lire directement sur un tableau de constantes $2A$, on utilise la structure de la forme quadratique comme un opérateur de test en l'entourant par un vecteur de déplacement virtuel $dx$ :
$$\text{Courbure} = (dx)^t H (dx) = 2 \cdot (dx)^t A (dx)$$

Si ce produit en sandwich donne un nombre strictement positif pour n'importe quel déplacement, l'espace courbe vers le haut dans toutes les directions.

---

## 2. Le concept de Matrice Définie Positive

Une matrice carrée et symétrique $A \in \mathbb{R}^{M \times M}$ est dite **définie positive** (notée $A \succ 0$) si et seulement si sa forme quadratique associée renvoie un scalaire strictement positif pour tout vecteur de déplacement non nul $x$ :
$$x^t A x > 0 \quad \forall x \neq 0_M$$

### A. Le piège des signes des coefficients
Une erreur classique consiste à croire qu'une matrice est définie positive si tous ses coefficients sont positifs. C'est faux. Une matrice contenant des coefficients négatifs hors-diagonale peut être définie positive, et une matrice avec des coefficients positifs peut être indéfinie si les interactions écrasent la diagonale.

### B. Critère Universel des Valeurs Propres
Dans la pratique analytique, on détermine la signature de la matrice à l'aide de ses valeurs propres ($\lambda_i$) issues de sa diagonalisation (Théorème Spectral) :
*   **Matrice Définie Positive ($A \succ 0$) :** $\forall \lambda_i > 0$. Courbure strictement ascendante dans toutes les directions. **Minimum global unique garanti**.
*   **Matrice Semi-Définie Positive ($A \succeq 0$) :** $\forall \lambda_i \ge 0$. Courbure plate ou ascendante (gouttière). Multiplicité infinie de solutions minimales.
*   **Matrice Indéfinie :** Présence simultanée de valeurs propres positives et négatives. Topologie en selle de cheval (col). Les algorithmes d'optimisation ne peuvent pas converger car ils divergent vers l'infini négatif dans les directions associées aux valeurs propres négatives.

---

## 3. L'Optimisation sous Contrainte : Le Lagrangien

En finance quantitative, les choix d'allocation de paramètres sont bridés par des contraintes réelles (budget, exposition maximale, niveau de rendement cible). L'optimisation ne s'effectue plus librement au fond de la cuvette, mais uniquement le long de la ligne droite (ou de l'hyperplan) tracée par la contrainte.

La méthode des **multiplicateurs de Lagrange** fusionne la fonction objectif et la contrainte au sein d'une unique fonction élargie appelée le **Lagrangien** ($\mathcal{L}$).

### A. Formulation Générale
Soit une fonction objectif de risque à minimiser $g(x) = x^t A x$ soumise à une contrainte d'égalité linéaire budgétaire du type $x_1 + x_2 + \dots + x_M = 1$.

On commence par basculer tous les termes du même côté pour obtenir une contrainte égale à zéro :
$$x_1 + x_2 + \dots + x_M - 1 = 0$$

On introduit une variable artificielle appelée le **multiplicateur de Lagrange**, notée $\lambda$, qui va multiplier ce bloc de contrainte. La fonction globale s'écrit :
$$\mathcal{L}(x, \lambda) = x^t A x + \lambda \cdot (x_1 + x_2 + \dots + x_M - 1)$$

### B. Les Conditions du Premier Ordre (CPO)
Pour localiser l'optimum sous contrainte, on cherche le point d'annulation du gradient global du Lagrangien par rapport à l'ensemble de ses variables ($x$ et $\lambda$). Cela génère un système de $M+1$ équations à résoudre :
$$\begin{cases}
\frac{\partial \mathcal{L}}{\partial x_1} = 0 \\
\frac{\partial \mathcal{L}}{\partial x_2} = 0 \\
\vdots \\
\frac{\partial \mathcal{L}}{\partial \lambda} = 0 \implies \text{Fait ressortir la contrainte initiale intacte et garantit son respect strict.}
\end{cases}$$

### C. Interprétation économique de $\lambda$
Le multiplicateur de Lagrange $\lambda$ n'est pas qu'un outil de calcul. En économie quantitative, il représente le **prix d'ombre** (shadow price). Il mesure de combien la fonction de coût (le risque global) varierait si on desserrait la contrainte d'une unité (par exemple, si le budget passait de $1$ à $2$).

---

## 4. Cas Pratique Résolu : Allocation de Portefeuille à 2 Actifs

### A. Énoncé du problème
On cherche à minimiser la fonction de risque (variance) d'un portefeuille composé de deux crypto-actifs, le Bitcoin ($x_1$) et l'Ethereum ($x_2$) :
$$g(x_1, x_2) = 3x_1^2 + 5x_2^2$$
La matrice centrale associée est $A = \begin{pmatrix} 3 & 0 \\ 0 & 5 \end{pmatrix}$. Ses valeurs propres sont $3$ et $5$ (toutes deux $>0$). La matrice est **définie positive**, la fonction est strictement convexe : le minimum sera unique.

La contrainte budgétaire impose d'allouer exactement 100% de notre capital (soit la constante $1$) :
$$x_1 + x_2 = 1 \implies x_1 + x_2 - 1 = 0$$

### B. Construction du Lagrangien
$$\mathcal{L}(x_1, x_2, \lambda) = 3x_1^2 + 5x_2^2 + \lambda(x_1 + x_2 - 1)$$

### C. Calcul des Dérivées Partielles (Gradient complet posé à zéro)
1.  **Dérivée selon $x_1$ :**
    $$\frac{\partial \mathcal{L}}{\partial x_1} = 6x_1 + \lambda = 0 \implies x_1 = -\frac{\lambda}{6}$$
2.  **Dérivée selon $x_2$ :**
    $$\frac{\partial \mathcal{L}}{\partial x_2} = 10x_2 + \lambda = 0 \implies x_2 = -\frac{\lambda}{10}$$
3.  **Dérivée selon $\lambda$ :**
    $$\frac{\partial \mathcal{L}}{\partial \lambda} = x_1 + x_2 - 1 = 0 \implies x_1 + x_2 = 1$$

### D. Résolution Algébrique
On injecte les expressions de $x_1$ et $x_2$ dans la contrainte budgétaire :
$$\left(-\frac{\lambda}{6}\right) + \left(-\frac{\lambda}{10}\right) = 1$$
Mise au dénominateur commun (30) :
$$-\frac{5\lambda}{30} - \frac{3\lambda}{30} = 1 \implies -\frac{8\lambda}{30} = 1 \implies \lambda = -\frac{30}{8} = -\frac{15}{4}$$

On remplace la valeur de $\lambda$ trouvée pour isoler les allocations optimales :
*   $$x_1 = -\frac{(-15/4)}{6} = \frac{15}{24} = \frac{5}{8} = \mathbf{62,5\%}$$
*   $$x_2 = -\frac{(-15/4)}{10} = \frac{15}{40} = \frac{3}{8} = \mathbf{37,5\%}$$

### E. Interprétation du Sens Physique
L'Ethereum possédant un coefficient de risque plus lourd ($5$) que le Bitcoin ($3$), l'algorithme d'optimisation sous contrainte réduit mécaniquement son exposition à $37,5\%$, tout en garantissant le respect strict du budget global ($62,5\% + 37,5\% = 100\%$).
