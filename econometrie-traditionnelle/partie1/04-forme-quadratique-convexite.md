
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
