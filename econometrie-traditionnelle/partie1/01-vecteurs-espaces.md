# CHAPITRE 1 : VECTEURS ET ESPACES VECTORIELS DE MARCHÉ

## 1. Introduction à la Numérisation des Actifs

En économétrie financière, pour étudier la trajectoire d'un actif, nous abandonnons la lecture brute des prix. Un actif à $100\$$ qui prend $2\$$ fournit le même effort économique qu'un actif à $10\$$. qui prend $0.20\$$. Nous travaillons donc exclusivement avec les **rendements périodiques** (exprimés en pourcentage ou en log-rendements).

Si nous observons le marché sur un historique fini de $T$ périodes (minutes, heures, jours), la trajectoire de performance de chaque actif se résume à une liste ordonnée de $T$ nombres réels. En algèbre linéaire, cette liste est appelée un **vecteur**.

Afin de séparer de manière étanche les unités de temps, l'économétrie utilise la **notation en colonne**. Pour un actif $A$ observé sur $T$ jours, le vecteur colonne s'écrit mathématiquement :

$$\mathbf{a} = \begin{bmatrix} a_1 \\ a_2 \\ \vdots \\ a_T \end{bmatrix}$$

Chaque ligne représente une coordonnée temporelle unique (la ligne $t$ correspond au rendement du Jour $t$).

---

## 2. Bloc A : Magnitude et Co-mouvement (Norme, Produit Scalaire, Cosinus)

### A. La Norme Euclidienne ($L_2$)
La **norme** d'un vecteur, notée $\Vert \mathbf{a} \Vert$, représente sa magnitude géométrique absolue (sa longueur en ligne droite depuis l'origine spatiale jusqu'au point de coordonnées).

Pour un espace à $T$ dimensions, elle se calcule en appliquant de manière généralisée le **théorème de Pythagore** : on fait la somme des carrés de tous les rendements, puis on prend la racine carrée du résultat.

$$\Vert \mathbf{a} \Vert = \sqrt{\sum_{t=1}^T a_t^2} = \sqrt{a_1^2 + a_2^2 + \dots + a_T^2}$$

*   **Intuition Financière :** Si le vecteur est centré sur sa moyenne, la norme est l'image directe de la **volatilité** (du risque) de l'actif. Une flèche géométriquement très longue trahit un actif hautement spéculatif. Une flèche très courte signale un actif stable.

### B. Le Produit Scalaire
Le **produit scalaire** entre deux vecteurs de rendements $\mathbf{a}$ et $\mathbf{b}$ (noté $\mathbf{a} \cdot \mathbf{b}$) est un opérateur qui croise leurs trajectoires période par période pour en additionner les résultats :

$$\mathbf{a} \cdot \mathbf{b} = \sum_{t=1}^T a_t b_t = a_1 b_1 + a_2 b_2 + \dots + a_T b_T$$

*   **Intuition Financière :** C'est le baromètre du co-mouvement. Si le Jour $t$, les deux actifs montent ensemble ($+ \times +$) ou baissent ensemble ($- \times -$), le produit est positif et fait grimper le score. S'ils bougent en opposition ($+ \times -$), le produit est négatif et fait baisser le score.
*   Un produit scalaire **très positif** indique des mouvements globalement synchrones.
*   Un produit scalaire **strictement égal à zéro** ($\mathbf{a} \cdot \mathbf{b} = 0$) prouve géométriquement que les deux flèches forment un angle droit parfait ($90^\circ$). Les deux vecteurs sont dits **orthogonaux**.

### C. La Similarité Cosinus
Le produit scalaire brut souffre d'un défaut : il est sensible aux effets d'échelle (la taille des flèches). Pour isoler la pure proximité directionnelle en neutralisant l'amplitude des variations, on divise le produit scalaire par le produit des normes individuelles. Cette formule dérive de la relation géométrique du lycée : $\mathbf{a} \cdot \mathbf{b} = \Vert \mathbf{a} \Vert \times \Vert \mathbf{b} \Vert \times \cos(\theta)$.

$$\cos(\theta) = \frac{\mathbf{a} \cdot \mathbf{b}}{\Vert \mathbf{a} \Vert \times \Vert \mathbf{b} \Vert}$$

*   **Intuition Financière :** Si les séries sont centrées, ce cosinus de l'angle $\theta$ formé par les deux flèches correspond exactement au **coefficient de corrélation linéaire de Pearson**.
    *   $\cos(\theta) = 1$ ($\theta = 0^\circ$) : Symbiose directionnelle parfaite.
    *   $\cos(\theta) = 0$ ($\theta = 90^\circ$) : Orthogonalité. Indépendance linéaire totale.
    *   $\cos(\theta) = -1$ ($\theta = 180^\circ$) : Opposition de phase absolue (idéal pour le *hedging*).

#### 📝 Application Numérique d'Exemple
Soient deux actifs, l'Or ($\mathbf{g}$) et le Pétrole ($\mathbf{p}$) observés sur deux jours de crise :
$$\mathbf{g} = \begin{bmatrix} 3 \\ 4 \end{bmatrix}, \quad \mathbf{p} = \begin{bmatrix} 4 \\ 3 \end{bmatrix}$$

1.  $\Vert \mathbf{g} \Vert = \sqrt{3^2 + 4^2} = \sqrt{9 + 16} = \sqrt{25} = 5$
2.  $\Vert \mathbf{p} \Vert = \sqrt{4^2 + 3^2} = \sqrt{16 + 9} = \sqrt{25} = 5$
3.  $\mathbf{g} \cdot \mathbf{p} = (3 \times 4) + (4 \times 3) = 12 + 12 = 24$
4.  $\cos(\theta) = \frac{24}{5 \times 5} = \frac{24}{25} = 0.96$

*Verdict :* L'angle entre l'Or et le Pétrole est infime ($\cos(\theta) \to 1$). Ils réagissent aux mêmes chocs directionnels.

---

## 3. Bloc B : Dépendance Linéaire et Méthodologie de la Preuve

### A. La Combinaison Linéaire
Créer une combinaison linéaire consiste à appliquer des coefficients multiplicateurs scalaires ($w_1, w_2, \dots, w_K$) à un ensemble de vecteurs de rendements, puis à sommer le tout. En trading, cela correspond à l'allocation de capital au sein d'un portefeuille :

$$\mathbf{r}_{\text{portefeuille}} = w_1 \mathbf{r}_1 + w_2 \mathbf{r}_2 + \dots + w_K \mathbf{r}_K = \begin{bmatrix} w_1 r_{1,1} + w_2 r_{2,1} + \dots \\ w_1 r_{1,2} + w_2 r_{2,2} + \dots \\ \vdots \end{bmatrix}$$

### B. Dépendance vs Indépendance Linéaire
*   Un ensemble de vecteurs est **linéairement dépendant** (colinéaire) si l'un d'eux peut s'exprimer comme une pure combinaison mathématique des autres (ex: $\mathbf{x}_2 = 2\mathbf{x}_1$). La deuxième variable n'est qu'une doublure qui n'apporte aucune épaisseur géométrique.
*   Un ensemble de vecteurs est **linéairement indépendant** si aucun d'eux ne peut être répliqué par les autres. Les branches du compas sont ouvertes, ouvrant de nouvelles dimensions d'information.

### C. Le Protocole de Preuve Scientifique Universel
Pour démontrer sur le papier et à coup sûr que deux vecteurs $\mathbf{a}$ et $\mathbf{b}$ sont indépendants, on pose par convention l'équation de la **combinaison nulle** :

$$w_1 \mathbf{a} + w_2 \mathbf{b} = \mathbf{0}$$

Si la résolution logique et algébrique prouve que la seule et unique solution possible sur Terre pour satisfaire cette équation est d'annuler tous les poids ($w_1 = 0$ et $w_2 = 0$), alors l'indépendance linéaire est **scientifiquement démontrée**. S'il existe des poids non nuls (ex: $w_1 = -2, w_2 = 1$), ils sont dépendants.

### 🛠️ Méthode Algébrique de Résolution (Coefficients Croisés)
Soit le système d'équations issu de la combinaison nulle sur deux périodes :
$$\begin{cases} L_1 : \quad \alpha_{1} w_1 + \beta_{1} w_2 = 0 \\ L_2 : \quad \alpha_{2} w_1 + \beta_{2} w_2 = 0 \end{cases}$$

Pour purger l'inconnue $w_1$ sans utiliser de fractions complexes, on croise les premiers coefficients en multipliant la ligne $L_1$ par $\alpha_2$ et la ligne $L_2$ par $\alpha_1$ :
$$\begin{cases} L_1' \leftarrow \alpha_{2} \times L_1 : \quad (\alpha_{2}\alpha_{1}) w_1 + (\alpha_{2}\beta_{1}) w_2 = 0 \\ L_2' \leftarrow \alpha_{1} \times L_2 : \quad (\alpha_{1}\alpha_{2}) w_1 + (\alpha_{1}\beta_{2}) w_2 = 0 \end{cases}$$

L'opération d'élimination par soustraction ($L_2' - L_1'$) isole le paramètre résiduel :
$$[\, \alpha_{1}\beta_{2} - \alpha_{2}\beta_{1} \,] w_2 = 0$$

*   **Règle de Diagnostic :** Si le bloc de gauche $[\alpha_{1}\beta_{2} - \alpha_{2}\beta_{1}]$ est différent de zéro, la seule solution est $w_2 = 0 \implies w_1 = 0$ (Indépendance). Si ce bloc vaut strictement zéro, alors $w_2$ peut prendre n'importe quelle valeur réelle, le système s'effondre dans le piège de la colinéarité.

---

## 4. Bloc C : Structure Spatiale (Rang et Dimension)

### A. Le Concept Fondamental du Rang
Le **Rang** d'un système ou d'une matrice représente **le nombre d'informations (ou de variables) réellement uniques, indépendantes et irremplaçables** qu'il contient après avoir nettoyé toutes les copies cachées.

*   *L'analogie des manettes :* Le Rang est le nombre exact de manettes indépendantes dont dispose un bot de trading pour piloter son capital. Si un junior donne 3 indicateurs au bot, mais que le troisième est une pure combinaison linéaire des deux premiers ($\mathbf{v}_3 = 2\mathbf{v}_1 + 3\mathbf{v}_2$), le troisième indicateur s'annule lors du nettoyage algébrique. Il ne reste que 2 manettes utiles : **Le Rang vaut 2**.
*   Si le Rang est égal au nombre initial de variables, le système est dit de **Rang Plein** (*Full Rank*). C'est la condition de sécurité absolue pour que les calculs de prédiction de prix (MCO) fonctionnent sans subir de division par zéro.

### B. Base et Couverture Dimensionnelle
Une **Base** est un ensemble de vecteurs linéairement indépendants qui engendre l'intégralité d'un espace vectoriel de marché.
*   **Base Canonique :** Formée par des actifs purement orthogonaux et de norme 1 alignés avec les axes temporels (ex: $\mathbf{e}_1 = \begin{bmatrix} 1 \\ 0 \end{bmatrix}$, $\mathbf{e}_2 = \begin{bmatrix} 0 \\ 1 \end{bmatrix}$). Trouver les allocations pour un objectif cible de performance y est direct.
*   **Base Non-Canonique :** Formée par des vecteurs indépendants mais non orthogonaux (ex: $\mathbf{a} = \begin{bmatrix} 1 \\ 2 \end{bmatrix}$, $\mathbf{b} = \begin{bmatrix} 3 \\ 1 \end{bmatrix}$). Ils forment un angle aigu mais permettent quand même de couvrir la totalité de l'espace de Rang 2, assurant au bot qu'il pourra atteindre n'importe quel profil de rendement cible par résolution algébrique.

**La Règle d'Or Temporelle :** Pour piloter efficacement des risques ou atteindre des objectifs cibles sur un horizon de $T$ jours, le bot de trading a impérativement besoin d'un univers d'actifs dont le **Rang est strictement égal à $T$**. S'il manque une dimension (Rang < $T$), il existera des journées entières de marché où le portefeuille sera mathématiquement incapable de se défendre.





















# FICHE TECHNIQUE QUANT - BLOC 1.A : VECTEURS DE MARCHÉ
## CONCEPTS : NORME, PRODUIT SCALAIRE ET SIMILARITÉ COSINUS

### 1. Formulations Mathématiques (Espace à T-Dimensions)
Soient $\mathbf{a}$ et $\mathbf{b}$ deux vecteurs colonnes de rendements financiers de dimension $(T \times 1)$ :

$$\mathbf{a} = \begin{bmatrix} a_1 \\ a_2 \\ \vdots \\ a_T \end{bmatrix}, \quad \mathbf{b} = \begin{bmatrix} b_1 \\ b_2 \\ \vdots \\ b_T \end{bmatrix}$$

*   **Norme Euclidienne ($L_2$) :** Mesure la magnitude géométrique absolue du vecteur. Si la série est centrée sur sa moyenne, la norme est l'image directe de la volatilité historique.
    $$\Vert \mathbf{a} \Vert = \sqrt{\sum_{t=1}^T a_t^2} = \sqrt{\mathbf{a}'\mathbf{a}}$$

*   **Produit Scalaire :** Baromètre du co-mouvement directionnel cumulé à travers le temps.
    $$\mathbf{a} \cdot \mathbf{b} = \sum_{t=1}^T a_t b_t = \mathbf{a}'\mathbf{b}$$

*   **Similarité Cosinus :** Extraction pure de l'alignement directionnel en neutralisant les effets d'échelle et d'amplitude. Strictement équivalent au coefficient de corrélation linéaire de Pearson si les variables sont centrées.
    $$\cos(\theta) = \frac{\mathbf{a} \cdot \mathbf{b}}{\Vert \mathbf{a} \Vert \times \Vert \mathbf{b} \Vert} = \frac{\mathbf{a}'\mathbf{b}}{\sqrt{\mathbf{a}'\mathbf{a}} \sqrt{\mathbf{b}'\mathbf{b}}}$$

### 2. Règles de Décision pour le Bot de Trading
*   **Si $\cos(\theta) \to 1$ :** Alignement directionnel fort (Symbiose). Les actifs partagent les mêmes facteurs de risque immédiats. Stratégie associée : Suivi de tendance croisée ou Momentum.
*   **Si $\cos(\theta) \to 0$ :** Orthogonalité stricte (Indépendance linéaire). L'actif B n'apporte aucune information prédictive linéaire sur l'actif A. Stratégie associée : Idéal pour maximiser la diversification du portefeuille.
*   **Si $\cos(\theta) \to -1$ :** Opposition directionnelle (Miroir). Stratégie associée : Couverture parfaite (*Hedging*) pour immuniser le capital contre les risques de marché.

# FICHE TECHNIQUE QUANT - BLOC 1.B : INDÉPENDANCE ET COLINÉARITÉ
## CONCEPTS : COMBINAISONS LINÉAIRES ET SÉCURITÉ DES SYSTÈMES

### 1. Formulations Mathématiques (Indépendance vs Dépendance)
Soit un ensemble de $K$ vecteurs de rendements ou d'indicateurs de marché $\mathbf{x}_1, \mathbf{x}_2, \dots, \mathbf{x}_K$ :

*   **Combinaison Linéaire :** Construction d'un vecteur synthétique par application de coefficients scalaires (poids d'allocation $w_k$).
    $$\mathbf{x}_{\text{synth}} = \sum_{k=1}^K w_k \mathbf{x}_k = w_1 \mathbf{x}_1 + w_2 \mathbf{x}_2 + \dots + w_K \mathbf{x}_K$$

*   **Condition d'Indépendance Linéaire :** L'ensemble est linéairement indépendant si et seulement si l'équation suivante n'admet qu'une unique solution triviale :
    $$\sum_{k=1}^K w_k \mathbf{x}_k = \mathbf{0} \implies w_1 = w_2 = \dots = w_K = 0$$

*   **Dépendance Linéaire (Colinéarité) :** Si un ensemble de poids non tous nuls permet d'obtenir le vecteur nul $\mathbf{0}$, alors au moins un des vecteurs est redondant.
    $$\text{Si } \mathbf{x}_2 = k\mathbf{x}_1 \implies [\,\mathbf{x}_1 \cdot \mathbf{x}_1 \times \mathbf{x}_2 \cdot \mathbf{x}_2\,] - (\mathbf{x}_1 \cdot \mathbf{x}_2)^2 = 0$$

### 2. Règles de Risque pour le Développeur de Bot
*   **Rejet de l'Alpha Fantôme :** Si une nouvelle stratégie de trading est une combinaison linéaire des anciennes stratégies, elle n'apporte aucune diversification. Elle doit être rejetée pour économiser les frais de friction (courtage).
*   **Protection Anti-Crash :** L'introduction de variables colinéaires ($\mathbf{x}_2 = k\mathbf{x}_1$) provoque une division par zéro lors de la résolution matricielle des Moindres Carrés Ordinaires. Le bot doit intégrer un filtre de sécurité mesurant le déterminant avant toute inversion.

# FICHE TECHNIQUE QUANT - BLOC 1.B (COMPLÉMENT) : MÉTHODOLOGIE DE LA PREUVE FORMELLE
## CONCEPTS : ALGORITHME CRÉATION DE ZÉROS ET DIAGNOSTIC DE SÉCURITÉ DU RANG

### 1. Protocole de Preuve Algébrique (Méthode des Coefficients Croisés)
Soit un système de dimensions temporelles réduites à deux indicateurs non proportionnels de prime abord :
$$\begin{cases} L_1 : \quad \alpha_{1} w_1 + \beta_{1} w_2 = 0 \\ L_2 : \quad \alpha_{2} w_1 + \beta_{2} w_2 = 0 \end{cases}$$

Pour purger l'inconnue $w_1$ sans générer de blocage par des fractions complexes, le moteur algébrique applique la transformation linéaire croisée suivante :
$$\begin{cases} L_1' \leftarrow \alpha_{2} \times L_1 : \quad (\alpha_{2}\alpha_{1}) w_1 + (\alpha_{2}\beta_{1}) w_2 = 0 \\ L_2' \leftarrow \alpha_{1} \times L_2 : \quad (\alpha_{1}\alpha_{2}) w_1 + (\alpha_{1}\beta_{2}) w_2 = 0 \end{cases}$$

L'opération d'élimination par soustraction ($L_2' - L_1'$) isole structurellement le paramètre résiduel :
$$[\, \alpha_{1}\beta_{2} - \alpha_{2}\beta_{1} \,] w_2 = 0$$

### 2. Interprétation Sécuritaire pour l'Algorithme
*   **Si $[\alpha_{1}\beta_{2} - \alpha_{2}\beta_{1}] \neq 0$ :** Le coefficient multiplicateur est une valeur réelle solide. L'unique solution mathématique est $w_2 = 0 \implies w_1 = 0$. Les vecteurs sont indépendants. Rang = 2 (Plein rang). Le bot valide la structure de données.
*   **Si $[\alpha_{1}\beta_{2} - \alpha_{2}\beta_{1}] = 0$ :** Le terme multiplicateur s'annule. L'équation devient $0 \times w_2 = 0$. Tout nombre réel sur Terre devient une solution valide pour $w_2$. Le système s'effondre en indétermination linéaire. Rang = 1. Le bot lève une exception de sécurité et rejette les variables.

# FICHE TECHNIQUE QUANT - BLOC 1.B (DIAGNOSTIC RAPIDE)
## CONCEPT : LE FILTRE GÉOMÉTRIQUE DE LA DÉPENDANCE LINÉAIRE

### 1. Règle de Décision par le Cosinus (Pour 2 variables)
Soit la similarité cosinus calculée entre deux vecteurs de caractéristiques de marché :
$$\cos(\theta) = \frac{\mathbf{a} \cdot \mathbf{b}}{\Vert \mathbf{a} \Vert \times \Vert \mathbf{b} \Vert}$$

*   **Condition de Dépendance Linéaire (Colinéarité) :**
    $$\text{Si } |\cos(\theta)| = 1 \iff \mathbf{a} \text{ et } \mathbf{b} \text{ sont linéairement dépendants.}$$
    *Le système est sous-déterminé, le modèle MCO va subir une division par zéro.*

*   **Condition d'Indépendance Linéaire :**
    $$\text{Si } |\cos(\theta)| \neq 1 \iff \mathbf{a} \text{ et } \mathbf{b} \text{ sont linéairement indépendants.}$$
    *Le système est solvable, le bot peut utiliser ces deux indicateurs en toute sécurité.*


# FICHE TECHNIQUE QUANT - BLOC 1.C : BASE ET DIMENSION SPATIALE
## CONCEPTS : REPRÉSENTATION VERTICALE ET COUVERTURE DE L'ESPACE DES TEMPALITÉS

### 1. Formulations Mathématiques (Espace Vectoriel Temporel)
Soit un univers de $K$ actifs représentés sous forme de vecteurs colonnes de dimension $(T \times 1)$ :

*   **Notation Colonne :** Structuration verticale des rendements où chaque ligne $t$ correspond de manière étanche à une unité de temps (période, bougie, milliseconde).
    $$\mathbf{v} = \begin{bmatrix} v_1 \\ v_2 \\ \vdots \\ v_T \end{bmatrix}$$

*   **Concept de Base :** Un ensemble de vecteurs $(\mathbf{e}_1, \mathbf{e}_2, \dots, \mathbf{e}_T)$ forme une base de l'espace si l'ensemble est linéairement indépendant et si tout objectif de rendement $\mathbf{y}$ peut s'exprimer de manière unique par leur combinaison linéaire :
    $$\mathbf{y} = w_1 \mathbf{e}_1 + w_2 \mathbf{e}_2 + \dots + w_T \mathbf{e}_T$$

*   **Dimension et Rang Plein :** La dimension de l'espace est le nombre maximal de vecteurs linéairement indépendants qu'il contient. Si le Rang d'une matrice d'actifs est strictement inférieur à la dimension temporelle $T$, le système est sous-déterminé (existence d'une infinité de solutions ou impossibilité stricte).

### 2. Application Pratique à l'Architecture du Bot
*   **Sélection d'Univers d'Actifs :** Le bot doit vérifier que les actifs inclus dans son portefeuille couvrent l'intégralité des dimensions de risques nécessaires. L'introduction d'actifs colinéaires (ex: Pétrole et Fioul) n'augmente pas le rang du système et laisse le bot sans défense face aux variations des dimensions non couvertes.
*   **Base Canonique de Marché :** L'utilisation d'actifs purement orthogonaux ($\mathbf{a} \cdot \mathbf{b} = 0$) simplifie drastiquement le calcul d'allocation de capital du bot en isolant proprement l'impact de chaque décision.
