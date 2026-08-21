### 🔬 Laboratoire Expérimental de la Partie 1 — Expérience 1 : Valider le constat

L'objectif de cette expérience est de prouver de manière empirique et rigoureuse qu'une série financière n'est pas indépendante. Nous allons concevoir une méthode pas à pas pour tester et rejeter statistiquement l'hypothèse i.i.d. sur des données boursières réelles.

---

### 1. La Construction Mathématique de la Base de Données

Pour mener à bien cette expérience, vous devez collecter l'historique d'un indice boursier liquide (ex: le S&P 500 ou le CAC 40) sur une période de 5 ans, ce qui correspond à environ $T = 1\ 250$ jours de bourse ou observations. À partir des prix de clôture bruts, vous allez construire deux indicateurs mathématiques successifs.

#### Étape A : Calcul des Rendements Logarithmiques ($R_t$)
Soit $P_t$ le prix de clôture du jour $t$, et $P_{t-1}$ le prix de clôture de la veille. Le rendement quotidien se calcule via la formule :

$$R_t = \ln\left(\frac{P_t}{P_{t-1}}\right)$$

*   **Motivation de la métrique :** Pourquoi utiliser le logarithme népérien plutôt qu'un pourcentage de variation classique $(\frac{P_t - P_{t-1}}{P_{t-1}})$ ? Le rendement logarithmique possède une propriété mathématique cruciale : l'**additivité temporelle**. Si vous voulez calculer le rendement combiné du jour 1 et du jour 2, il vous suffit de faire la somme des deux rendements logarithmiques $(\ln(P_2/P_0) = \ln(P_2/P_1) + \ln(P_1/P_0))$. Cette linéarité rend les modélisations temporelles mathématiquement stables et rigoureuses.

#### Étape B : Calcul de l'Intensité du Risque ($R_t^2$)
Créez un nouveau vecteur où chaque rendement trouvé est élevé au carré : $R_t^2$.
*   **Motivation de la métrique :** Si vous mesurez le lien entre les rendements bruts $R_t$ d'un jour à l'autre, vous obtiendrez un résultat proche de 0. Le marché change de direction (positif ou négatif) de manière quasi-imprévisible. En revanche, si le marché subit un krach de $-3\%$, le carré devient $+9\%$. Si le lendemain il rebondit de $+2.5\%$, le carré vaut $+6.25\%$. Élever au carré élimine le signe pour **isoler uniquement l'amplitude du mouvement (la volatilité)**. C'est l'indicateur universel de l'intensité du risque.

---

### 2. Le Cœur de la Mesure : L'Autocorrélation Linéaire et le Choix de Pearson

#### Qu'est-ce que l'autocorrélation linéaire ?
Une corrélation standard mesure la force du lien entre deux variables distinctes (le poids et la taille d'un individu). L'**auto**-corrélation mesure le lien entre **une variable et elle-même, mais décalée dans le temps**. Ce décalage temporel s'appelle un *lag* (noté $k$) :
*   Un décalage de $k=1$ jour cherche à savoir si l'intensité du risque d'aujourd'hui est liée à celle d'hier.
*   Un décalage de $k=2$ jours regarde le lien entre aujourd'hui et avant-hier.

#### Pourquoi le choix s'est-il porté sur le coefficient de Pearson ?
Le coefficient de Pearson est la métrique mathématique de référence pour mesurer la relation **linéaire** entre deux listes de données. Une relation linéaire signifie que si la première liste augmente, la seconde liste augmente (ou diminue) de manière proportionnelle, formant graphiquement une ligne droite.

Le grand avantage du coefficient de Pearson est qu'il est **normalisé**. La formule intègre une division par la dispersion totale des données (les écarts-types), ce qui force le résultat final à être un chiffre pur, obligatoirement confiné entre **$-1$ et $+1$**. Cela rend son interprétation universelle :
*   **$+1$** : Relation positive parfaite (si le risque d'hier augmente, le risque d'aujourd'hui augmente obligatoirement dans la même proportion).
*   **$0$** : Absence totale de relation (les secousses d'hier ne donnent aucun indice sur le comportement du marché aujourd'hui).
*   **$-1$** : Relation négative parfaite (si le risque d'hier augmente, le risque d'aujourd'hui diminue obligatoirement de façon proportionnelle).

---

### 3. Application Numérique : Alignement et Calcul pas à pas

Pour comprendre la mécanique exacte que l'ordinateur va exécuter sur vos 1 250 jours de données, appliquons la formule de Pearson sur une mini-série de volatilité ($R_t^2$) mesurée sur **5 jours** :

$$\text{Série Temporelle d'Origine} : Y = [1\%, \ 4\%, \ 9\%, \ 4\%, \ 2\%]$$

Pour mesurer l'autocorrélation à court terme ($k=1$), l'algorithme va dupliquer la série et la décaler d'une ligne vers le bas afin d'aligner le jour présent avec la veille. On perd mécaniquement le premier jour car il n'a pas de "veille" dans l'échantillon.

L'ordinateur se retrouve face à deux listes synchronisées de 4 éléments :
*   $$\text{Liste 1 (Aujourd'hui)} : Y_{\text{Auj}} = [4\%, \ 9\%, \ 4\%, \ 2\%]$$
*   $$\text{Liste 2 (Hier)} : Y_{\text{Hier}} = [1\%, \ 4\%, \ 9\%, \ 4\%]$$

#### Exécution algébrique de Pearson :

**Étape 1 : Calcul des moyennes arithmétiques**
*   $\bar{Y}_{\text{Auj}} = (4 + 9 + 4 + 2) / 4 = 4.75\%$
*   $\bar{Y}_{\text{Hier}} = (1 + 4 + 9 + 4) / 4 = 4.5\%$

**Étape 2 : Calcul de la covariance (Numérateur de Pearson)**
On calcule l'écart de chaque point par rapport à sa moyenne respective, puis on multiplie ces écarts entre eux :
*   Point 1 : $(4 - 4.75) \times (1 - 4.5) = (-0.75) \times (-3.5) = 2.625$
*   Point 2 : $(9 - 4.75) \times (4 - 4.5) = (4.25) \times (-0.5) = -2.125$
*   Point 3 : $(4 - 4.75) \times (9 - 4.5) = (-0.75) \times (4.5) = -3.375$
*   Point 4 : $(2 - 4.75) \times (4 - 4.5) = (-2.75) \times (-0.5) = 1.375$

On fait la somme de ces résultats : $\text{Covariance} = 2.625 + (-2.125) + (-3.375) + 1.375 = \mathbf{-1.5}$

**Étape 3 : Calcul de la variance des listes (Dénominateur de Pearson)**
On calcule la dispersion totale des données face à leur moyenne :
*   $\text{Variance Auj} = [(-0.75)^2 + (4.25)^2 + (-0.75)^2 + (-2.75)^2] = 0.5625 + 18.0625 + 0.5625 + 7.5625 = 26.75$
*   $\text{Variance Hier} = [(-3.5)^2 + (-0.5)^2 + (4.5)^2 + (-0.5)^2] = 12.25 + 0.25 + 20.25 + 0.25 = 33$

**Étape 4 : Division finale**
Le coefficient d'autocorrélation de Pearson d'ordre 1 ($\rho_1$) vaut :

$$\rho_1 = \frac{\text{Covariance}}{\sqrt{\text{Variance Auj} \times \text{Variance Hier}}} = \frac{-1.5}{\sqrt{26.75 \times 33}} = \frac{-1.5}{\sqrt{882.75}} = \frac{-1.5}{29.711} \approx \mathbf{-0.0505}$$

---

### 4. Règle de Décision et Seuil Statistiques

Pour prouver que le coefficient obtenu sur votre échantillon réel de $T = 1\ 250$ jours n'est pas le fruit du hasard, vous devez implanter une règle de décision rigoureuse issue de la règle de Bartlett.

Au seuil de confiance de 95 %, la borne de tolérance maximale du hasard (le bruit de fond) est formulée par :

$$\text{Seuil de sécurité} = \frac{1.96}{\sqrt{T}} = \frac{1.96}{\sqrt{1250}} \approx \mathbf{0.055}$$

*   **Règle de décision :** Si le coefficient de Pearson est compris entre $-0.055$ et $+0.055$, la corrélation est statistiquement insignifiante. Le résultat est modélisé comme du pur bruit aléatoire. L'hypothèse i.i.d. est validée. Si le coefficient dépasse $+0.055$, l'hypothèse i.i.d. est formellement rejetée.

---

### 5. Ce que vous allez observer

En appliquant ce protocole sur les 1 250 jours de l'indice S&P 500, votre ordinateur va extraire des coefficients de Pearson pour les décalages $k=1$ à $k=10$. Vous allez constater que :
*   Le coefficient à très court terme $\rho_1$ oscille généralement entre $\mathbf{0.18}$ et $\mathbf{0.25}$.
*   Les coefficients pour les jours suivants ($\rho_2, \rho_3, \dots, \rho_{10}$) diminuent de manière très progressive, restant par exemple à $0.12$ au bout d'une semaine boursière ($k=5$).

---

### 6. Analyse des Résultats

Chacun des coefficients observés sur les données réelles se situe largement au-dessus de notre seuil critique de bruit fixé à $0.055$.

L'analyse de cet écart nous permet de rejeter mathématiquement le postulat i.i.d. : la volatilité des marchés financiers possède une mémoire persistante. Un choc d'amplitude aujourd'hui propage des ondes de corrélation linéaires qui influencent de manière mesurable le comportement des rendements sur plusieurs jours consécutifs.














### 🔬 Laboratoire Expérimental de la Partie 1 — Expérience 1 : Le Paradoxe des Séries Financières et la Validation du Constat

L'objectif de cette expérience est de prouver de manière empirique et rigoureuse qu'une série financière viole l'hypothèse i.i.d. Pour y parvenir, nous allons mettre en lumière un paradoxe fondamental des marchés : l'absence de mémoire linéaire dans les rendements bruts, qui cache en réalité une mémoire non linéaire profonde et persistante dans la volatilité.

---

### 1. La Construction Mathématique de la Base de Données

Pour mener à bien cette expérience, vous devez collecter l'historique d'un indice boursier liquide (ex: le S&P 500 ou le CAC 40) sur une période de 5 ans, ce qui correspond à environ $T = 1\ 250$ jours de bourse ou observations. À partir des prix de clôture bruts, vous allez construire deux indicateurs mathématiques successifs.

#### Étape A : Calcul des Rendements Logarithmiques ($R_t$)
Soit $P_t$ le prix de clôture du jour $t$, et $P_{t-1}$ le prix de clôture de la veille. Le rendement quotidien se calcule via la formule :

$$R_t = \ln\left(\frac{P_t}{P_{t-1}}\right)$$

*   **Motivation de la métrique :** Pourquoi utiliser le logarithme népérien plutôt qu'un pourcentage de variation classique $(\frac{P_t - P_{t-1}}{P_{t-1}})$ ? Le rendement logarithmique possède une propriété mathématique cruciale : l'**additivité temporelle**. Si vous voulez calculer le rendement combiné du jour 1 et du jour 2, il vous suffit de faire la somme des deux rendements logarithmiques $(\ln(P_2/P_0) = \ln(P_2/P_1) + \ln(P_1/P_0))$. Cette linéarité rend les modélisations temporelles mathématiquement stables et rigoureuses.

#### Étape B : Calcul de l'Intensité du Risque ($R_t^2$)
Créez un nouveau vecteur où chaque rendement trouvé est élevé au carré : $R_t^2$.
*   **Motivation de la métrique :** C'est le cœur de notre démonstration. Si le marché subit un choc de $-3\%$, le carré devient $+9\%$. Si le lendemain il rebondit de $+2.5\%$, le carré vaut $+6.25\%$. Élever au carré élimine le signe (la direction de la tendance) pour **isoler uniquement l'amplitude du mouvement (la volatilité)**. C'est l'indicateur universel de l'intensité du risque.

---

### 2. Le Cœur de la Mesure : L'Autocorrélation Linéaire et le Choix de Pearson

#### Qu'est-ce que l'autocorrélation linéaire ?
Une corrélation standard mesure la force du lien entre deux variables distinctes (le poids et la taille d'un individu). L'**auto**-corrélation mesure le lien entre **une variable et elle-même, mais décalée dans le temps**. Ce décalage temporel s'appelle un *lag* (noté $k$). Un décalage de $k=1$ jour cherche à savoir si la valeur d'aujourd'hui est liée à celle d'hier.

#### Pourquoi le choix s'est-il porté sur le coefficient de Pearson ?
Le coefficient de Pearson est la métrique mathématique de référence pour mesurer la relation **linéaire** entre deux listes de données. La formule intègre une division par la dispersion totale des données (les écarts-types), ce qui force le résultat final à être un chiffre pur, obligatoirement confiné entre **$-1$ et $+1$**. Cela rend son interprétation universelle : $+1$ signifie une relation positive parfaite, $0$ une absence totale de relation, et $-1$ une relation négative parfaite.

---

### 3. Application Numérique : Le Paradoxe des Deux Listes

Pour comprendre le paradoxe que l'ordinateur va mettre en évidence, appliquons la formule de Pearson sur une mini-série de **5 jours** en analysant d'abord les rendements bruts ($R_t$), puis la volatilité au carré ($R_t^2$).

Imaginons un marché très agité qui alterne hausses et baisses :
$$\text{Série des Rendements Bruts} : R = [+3\%, \ -3\%, \ +3\%, \ -3\%, \ +2\%]$$

#### Étape 1 : Analyse des Rendements Bruts ($R_t$)
Pour mesurer l'autocorrélation d'ordre 1 ($k=1$), l'algorithme décale la série d'une ligne pour aligner le jour présent avec la veille. On obtient deux listes synchronisées de 4 éléments :
*   $$\text{Liste 1 (Rendement Aujourd'hui)} : Y_{\text{Auj}} = [-3\%, \ +3\%, \ -3\%, \ +2\%] \quad (\text{Moyenne } \bar{Y}_{\text{Auj}} = -0.25\%)$$
*   $$\text{Liste 2 (Rendement Hier)} : Y_{\text{Hier}} = [+3\%, \ -3\%, \ +3\%, \ -3\%] \quad (\text{Moyenne } \bar{Y}_{\text{Hier}} = 0.00\%)$$

Exécutons le calcul de la covariance (le numérateur de Pearson) en multipliant les écarts à la moyenne :
*   $$\text{Jour 2} : (-3 - (-0.25)) \times (3 - 0.00) = -2.75 \times 3 = -8.25$$
*   $$\text{Jour 3} : (3 - (-0.25)) \times (-3 - 0.00) = 3.25 \times (-3) = -9.75$$
*   $$\text{Jour 4} : (-3 - (-0.25)) \times (3 - 0.00) = -2.75 \times 3 = -8.25$$
*   $$\text{Jour 5} : (2 - (-0.25)) \times (-3 - 0.00) = 2.25 \times (-3) = -6.75$$

$$\text{Somme de la Covariance} = -8.25 - 9.75 - 8.25 - 6.75 = \mathbf{-33}$$

Après division par les écarts-types (dénominateur), le coefficient de Pearson des rendements bruts s'établit à :
$$\rho_{1, \text{bruts}} \approx \mathbf{-0.94}$$
*Note théorique :* Dans cet exemple extrême, la mémoire est fortement négative. Sur un vrai marché de 1 250 jours, ces alternances de signes sont tellement chaotiques et imprévisibles (en raison de l'efficience des marchés) que la somme des covariances s'annule complètement. **Le coefficient final des rendements bruts s'effondre vers 0.**

#### Étape 2 : Analyse de la Volatilité au Carré ($R_t^2$)
Reprenons la même série, mais élevons chaque jour au carré pour isoler l'amplitude :
$$\text{Série de la Volatilité} : R^2 = [9\%, \ 9\%, \ 9\%, \ 9\%, \ 4\%]$$

L'algorithme réaligne les listes pour $k=1$ :
*   $$\text{Liste 1 (Volatilité Aujourd'hui)} : Y_{\text{Auj}}^2 = [9\%, \ 9\%, \ 9\%, \ 4\%] \quad (\text{Moyenne } \bar{Y}_{\text{Auj}}^2 = 7.75\%)$$
*   $$\text{Liste 2 (Volatilité Hier)} : Y_{\text{Hier}}^2 = [9\%, \ 9\%, \ 9\%, \ 9\%] \quad (\text{Moyenne } \bar{Y}_{\text{Hier}}^2 = 9.00\%)$$

Calculons la covariance des amplitudes :
*   $$\text{Jour 2} : (9 - 7.75) \times (9 - 9.00) = 1.25 \times 0 = 0$$
*   $$\text{Jour 3} : (9 - 7.75) \times (9 - 9.00) = 1.25 \times 0 = 0$$
*   $$\text{Jour 4} : (9 - 7.75) \times (9 - 9.00) = 1.25 \times 0 = 0$$
*   $$\text{Jour 5} : (4 - 7.75) \times (9 - 9.00) = -3.75 \times 0 = 0$$

$$\text{Somme de la Covariance} = 0 + 0 + 0 + 0 = \mathbf{0}$$
*Note théorique :* Dans ce mini-exemple, la volatilité est restée fixe au début, la covariance est nulle. Mais sur les vrais marchés financiers, la volatilité progresse par vagues (grappes). Si le marché subit une forte secousse à cause d'une crise, les jours suivants afficheront de grands carrés, qu'il s'agisse de fortes hausses ou de fortes baisses. **La covariance des carrés devient fortement positive.**

---

### 4. Règle de Décision et Seuil Statistiques

Pour prouver que les coefficients obtenus sur votre échantillon réel de $T = 1\ 250$ jours ne sont pas le fruit du hasard, vous appliquez la borne de tolérance maximale du hasard (le bruit de fond de la règle de Bartlett) au seuil de confiance de 95 % :

$$\text{Seuil de sécurité} = \frac{1.96}{\sqrt{T}} = \frac{1.96}{\sqrt{1250}} \approx \mathbf{0.055}$$

*   **Règle de décision :** Tout coefficient de Pearson compris entre $-0.055$ et $+0.055$ indique une absence de mémoire (données i.i.d.). Tout coefficient en dehors de cette borne valide l'existence d'une mémoire statistique.


###### 🧮 Focus Théorique : La Règle de Bartlett et le Seuil Statistique de Bruit

Pour interpréter rigoureusement les résultats de notre expérience, il ne suffit pas de constater qu'un coefficient de Pearson est égal à $0.08$ ou $0.22$. Nous devons déterminer si ce chiffre traduit une véritable mémoire ou s'il s'agit d'une simple fluctuation due au hasard. C'est ici qu'intervient la **Règle de Bartlett (1946)**.

---

###### 1. La Motivation Théorique de Bartlett

Si une série temporelle était *parfaitement* indépendante et identiquement distribuée (un bruit blanc i.i.d.), son autocorrélation théorique pour tout décalage $k > 0$ serait strictement égale à zéro ($\rho_k = 0$).

Cependant, lorsque l'on calcule ce coefficient sur un échantillon concret et fini de taille $T$, le hasard des tirages génère de petites oscillations. Maurice Bartlett a démontré mathématiquement que pour un échantillon de taille $T$ suffisamment grand, les coefficients d'autocorrélation empiriques d'un bruit blanc suivent une **loi normale** centrée sur $0$, dont la variance est inversement proportionnelle à la taille de l'échantillon :

$$\text{Variance Asymptotique} = \sigma^2(\rho_k) \approx \frac{1}{T}$$

L'écart-type de cette distribution (qui mesure la dispersion typique du hasard) est donc égal à :

$$\text{Écart-type} = \frac{1}{\sqrt{T}}$$

---

###### 2. Le Calcul Mathématique du Seuil (Intervalle de Confiance)

En statistique, pour rejeter une hypothèse avec un niveau de confiance de 95 %, on cherche à délimiter une zone qui englobe 95 % des résultats que le hasard peut produire.

Selon les propriétés de la loi normale (la courbe de Gauss), 95 % des observations se situent à l'intérieur d'un intervalle de **$\pm 1.96$ écarts-types** autour de la moyenne. En combinant la découverte de Bartlett et cette propriété universelle, nous obtenons la formule du seuil critique de bruit :

$$\text{Seuil de sécurité (Borne supérieure)} = 0 + 1.96 \times \frac{1}{\sqrt{T}} = \frac{1.96}{\sqrt{T}}$$

$$\text{Seuil de sécurité (Borne inférieure)} = 0 - 1.96 \times \frac{1}{\sqrt{T}} = -\frac{1.96}{\sqrt{T}}$$

---

###### 3. Application Numérique Rigoureuse

Appliquons cette formule aux deux échelles de notre laboratoire pour comprendre comment la taille de l'échantillon modifie la tolérance du statisticien.

###### Cas A : Notre échantillon réel de 5 ans ($T = 1\ 250$ jours)
$$\text{Seuil} = \frac{1.96}{\sqrt{1250}} = \frac{1.96}{35.3553} \approx \mathbf{0.0554}$$

*   **Règle de décision :** L'intervalle de tolérance du hasard est $[-0.0554 \ ; \ +0.0554]$.
    *   Si le coefficient de Pearson calculé par l'ordinateur tombe à l'intérieur de cette zone (ex: $\rho_1 = 0.02$), on dit que l'écart à zéro n'est pas statistiquement significatif. On accepte l'hypothèse i.i.d.
    *   Si le coefficient sort de cette zone (ex: $\rho_1 = 0.22$), l'écart est trop grand pour être l'œuvre de la chance. On rejette l'hypothèse i.i.d. avec un risque d'erreur inférieur à 5 %.

###### Cas B : Notre mini-exemple illustratif ($T = 5$ jours)
$$\text{Seuil} = \frac{1.96}{\sqrt{5}} = \frac{1.96}{2.2360} \approx \mathbf{0.8765}$$

*   **Règle de décision :** Sur un échantillon aussi minuscule, le hasard a le droit d'osciller violemment entre $[-0.8765 \ ; \ +0.8765]$. C'est pour cela que dans notre calcul pas à pas sur 5 jours, bien que le résultat soit spectaculaire ($\rho_1 = -0.0505$), il reste confiné dans la zone grise du hasard. On ne peut rien conclure sur 5 jours. Il faut un échantillon large ($T \ge 1\ 250$) pour que le seuil se resserre et devienne discriminant.


---

### 5. Ce que vous allez observer

En exécutant ce double test sur les 1 250 jours réels de votre indice boursier, vous allez observer un contraste saisissant :
*   **Sur les rendements bruts ($R_t$) :** Le coefficient de Pearson d'ordre 1 est statistiquement nul : $\rho_{1, \text{bruts}} \approx \mathbf{0.01}$. Il est inférieur au seuil de $0.055$.
*   **Sur la volatilité au carré ($R_t^2$) :** Le coefficient de Pearson d'ordre 1 devient fortement positif : $\rho_{1, \text{volatilité}} \approx \mathbf{0.22}$. Il dépasse largement le seuil de $0.055$.

---

### 6. Analyse des Résultats

Cette expérience démontre de manière éclatante le paradoxe des séries financières. Si l'on se contente d'analyser les rendements bruts, l'absence d'autocorrélation linéaire ($\rho \approx 0.01$) donne l'illusion que les données sont indépendantes et que le bootstrap classique d'Efron est applicable.

C'est un piège mathématique. En passant au carré, on révèle que les rendements partagent une dépendance non linéaire profonde : les amplitudes des mouvements sont corrélées entre elles. Les marchés ont une mémoire de la volatilité (le risque appelle le risque). Le postulat i.i.d. est donc fondamentalement invalidé, ce qui rend le bootstrap classique obsolète et impose de conserver la structure chronologique par le biais de blocs.
