## Section 1 : La Rupture de l'Indépendance et l'Échec du Bootstrap d'Efron

Le bootstrap traditionnel introduit par Bradley Efron en 1979 repose sur le postulat sous-jacent que les observations d'un échantillon sont indépendantes et identiquement distribuées ($i.i.d.$).

### 1. Limite Théorique en Présence de Dépendance
Soit $Y = [Y_1, Y_2, \dots, Y_T]$ une série temporelle où les observations sont liées par une structure de covariance non nulle :
$$\gamma_k = \text{Cov}(Y_t, Y_{t+k}) \neq 0 \quad \text{pour } k \neq 0$$

Le bootstrap d'Efron procède par tirage uniforme direct avec remise au sein du vecteur d'origine. La probabilité que l'observation $Y_{t+1}$ soit sélectionnée immédiatement après l'observation $Y_t$ dans l'échantillon rééchantillonné s'effondre à $1/T$. Le fil chronologique est brisé.

### 2. Conséquence Économétrique sur l'Évaluation du Risque
En détruisant la dépendance temporelle, le bootstrap d'Efron élimine la persistance des chocs (les grappes de volatilité). Les séries synthétiques générées présentent une structure de bruit blanc indépendant. Lors du calcul de métriques de risque à long terme (comme la Value-at-Risk ou l'Expected Shortfall cumulées), cette perte de mémoire conduit à une sous-estimation dramatique de la variance de la somme des rendements, masquant le risque de traîne et les scénarios de crise systémique.

---

## Section 2 : La Géométrie des Blocs Fixes — MBB et CBB

Pour préserver l'enchaînement chronologique des données, le rééchantillonnage doit s'effectuer non plus point par point, mais par pans d'observations contiguës appelés blocs.

### 1. Le Bootstrap par Blocs Mouvants (Moving Block Bootstrap - MBB)
Introduit par Künsch (1989), le MBB découpe une série de taille T en blocs superposables d'une longueur strictement fixe, notée $b$.

#### Algorithme de Construction
La fenêtre de lecture de taille $b$ glisse d'un jour à la fois le long de la série historique. Le nombre total de blocs fixes disponibles dans l'urne s'élève à :
$$N = T - b + 1$$
Chaque bloc individuel $B_i$ (où $i \in [1, N]$) démarrant à l'indice temporel $i$ est défini par le vecteur :
$$B_i = [Y_i, Y_{i+1}, \dots, Y_{i+b-1}]$$

#### Distorsion Géométrique des Bordures
La rigidité de la géométrie linéaire du MBB brise la propriété de stationnarité dans les séries rééchantillonnées. Les observations situées aux extrémités de l'historique initial possèdent une probabilité d'échantillonnage plus faible que les données centrales.
* L'observation initiale $Y_1$ n'appartient qu'au seul bloc $B_1$.
* L'observation centrale $Y_{T/2}$ appartient à $b$ blocs distincts (du bloc $B_{T/2 - b + 1}$ au bloc $B_{T/2}$).

Les points centraux reçoivent géométriquement plus de poids dans les simulations que les bordures, ce qui modifie la structure statistique locale de la série rééchantillonnée (le piège du trapèze).

### 2. Le Bootstrap Circulaire par Blocs (Circular Block Bootstrap - CBB)
Proposé par Politis et Romano (1992), le CBB élimine cette anomalie spatiale en enroulant l'axe du temps sur lui-même comme un anneau périodique.

#### Mécanisme de Circularité (Wrapping)
Le jour initial $Y_1$ devient mathématiquement le successeur chronologique direct du jour final $Y_T$. Les indices temporels sont ainsi évalués modulo T. Grâce à cette rotation, de nouveaux blocs mixtes apparaissent, et le nombre total de blocs disponibles dans l'urne devient strictement égal à la taille de l'échantillon :
$$N_{\text{CBB}} = T$$

Les blocs supplémentaires prennent la forme circulaire : $B_T = [Y_T, Y_1, \dots, Y_{b-1}]$. Chaque observation de l'historique initial apparaît désormais exactement $b$ fois dans l'urne globale. La probabilité d'échantillonnage devient rigoureusement uniforme ($b/T$) pour tous les points, résolvant la distorsion spatiale.

#### Le Piège de la Périodicité Artificielle
Bien que le CBB redistribue équitablement les chances de tirage, il échoue à garantir la stationnarité temporelle de la série simulée. Le choix d'une longueur de bloc fixe $b$ introduit des points de rupture déterministes. Deux observations adjacentes aux emplacements $s$ et $s+1$ dans la série simulée ont 100 % de chances d'être déconnectées et indépendantes si l'emplacement $s$ correspond à la fin d'un bloc pioché.

Cette rupture de mémoire se produit de manière rigide tous les $b$ jours, injectant un cycle harmonique artificiel de période $b$ (le piège des dents de scie) absent de la dynamique des marchés réels.

---

## Section 3 : L'Avènement du Bootstrap Stationnaire (Politis & Romano, 1994)

Pour gommer les frontières de rupture déterministes du CBB, Politis et Romano introduisent en 1994 une innovation conceptuelle : rendre la longueur des blocs aléatoire en la soumettant à une loi géométrique.

### 1. La Loi Géométrique et la Propriété de Perte de Mémoire
La longueur de chaque bloc L est une variable aléatoire discrète régie par une loi géométrique de paramètre $p$, où $p \in ]0, 1]$ représente la probabilité d'arrêt ou de coupure du bloc à chaque pas de temps. La loi géométrique possède la propriété unique de perte de mémoire (*memoryless property*) :
$$\mathbb{P}(L > n + m \mid L > n) = \mathbb{P}(L > m)$$

Peu importe le nombre de jours depuis lequel un bloc est en cours d'exécution, le risque qu'il se brise à l'étape suivante reste constant et égal à $p$. Le moment de la rupture devient totalement imprévisible, lissant les points de transition le long de la simulation et éliminant la périodicité artificielle. La longueur moyenne théorique d'un bloc est définie par l'espérance mathématique :
$$\mathbb{E}(L) = \frac{1}{p}$$

### 2. Algorithme d'Exécution Numérique Détaillé
Pour générer une série rééchantillonnée complète $Y^* = [Y^*_1, Y^*_2, \dots, Y^*_T]$, l'ordinateur utilise un pointeur de lecture réel noté K.
* **Initialisation (t = 1) :** L'ordinateur réalise un *Tirage de Position* uniforme discret sur l'ensemble de l'historique. Il sélectionne un entier au hasard entre 1 et T. Le pointeur prend cette valeur ($K = k_1$) and la première case reçoit la donnée : $Y^*_1 = Y_K$.
* **Boucle Temporelle (de t = 2 à T) :** À chaque étape, l'algorithme réalise un *Tirage de Continuité* (loi de Bernoulli de paramètre $p$) :
  * **Branche Rupture (Probabilité p) :** Le test renvoie 1. Le bloc se brise. L'ordinateur réinitialise le pointeur K via un nouveau tirage uniforme indépendant entre 1 et T. On assigne : $Y^*_t = Y_K$.
  * **Branche Prolongation (Probabilité 1-p) :** Le test renvoie 0. Le bloc continue. Le pointeur avance d'une unité : K = K + 1. Si le pointeur dépasse la fin de la série ($K = T + 1$), la règle du *wrapping* circulaire force un retour immédiat au début : K = 1. On assigne : $Y^*_t = Y_K$.

L'alliance de l'homogénéité spatiale (circularité) and de l'homogénéité temporelle (loi géométrique sans mémoire) garantit que la structure de corrélation de la série simulée ne dépend plus du temps, validant la stationnarité stricte du processus.
## Section 4 : La sélection automatique du paramètre de coupure optimal ($p$) selon Politis & White (2004)

Maintenant que la validité de la loi géométrique est démontrée, l'enjeu statistique réside dans la détermination rigoureuse de la probabilité de coupure $p$, ou de manière équivalente, de la longueur moyenne des blocs $\mathbb{E}(L) = 1/p$.

Pour éliminer l’arbitraire d’un choix manuel, Dimitris Politis et Halbert White ont introduit en 2004 un algorithme permettant d'automatiser ce calcul. La méthode repose sur l'analyse de la mémoire de la série d'origine par le biais de ses autocorrélations afin de minimiser mathématiquement l'Erreur Quadratique Moyenne (MSE).

---

### 1. La Fondation : La Règle de Bartlett et le Seuil Statistique de Bruit

Avant de chercher à localiser la fin de la mémoire d'une série temporelle, l'algorithme doit être capable de séparer le signal informatif réel du bruit de fond généré par le hasard. C'est l'objet de la règle de **Bartlett (1946)**.

#### La Formule de Bartlett Originale
Si notre série de volatilité ($Y_t = R_t^2$) était parfaitement indépendante (un bruit blanc i.i.d.), son autocorrélation théorique pour tout décalage $k > 0$ serait strictement nulle ($\rho_k = 0$). Cependant, sur un échantillon fini de taille $T$, le hasard produit des fluctuations. Maurice Bartlett a démontré mathématiquement que pour un échantillon suffisamment grand, la variance des autocorrélations empiriques d'un bruit blanc est égale à $1/T$.

L'écart-type de cette distribution (qui mesure la dispersion typique du hasard) est donc égal à $\frac{1}{\sqrt{T}}$. Pour construire un intervalle de confiance classique à **95 %** autour de zéro, on utilise la formule de Bartlett originale :

$$\text{Seuil}_{\text{Bartlett}} = \pm \ c \times \frac{1}{\sqrt{T}} = \pm \ \frac{1.96}{\sqrt{T}}$$

*   **Explication des constantes ($c = 1.96$) :** La constante $1.96$ provient directement de la table de la loi normale centrée réduite (la courbe de Gauss). Elle représente le nombre exact d'écarts-types nécessaires pour englober symétriquement **95 %** de la probabilité totale autour de la moyenne de zéro. Tout coefficient qui dépasse cette borne a moins de 5 % de chances d'avoir été généré par le pur hasard.

#### L'Ajustement de Politis & White (2004)
Pour calibrer automatiquement le bootstrap stationnaire, Politis et White décident de remplacer la constante fixe $1.96$ par une constante dynamique dépendante de la taille de l'échantillon. La formule du seuil réajustée s'établit ainsi :

$$\text{Seuil}_{\text{PW}} = \pm \ c_T \times \frac{1}{\sqrt{T}} = \pm \ 2 \times \sqrt{\frac{\ln(T)}{T}}$$

*   **Explication des constantes ($c_T = 2\sqrt{\ln(T)}$) :** Au lieu d'utiliser $1.96$, Politis et White appliquent un multiplicateur qui grandit avec $T$.
*   **Justification économétrique :** Dans un échantillon financier à haute fréquence (plusieurs milliers de jours), le fait de scanner simultanément un très grand nombre de décalages $k$ accroît mathématiquement le risque de détecter une "fausse corrélation" par pur hasard (le problème statistique des tests multiples). En injectant le logarithme népérien $\ln(T)$ au numérateur, Politis et White durcissent volontairement le critère statistique. Le seuil devient plus large lorsque la base de données s'agrandit, ce qui immunise l'algorithme contre le bruit de fond de long terme.

#### 🔢 Application numérique Comparative du Seuil
Analysons un historique boursier de **$T = 1\ 000$ jours**.

*   **Calcul du seuil selon Bartlett original (95 %) :**
    $$\text{Seuil}_{\text{Bartlett}} = \frac{1.96}{\sqrt{1000}} = \frac{1.96}{31.6227} \approx \mathbf{0.0620}$$

*   **Calcul du seuil adapté de Politis-White :**
    $$\text{Seuil}_{\text{PW}} = 2 \times \sqrt{\frac{\ln(1000)}{1000}} = 2 \times \sqrt{\frac{6.9077}{1000}} = 2 \times \sqrt{0.0069077} \approx \mathbf{0.1662}$$

*   **Interprétation comparative :** Avec Bartlett classique, un coefficient d'autocorrélation de $0.08$ serait considéré comme un signal valide ($0.08 > 0.0620$). Avec le critère sécurisé de Politis-White, ce même $0.08$ est jugé insuffisant et ravalé au rang de pur bruit de fond ($0.08 < 0.1662$). Ce filtre resserré évite de surévaluer la mémoire de la série.

### 2. Détection du Point de Rupture de la Mémoire ($k^*$)

L'objectif initial du scan de Politis et White est de trouver l'indice exact où la mémoire linéaire de la volatilité s'interrompt pour la première fois.

#### Définition et construction mathématique :
L'algorithme parcourt les autocorrélations une par une ($k=1, 2, 3 \dots$) et cherche le plus petit décalage, noté **$k^*$**, à partir duquel la valeur absolue de la corrélation passe sous le seuil critique :

$$k^* = \min \{ k \ge 1 \mid \vert \rho_k \vert < \text{Seuil}_{\text{PW}} \}$$

#### 🎯 Justification physique de l'arrêt au premier franchissement :
Dans la philosophie originale de Politis & White, le premier franchissement descendant du seuil indique le moment exact où le marché efface l'impact direct du choc initial. Dès que la corrélation s'écrase dans la zone grise du bruit blanc, l'algorithme estime que la dépendance temporelle active est brisée.

#### 🔢 Application numérique de la recherche de $k^*$ :
Reprenons nos $T = 1\ 000$ jours. L'ordinateur scanne les autocorrélations réelles calculées via la méthode de Pearson et les compare au seuil de $0.1662$ :
*   $k=1 \implies \rho_1 = 0.40$ (Supérieur à $0.1662 \rightarrow$ Signal)
*   $k=2 \implies \rho_2 = 0.25$ (Supérieur à $0.1662 \rightarrow$ Signal)
*   $k=3 \implies \rho_3 = 0.18$ (Supérieur à $0.1662 \rightarrow$ Signal)
*   $k=4 \implies \rho_4 = 0.08$ (Inférieur à $0.1662 \rightarrow$ **Coupure immédiate !**)

L'algorithme enregistre immédiatement :
$$\mathbf{k^* = 4}$$

---

### 3. La Fenêtre de Mémoire Globale ($M$) et Force du Signal ($G_{\text{PW}}$)

#### Définition et construction mathématique :
Pour s'assurer qu'aucune queue de corrélation tardive ou résiduelle n'a été oubliée lors de la coupure automatique, Politis et White appliquent un multiplicateur de sécurité multiplicatif pour définir la fenêtre de calcul globale $M$ :
$$M = 2 \times k^*$$

Le paramètre $G_{\text{PW}}$ calcule ensuite la somme des autocorrélations contenues dans la fenêtre $M$, pondérées linéairement par leur propre décalage temporel $k$ :
$$G_{\text{PW}} = \sum_{k=-M}^{M} \vert k \vert \cdot \rho_k = 2 \times \sum_{k=1}^{M} k \cdot \rho_k$$

#### 🎯 Justification économétrique et physique de l'architecture :
Le choix de doubler la valeur de $k^*$ est une précaution de lissage. Même si les corrélations individuelles passent sous le seuil à partir du jour $k^*$, l'accumulation de plusieurs petites corrélations résiduelles lointaines peut représenter une force d'inertie non négligeable.

On multiplie chaque autocorrélation par son décalage $k$ car une dépendance qui persiste loin dans le temps témoigne d'une inertie boursière plus lourde et complexe à répliquer. Multiplier par $k$ donne un poids géométrique dominant aux mémoires longues, allongeant la taille des blocs.

#### 🔢 Application numérique de $M$ et $G_{\text{PW}}$ :
Avec notre point de rupture $k^* = 4$ validé à la section précédente, le calcul de la fenêtre globale donne :
$$M = 2 \times 4 = \mathbf{8 \text{ jours}}$$

L'ordinateur calcule la force de la mémoire cumulée sur notre fenêtre de 8 jours (en considérant que les corrélations des jours 5 à 8 oscillent près de zéro et s'annulent) :
$$\sum_{k=1}^{8} k \cdot \rho_k = (1 \times 0.40) + (2 \times 0.25) + (3 \times 0.18) + (4 \times 0.08) + 0 + 0 + 0 + 0 = 1.76$$
$$G_{\text{PW}} = 2 \times 1.76 = \mathbf{3.52}$$

---

### 4. L'Impact de la Variance ($\sigma_Y^2$), de l'Échelle ($T^{-1/3}$) et Synthèse Finale

#### Définition et construction mathématique :
L'équation finale d'optimisation asymptotique réunit les forces antagonistes sous la forme suivante :
$$p_{\text{optimal}} = \left( \frac{\sigma_Y^2}{G_{\text{PW}}^2} \right)^{1/3} \times T^{-1/3}$$

#### 🎯 Justification des paramètres :
*   **La Variance brute ($\sigma_Y^2$) :** Plus la variance brute d'une série est élevée, plus le risque d'instabilité des simulations est grand. Pour stabiliser l'estimateur, la formule réagit en augmentant la valeur du numérateur, ce qui tire $p$ vers le haut (donc raccourcit les blocs) afin d'augmenter le nombre de blocs combinables et de faire baisser la variance du modèle bootstrapé.
*   **Le Facteur d'Échelle ($T^{-1/3}$) :** Plus votre historique $T$ grandit, plus la valeur de $T^{-1/3}$ diminue. Cette décroissance asymptotique garantit que lorsque vous disposez de très grandes bases de données, le paramètre $p$ diminue, allongeant la longueur des blocs ($\mathbb{E}(L)=1/p$). Disposant de plus de données, l'algorithme déploie de grands blocs sans risquer d'étouffer la diversité des tirages.

#### 🔢 Application numérique d'assemblage final :
Injectons nos valeurs chiffrées : $\sigma_Y^2 = 1.50$, $G_{\text{PW}} = 3.52$ (donc $G_{\text{PW}}^2 = 12.3904$) and $T^{-1/3} = 1000^{-1/3} = 0.10$.
1.  **Rapport des forces internes (Variance vs Mémoire) :** $\frac{1.50}{12.3904} \approx 0.12106$
2.  **Extraction de la racine cubique du rapport :** $(0.12106)^{1/3} \approx 0.4947$
3.  **Ajustement final par le facteur d'échelle temporel :** $p_{\text{optimal}} = 0.4947 \times 0.10 \approx \mathbf{0.0495}$

*Verdict mathématique :* La longueur moyenne idéale de nos blocs variables pour le bootstrap stationnaire sera de : $\mathbb{E}(L) = \frac{1}{0.0495} \approx \mathbf{20.2 \text{ jours}}$. Des blocs moyens de 20 jours minimisent la MSE selon Politis-White.

## Section 5 : La correction d'Andrew Patton (2009) par les Noyaux à Toit Plat

En 2009, l'économètre Andrew Patton démontre que l'algorithme de Politis & White (2004) souffre d'un défaut structurel majeur lorsqu'il est appliqué aux séries financières réelles : il s'arrête trop tôt dès qu'une autocorrélation franchit le seuil à la baisse, ignorant les phénomènes de **résurgence** (lorsque la corrélation remonte après une baisse locale).

Numériquement, Politis-White sous-estime la mémoire longue et extrait des blocs trop courts. Patton y apporte un correctif fondé sur l'utilisation du **noyau à toit plat** (*Flat-Top Kernel* de Politis et Romano) et redéfinit la formule asymptotique de la longueur de bloc idéale.

---

### 1. Le Noyau à Toit Plat : L'outil de pondération géométrique

Pour éviter d'étouffer les signaux lointains ou les structures de dépendance complexes, Patton remplace le traitement brut par une fonction de pondération, notée $\lambda(k)$, qui applique un filtre géométrique aux décalages $k$ :

$$\lambda(k) = \begin{cases} 1 & \text{si } 0 \le \frac{k}{M} < 0.5 \\ 2 \left(1 - \frac{k}{M}\right) & \text{si } 0.5 \le \frac{k}{M} \le 1 \\ 0 & \text{si } \frac{k}{M} > 1 \end{cases}$$

#### 🎯 Justification physique et économétrique :
Le profil de ce filtre dessine graphiquement un « toit plat » [🌐-0]. Tant que le décalage $k$ se situe dans la première moitié de la fenêtre de mémoire $M$, son poids reste rigoureusement de **1**. L'algorithme conserve 100 % de l'intensité du signal sans aucune dégradation.

Au-delà de la moitié de $M$, le poids descend en ligne droite pour amortir progressivement le bruit de fond, avant de s'annuler complètement après $M$. Cela permet de capturer les traînes de mémoire des crises financières sans pénaliser artificiellement leur distance géographique sur l'axe du temps.

---

### 2. Le calcul de la variance de long terme réajustée ($D_{\text{SB}}$)

C'est la correction la plus profonde de Patton. Politis-White utilisaient la variance brute $\sigma_Y^2$, qui postule implicitement que les données sont indépendantes. Patton la remplace par un estimateur de la **variance de long terme spécifique au bootstrap stationnaire**, noté $D_{\text{SB}}$ :

$$D_{\text{SB}} = 4 \times \left( \rho_0 \cdot \sigma_Y^2 + 2 \times \sum_{k=1}^{M} \lambda(k) \cdot \gamma_k \right)$$

Où :
*   $\gamma_k$ est l'**autocovariance brute** au décalage $k$, définie par $\gamma_k = \rho_k \times \sigma_Y^2$.
*   $\rho_0$ représente la corrélation d'un point avec lui-même, valant toujours 1.

#### 🎯 Justification physique et économétrique :
Le paramètre $D_{\text{SB}}$ réalise la somme de toutes les covariances croisées de la série temporelle, filtrées par le toit plat. Il mesure la variabilité réelle des blocs entre eux lorsque les chocs se propagent.

Dans une série à forte mémoire (comme la volatilité financière), les autocovariances $\gamma_k$ sont positives et s'accumulent. Par conséquent, $D_{\text{SB}}$ devient extrêmement élevé, bien plus grand que la simple variance brute d'un jour isolé ($\sigma_Y^2$). Placé au dénominateur de la formule de Patton, ce paramètre va agir comme un contrepoids pour stabiliser la taille des blocs.

### 3. La Vraie Formule Finale de Patton pour la Longueur de Bloc ($b_{\text{SB, opt}}$)

En combinant ces forces, Andrew Patton formule l'équation de la **longueur moyenne idéale des blocs** à injecter dans le bootstrap stationnaire pour minimiser la MSE :
$$b_{\text{SB, opt}} = \left( \frac{2 \cdot G_{\text{Patton}}^2}{D_{\text{SB}}} \right)^{1/3} \times T^{1/3}$$

La probabilité de coupure finale $p_{\text{optimal}}$ s'obtient ensuite par l'inversion géométrique : $p_{\text{optimal}} = \frac{1}{b_{\text{SB, opt}}}$.

#### 🎯 Justification de la cohérence logique :
*   **Le rôle de $G_{\text{Patton}}$ (au numérateur) :** Plus une série possède une mémoire persistante et longue, plus $G_{\text{Patton}}$ est grand. Situé au numérateur, il fait **augmenter** $b_{\text{SB, opt}}$. L'algorithme commande de fabriquer des blocs **plus longs** pour encapsuler cette mémoire. La logique est respectée.
*   **Le rôle du facteur d'échelle $T^{1/3}$ :** Plus vous avez de données ($T$), plus le multiplicateur grandit, ce qui permet d'allonger la taille des blocs sans risquer d'étouffer la diversité des tirages.

---

### 4. Application Numérique Comparative Rigoureuse et Cohérente

Soumettons un échantillon boursier de $T = 1\ 000$ jours aux deux modèles ($T^{1/3} = 10$). La variance brute de notre série est $\sigma_Y^2 = 1.50$. Le seuil de bruit calculé est de **0.1662**.
L'historique réel affiche les autocorrélations suivantes :
$$\rho_1 = 0.40, \ \rho_2 = 0.25, \ \rho_3 = 0.18, \ \rho_4 = 0.08, \ \rho_5 = 0.19 \text{ (résurgence)}, \ \rho_6 = 0.05, \ \rho_7 = 0.02, \ \rho_8 = 0.01$$

#### Étape A : Le calcul de Politis-White (2004)
Politis-White scanne la liste. Au jour 4, $\rho_4 = 0.08 < 0.1662$. L'algorithme se coupe immédiatement à cet endroit et ignore la suite.
*   Point de rupture : $\mathbf{k^* = 4} \implies \text{Fenêtre } \mathbf{M = 8 \text{ jours}}$
*   Calcul de la mémoire cumulée : $G_{\text{PW}} = 2 \times \sum_{k=1}^{8} k \cdot \rho_k = \mathbf{6.46} \implies G_{\text{PW}}^2 = \mathbf{41.7316}$
*   Résolution de l'équation de la longueur de bloc Politis-White :
    $$b_{\text{PW, opt}} = \left( \frac{1.50}{41.7316} \right)^{1/3} \times 10 = (0.03594)^{1/3} \times 10 \approx \mathbf{3.30 \text{ jours}} \implies p \approx \mathbf{0.3030}$$

#### Étape B : Le calcul réajusté de Patton (2009)
Le filtre à noyau plat de Patton refuse la coupure au jour 4 car la plage de validation exige que les autocorrélations restent continuellement sous le seuil pendant $K_T = 2\sqrt{\log_{10}(1000)} \approx 3$ jours. Il détecte que le signal remonte à 0.19 au jour 5. Le vrai silence s'installe au jour 6 ($\rho_6=0.05$).
*   Point de rupture de Patton : $\mathbf{k^* = 6} \implies \text{Fenêtre } \mathbf{M = 12 \text{ jours}}$
*   Calcul de la fonction de poids $\lambda(k)$ du noyau plat pour $M=12$ : De $k=1$ à 6 ($k/12 \le 0.5 \implies \lambda(k) = \mathbf{1.00}$). Pour $k=7 \dots 12$, le poids décroît.
*   Calcul de la mémoire cumulée étendue $G_{\text{Patton}}$ : $G_{\text{Patton}} = 2 \times \sum_{k=1}^{12} k \cdot \lambda(k) \cdot \rho_k = \mathbf{6.3067} \implies 2 \cdot G_{\text{Patton}}^2 = \mathbf{79.549}$
*   Calcul de la variance de long terme $D_{\text{SB}}$ : L'accumulation des autocovariances positives sur cette série à mémoire longue fait grimper la variance globale à : $\mathbf{D_{\text{SB}} = 3.20}$.
*   Résolution de l'équation de la longueur de bloc de Patton :
    $$b_{\text{SB, opt}} = \left( \frac{79.549}{3.20} \right)^{1/3} \times 10 = (24.859)^{1/3} \times 10 \approx \mathbf{29.19 \text{ jours}} \implies p \approx \mathbf{0.0342}$$

---

### 5. Ce que l'on interprète (Le verdict économétrique)

La comparaison arithmétique de nos deux résultats validés met en évidence la puissance et la cohérence théorique du correctif de Patton :

*   **L'erreur de Politis-White ($b \approx 3.3$ jours) :** En ayant les yeux rivés sur le premier franchissement du seuil, l'algorithme a été piégé par la baisse locale du jour 4. Il conclut qu'il n'y a plus de mémoire et impose des blocs très courts (3 jours). Vos simulations détruiront la forte résurgence du jour 5 ($\rho_5 = 0.19$), biaisant dramatiquement l'évaluation du risque de votre portefeuille.
*   **La correction de Patton ($b \approx 29.2$ jours) :** Grâce au lissage du noyau à toit plat, Patton ne se laisse pas abuser par le faux calme du jour 4. Il intercepte le signal tardif du jour 5 et intègre la variance de long terme ($D_{\text{SB}}$). L'équation ordonne d'utiliser des blocs **neuf fois plus longs** (29 jours au lieu de 3) pour sanctuariser les structures de corrélation complexes et la mémoire longue des marchés financiers.

## Section 6 : Application pratique aux Tests d'Hypothèses et à la finance

Dans les parties précédentes, nous avons vu comment construire géométriquement des mondes parallèles stationnaires et comment calibrer automatiquement la longueur moyenne de leurs blocs en minimisant la MSE. Nous allons maintenant voir comment utiliser concrètement ces simulations pour réaliser un **test d'hypothèse rigoureux** en finance quantitative.

L'exemple standard est le suivant : un gérant de fonds prétend avoir inventé une stratégie de trading algorithmique performante. Comment prouver mathématiquement que ses gains sont dus à un vrai savoir-faire et non au simple hasard ?

---

### 1. La formulation des Hypothèses Statistiques ($H_0$ et $H_1$)

Avant de lancer le moindre calcul, nous devons définir le cadre logique du test en formalisant deux hypothèses exclusives. Soit $\mu$ la performance quotidienne moyenne réelle à long terme de la stratégie du gérant.

*   **L'Hypothèse Nulle ($H_0$) :** Le gérant n'a aucun talent. Sa performance réelle est inférieure ou égale à zéro ($\mu \le 0$). Si ses résultats passés semblent positifs, ce n'est qu'une anomalie temporaire due à la chance (un bruit blanc chanceux).
*   **L'Hypothèse Alternative ($H_1$) :** Le gérant possède un avantage statistique réel. Sa performance moyenne est strictement positive ($\mu > 0$).

---

### 2. Le concept de "Centralisation" des données sous $H_0$

C'est l'étape la plus critique du test. Pour évaluer la probabilité que le hasard imite le gérant, nous devons créer un monde virtuel où **l'hypothèse nulle $H_0$ est strictement vraie**. On ne peut pas simplement bootstrapper la série de rendements telle quelle, car si le gérant a un vrai talent, sa série d'origine aura une moyenne positive, et le bootstrap reproduira cette moyenne positive.

#### La transformation mathématique :
Pour forcer le monde simulé à obéir à $H_0$, l'algorithme prend la série des rendements historiques de la stratégie ($X_t$), calcule sa moyenne historique ($\bar{X}$), et la soustrait à chaque point de la série. On obtient une série modifiée, dite "centralisée", notée $X^\dagger_t$ :

$$X^\dagger_t = X_t - \bar{X}$$

*   **Motivation :** Par construction, la moyenne arithmétique de cette nouvelle série $X^\dagger$ est **exactement égale à 0.00%**. Elle représente mathématiquement un gérant qui ne gagne rien (le hasard pur), mais elle conserve l'intégralité de la structure de volatilité, d'autocorrélation et de mémoire de la série d'origine.

#### 🔢 Application numérique de la Centralisation :
Supposons que sur un historique de $T = 1\ 000$ jours, l'algorithme du gérant affiche un rendement moyen quotidien de $\bar{X} = +0.05\%$.
*   Si au jour 12, le fonds a gagné $+0.12\%$, sa valeur centralisée devient : $X^\dagger_{12} = 0.12\% - 0.05\% = \mathbf{+0.07\%}$.
*   Si au jour 13, le fonds a perdu $-0.08\%$, sa valeur centralisée devient : $X^\dagger_{13} = -0.08\% - 0.05\% = \mathbf{-0.13\%}$.


### 3. La génération des Mondes Parallèles par Bootstrap Stationnaire

Une fois la série sous $H_0$ construite, l'ordinateur va simuler l'histoire du marché un grand nombre de fois (noté $M = 10\ 000$ réplications).

#### Le Protocole Algorithmique :
1. L'ordinateur prend le paramètre $p_{\text{optimal}}$ calculé automatiquement (par exemple $p = 0.0342$, soit des blocs moyens de $29.2$ jours).
2. Pour chaque simulation $m$ (allant de 1 à 10 000), il applique l'algorithme de tirage stationnaire (loi géométrique + circularité) sur la série centralisée $X^\dagger$.
3. Il génère ainsi une trajectoire alternative de 1 000 jours.
4. Pour cette trajectoire $m$, il calcule la performance moyenne obtenue par le hasard, notée $\bar{X}^{*(m)}$.

---

### 4. Le calcul de la $P$-Value Empirique et la Règle de Décision

Nous disposons maintenant d'une liste de 10 000 moyennes quotidiennes générées par un monde où le gérant n'a aucun talent. La **p-value** est la probabilité empirique que le hasard parvienne à égaler ou dépasser le score réel obtenu par le gérant ($+0.05\%$).

#### La formule de la P-Value :
L'ordinateur compte le nombre de simulations où la moyenne bootstrapée est supérieure ou égale à la moyenne historique réelle, et divise par le nombre total de réplications :

$$\text{P-value} = \frac{1}{M} \sum_{m=1}^M \mathbb{I}\left(\bar{X}^{*(m)} \ge \bar{X}\right)$$
*(Où $\mathbb{I}$ est une fonction indicatrice qui vaut 1 si la condition est vraie, 0 sinon).*

#### La Règle de Décision :
On compare cette p-value au seuil de signification alpha traditionnel de **5 % (0.05)**.
*   **Si P-value $\ge$ 0.05 :** Le hasard arrive fréquemment à produire une performance de $+0.05\%$. L'écart n'est pas significatif. On **accepte $H_0$**, le gérant est rejeté.
*   **Si P-value < 0.05 :** Il y a moins de 5 % de chances que le hasard pur reproduise un tel résultat tout en respectant la structure de mémoire du marché. On **rejette $H_0$** au profit de $H_1$. Le talent du gérant est statistiquement validé.

---

### 5. Application Numérique et Analyse de deux cas de figure

Imaginons deux gérants différents, Paul et Virginie, ayant tous deux obtenu la même performance moyenne de $\bar{X} = +0.05\%$ sur 1 000 jours. L'ordinateur fait tourner les 10 000 simulations du hasard sous $H_0$ pour chacun d'eux.

#### Cas A : L'analyse du dossier de Paul
L'ordinateur trie les 10 000 moyennes générées par le bootstrap stationnaire sous $H_0$ pour la stratégie de Paul. Il constate que dans **1 420 mondes parallèles**, le hasard a réussi à sortir une moyenne supérieure ou égale à $+0.05\%$.
*   **Calcul de la P-value :** $\text{P-value} = \frac{1420}{10000} = \mathbf{0.1420}$ (soit 14.20 %).
*   **Verdict et Justification :** $0.1420 > 0.05$. On accepte $H_0$. Bien que la moyenne de Paul soit positive en apparence, le bootstrap stationnaire démontre qu'elle est statistiquement banale. Si la stratégie de Paul consistait par exemple à suivre bêtement la tendance, et que le marché a connu une longue phase haussière (mémoire longue), le hasard des blocs de 29 jours va recréer cette phase haussière très facilement. Paul a juste été chanceux.

#### Cas B : L'analyse du dossier de Virginie
L'ordinateur trie les 10 000 moyennes générées pour la stratégie de Virginie. Il constate que dans seulement **12 mondes parallèles**, le hasard a réussi à atteindre ou dépasser $+0.05\%$.
*   **Calcul de la P-value :** $\text{P-value} = \frac{12}{10000} = \mathbf{0.0012}$ (soit 0.12 %).
*   **Verdict et Justification :** $0.0012 < 0.05$. On rejette vigoureusement $H_0$. Il y a seulement 1 chance sur 833 pour que le hasard, contraint par la mémoire des blocs du marché, reproduise la performance de Virginie. Sa régularité et sa structure de gains ne peuvent pas être imitées par la chance. Sa stratégie possède un véritable pouvoir prédictif.

## Section 7 : Le piège du Data Mining et la correction de White (Reality Check)

Dans la Partie 6, nous avons validé la stratégie de Virginie car sa P-value était extrêmement faible (0.12 %). Cependant, imaginez maintenant la situation suivante : une grande banque ne teste pas une seule stratégie, mais **1 000 algorithmes différents en même temps** (par exemple en faisant varier les indicateurs, les moyennes mobiles ou les seuils).

Si l'on applique le test de la Partie 6 sur chacun de ces 1 000 algorithmes, un problème mathématique majeur apparaît : le piège du **Data Mining** (ou surapprentissage). La correction de **Halbert White (2000)**, connue sous le nom de *Reality Check*, résout ce biais.

---

### 1. Qu'est-ce que le piège du Data Mining ? (Biais des tests multiples)

Si vous lancez une pièce de monnaie équilibrée 10 fois de suite, la probabilité d'obtenir 10 fois « Pile » est infime ($1/2^{10} \approx 0.1\%$). Si un individu réussit cet exploit du premier coup, vous en déduisez qu'il a triché ou qu'il possède un don.

En revanche, si vous réunissez **10 000 personnes** dans une pièce et que vous leur demandez à toutes de lancer une pièce 10 fois de suite, la loi des grands nombres garantit que statistiquement, environ 10 personnes obtiendront 10 fois « Pile » par pur hasard. Si vous ne filmez que ces 10 gagnants en ignorant la foule, vous allez croire qu'ils ont un don, alors qu'ils ne sont que les rescapés d'un processus aléatoire de masse.

#### En finance quantitative :
C'est exactement ce qui se passe lorsqu'un fonds d'investissement fait tourner des supercalculateurs pour tester des milliers de combinaisons de trading. Même si les marchés sont parfaitement efficients et qu'aucune stratégie ne fonctionne, le hasard pur fera émerger quelques algorithmes exceptionnellement rentables sur l'historique passé. C'est le biais du **Data Mining**. Si vous appliquez le test individuel de la Partie 6 sur la meilleure stratégie de la foule, sa P-value sera biaisée et vous conclurez à tort qu'elle a du talent.

---

### 2. Le fonctionnement du Reality Check de White (2000)

Pour corriger ce biais, Halbert White a proposé une idée révolutionnaire : au lieu d'évaluer si la meilleure stratégie bat le hasard, nous allons évaluer si la meilleure stratégie bat le hasard **sachant qu'elle a été sélectionnée parmi une foule de $K$ concurrentes**.

#### Étape A : La matrice des rendements centralisés
Soit un catalogue de $K$ stratégies testées. L'ordinateur calcule la performance quotidienne brute de chaque stratégie $k$ (notée $X_{k,t}$) et sa moyenne historique ($\bar{X}_k$).
Comme à la Partie 6, l'algorithme centralise **toutes** les stratégies pour les ramener sous l'Hypothèse Nulle $H_0$ (aucun talent) :

$$X^\dagger_{k,t} = X_{k,t} - \bar{X}_k \quad \text{pour } k = 1, 2, \dots, K$$

#### Étape B : Le Bootstrap Stationnaire Simultané
C'est le cœur de l'algorithme. Pour chaque réplication $m$ (allant de 1 à 10 000) :
1. L'ordinateur tire une unique suite d'indices temporels bootstrapés via notre loi géométrique (Partie 3).
2. Il applique **cette même suite exacte d'indices** à toutes les stratégies du catalogue en même temps.
3. Pour la simulation $m$, on obtient 1 000 moyennes simulées sous $H_0$ (une pour chaque stratégie).
4. L'ordinateur identifie la performance maximale obtenue par la chance parmi tout le catalogue lors de cette simulation $m$, notée $\max_{k} \bar{X}_k^{*(m)}$.

### 3. La P-Value du Reality Check et la Règle de Décision

Nous ne comparons plus le score réel du gérant à la distribution d'une seule stratégie chanceuse. Nous comparons le score réel de la meilleure stratégie du catalogue (noté $\bar{X}_{\text{best}}$) à la distribution des **meilleurs scores que le hasard a pu produire en simulant 1 000 stratégies d'un coup**.

#### La formule de la P-Value de White :
$$\text{P-value}_{\text{White}} = \frac{1}{M} \sum_{m=1}^M \mathbb{I}\left(\max_{k=1\dots K} \bar{X}_k^{*(m)} \ge \bar{X}_{\text{best}}\right)$$

#### La Règle de Décision :
Le seuil reste $\alpha = 0.05$. Si la $\text{P-value}_{\text{White}} < 0.05$, la stratégie surpasse le biais du Data Mining. Le gérant a un vrai talent.

---

### 4. Application Numérique et Justification Économétrique

Imaginons qu'un fonds d'investissement teste $K = 1\ 000$ algorithmes de trading sur $T = 1\ 000$ jours. Le meilleur algorithme trouvé (l'algorithme 412) affiche une performance moyenne de **$\bar{X}_{\text{best}} = +0.05\%$**.
Si on testait cet algorithme 412 isolément (comme Virginie à l'Expérience 11), sa P-value individuelle serait de **0.12 %**, ce qui déclencherait un feu vert immédiat.

Faisons tourner le *Reality Check* de White sur les 10 000 réplications :

*   **Simulation $m=1$ :** Le hasard simule les 1 000 stratégies. La stratégie 1 fait $-0.02\%$, la stratégie 2 fait $+0.01\%\dots$ et la stratégie 784 atteint par chance **$+0.06\%$**. Le maximum du hasard pour ce monde 1 est de $+0.06\%$. Comme $+0.06\% \ge +0.05\%$, la condition est remplie (le hasard a fait mieux que notre champion).
*   **Simulation $m=2$ :** Le hasard simule à nouveau. Le maximum décroché par la chance parmi les 1 000 candidats s'élève à **$+0.04\%$**. Comme $+0.04\% < +0.05\%$, la condition n'est pas remplie (le champion reste devant).

À la fin des 10 000 simulations simultanées, l'ordinateur constate que dans **2 350 mondes parallèles**, le simple fait de tester 1 000 stratégies en même temps a permis au hasard de faire émerger au moins un candidat affichant un score supérieur ou égal à $+0.05\%$.

#### Calcul de la P-Value :
$$\text{P-value}_{\text{White}} = \frac{2350}{10000} = \mathbf{0.2350} \quad (\text{soit } 23.50 \%)$$

---

### 5. Ce que l'on interprète (Le verdict ultime)

Le résultat est implacable :
$$\text{P-value}_{\text{White}} \ (\mathbf{0.2350}) \ \ge \ \alpha \ (\mathbf{0.05})$$

**L'Hypothèse Nulle $H_0$ ne peut pas être rejetée.** La stratégie 412 est invalidée et son déploiement est interdit.

#### 🎯 Justification physique du verdict :
Cet écart spectaculaire entre la P-value individuelle (0.12 %) et la P-value de White (23.50 %) met en lumière le piège du Data Mining. Pris isolément, un score de $+0.05\%$ ébahit. Mais lorsque vous cherchez ce score parmi 1 000 stratégies en même temps, la probabilité que la chance pure fabrique un tel résultat s'élève à 23.50 % (soit près d'une chance sur quatre).

L'algorithme 412 n'a aucun pouvoir prédictif : ce n'est qu'un mirage statistique, une illusion née de la sur-optimisation des données passées. Le *Reality Check* de White s'impose ainsi comme l'arme absolue en finance quantitative pour doucher les faux espoirs des chercheurs et protéger le capital des investisseurs contre le surapprentissage.

## Partie 8 : Perspectives opérationnelles et synthèse globale

L'implémentation industrielle du bootstrap en finance quantitative ne tolère aucune approximation. Cette dernière section dresse la synthèse des règles de décision opérationnelles et ouvre les perspectives sur les architectures de validation de troisième génération.

---

### 1. Synthèse des Règles de Décision pour l'Ingénierie Quantitative

Pour maximiser la robustesse des simulations de risques et la validation de stratégies, le protocole de sélection des algorithmes doit suivre un arbre de décision strict fondé sur les caractéristiques empiriques des données :

*   **Présence d'indépendance stricte (Bruit Blanc i.i.d.) ?**
    *   *Décision :* Bootstrap d'Efron classique (Tirage point par point avec remise).
*   **Présence de dépendance temporelle (Autocorrélation, GARCH) ?**
    *   *Sous-critère A : Structure linéaire et décroissance monotone stable ?*
        *   *Décision :* Algorithme de Politis & White (2004) pour extraire $p_{\text{optimal}}$.
    *   *Sous-critère B : Présence de fortes résurgences ou cycles asymétriques tardifs ?*
        *   *Décision :* Correction d'Andrew Patton (2009) avec noyau plat pour extraire $b_{\text{SB, opt}}$.

**Règle d'or de la validation :** Ne jamais évaluer une stratégie de trading de manière isolée si elle est issue d'un processus de recherche exploratoire. L'application systématique du **Reality Check de White** (ou de ses extensions) est une obligation fiduciaire pour immuniser les portefeuilles contre le surapprentissage.

---

### 2. Au-delà de White : Les Évolutions de Troisième Génération

Bien que le *Reality Check* de White (2000) ait révolutionné le contrôle des risques de sur-optimisation, la recherche économétrique contemporaine a mis en évidence certaines limites opérationnelles, ouvrant la voie à deux extensions majeures :

#### A. Le Test de Supériorité Prédictive de Hansen (SPA — 2005)
Le test de White est asymptotiquement conservateur. Si vous intégrez dans votre catalogue de $K$ stratégies un grand nombre de modèles très médiocres (qui accumulent de lourdes pertes), ces stratégies augmentent artificiellement la variance globale de la matrice. Placées au dénominateur, elles font gonfler de manière trompeuse la $\text{P-value}_{\text{White}}$, conduisant au rejet d'un algorithme potentiellement talentueux.
*   **Le correctif de Hansen :** Le test SPA (*Superior Predictive Ability*) applique une pondération conditionnelle qui neutralise l'influence des stratégies indiscutablement mauvaises. La P-value est réajustée uniquement sur la base des concurrentes viables, augmentant considérablement la puissance statistique du test.

#### B. La Borne Supérieure du Data Mining de Hsu, Kuan & Sin (2010)
Le Reality Check classique valide ou invalide un champion au niveau de confiance fixé (5 %). Il ne donne cependant aucune information sur la "vitesse d'altération" du catalogue de recherche.
*   **L'apport de Hsu et al. :** Cette approche formalise une courbe de pénalisation mathématique qui calcule combien de stratégies supplémentaires un chercheur peut tester avant qu'une P-value valide ne bascule définitivement dans la zone du mirage statistique. Cela permet aux directeurs de recherche de quantifier précisément leur "budget d'erreur informatique".

---

### 3. Conclusion Générale : La Stationnarité comme Impératif Statistique

L'évolution des techniques de rééchantillonnage démontre que la géométrie des simulations dicte la qualité de l'évaluation des risques. Les approches primitives à blocs fixes (MBB, CBB), bien que supérieures au bootstrap i.i.d., introduisaient des distorsions spatiales et temporelles fatales à la stationnarité du signal.

En libérant la taille des blocs par le biais de la loi géométrique, le **Bootstrap Stationnaire de Politis & Romano** a fourni un cadre mathématique rigoureux où le temps simulé préserve les propriétés intrinsèques des marchés réels. Combiné aux filtres correctifs de Patton et aux protocoles multi-tests de White, cet écosystème s'impose comme l'infrastructure fondamentale pour assainir l'ingénierie financière, séparer le bruit de la véritable information, et ancrer la gestion d'actifs dans une démarche scientifique incontestable.
