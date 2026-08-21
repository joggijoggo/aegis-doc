## Partie 4 : La sélection automatique du paramètre de coupure optimal ($p$) selon Politis & White (2004)

Maintenant que la validité de la loi géométrique est démontrée, l'enjeu statistique réside dans la détermination rigoureuse de la probabilité de coupure $p$, ou de manière équivalente, de la longueur moyenne des blocs $\mathbb{E}(L) = 1/p$.

Pour éliminer l’arbitraire d’un choix manuel, Dimitris Politis et Halbert White ont introduit en 2004 un algorithme permettant d'automatiser ce calcul. La méthode repose sur l'analyse de la mémoire de la série d'origine par le biais de ses autocorrélations afin de minimiser mathématiquement l'Erreur Quadratique Moyenne (MSE).

## Partie 4.1 : La Fondation – La Règle de Bartlett et le Seuil Statistique de Bruit

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






## Partie 4.2 : Détection du Point de Rupture ($k^*$) et Fenêtre Globale ($M$)

L'étape suivante consiste à repérer la fin de la mémoire active et à délimiter l'espace total où les corrélations seront sommées.

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

### 3. La Fenêtre de Mémoire Globale ($M$)
Avant de quantifier la puissance de la mémoire, l'algorithme doit délimiter l'espace total où les corrélations seront sommées.

#### Définition et construction mathématique :
Pour s'assurer qu'aucune queue de corrélation tardive ou résiduelle n'a été oubliée lors de la coupure automatique, Politis et White appliquent un multiplicateur de sécurité multiplicatif pour définir la fenêtre de calcul globale $M$ :

$$M = 2 \times k^*$$

#### 🎯 Justification physique de l'élargissement de $M$ :
Le choix de doubler la valeur de $k^*$ est une précaution de lissage. Même si les corrélations individuelles passent sous le seuil à partir du jour $k^*$, l'accumulation de plusieurs petites corrélations insignifiantes (par exemple quatre jours d'affilée à 0.04) peut représenter, une fois additionnée, une force d'inertie non négligeable. Pousser la fenêtre jusqu'à $2k^*$ permet de saturer l'examen statistique et d'englober la totalité des effets de traîne de la série temporelle.

#### 🔢 Application numérique de $M$ :
Avec notre point de rupture $k^* = 4$ validé à la section précédente, le calcul de la fenêtre globale donne :
$$M = 2 \times 4 = \mathbf{8 \text{ jours}}$$
L'ordinateur sait qu'il devra analyser et intégrer les données d'autocorrélations du marché jusqu'à un horizon de 8 jours en arrière.

## Partie 4.3 : Force du Signal, Variabilité et Synthèse de $p_{\text{optimal}}$

Cette ultime phase assemble la force de la mémoire, la dispersion de la série et le facteur d'échelle pour extraire la probabilité de coupure idéale.

### 4. La Force du Signal de Mémoire Cumulée ($G_{\text{PW}}$)
L'algorithme agrège désormais mathématiquement la structure de la mémoire passée pour calibrer l'intensité des dépendances.

#### Définition et construction mathématique :
Le paramètre $G_{\text{PW}}$ calcule la somme des autocorrélations contenues dans la fenêtre $M$, pondérées linéairement par leur propre décalage temporel $k$ :

$$G_{\text{PW}} = \sum_{k=-M}^{M} \vert k \vert \cdot \rho_k = 2 \times \sum_{k=1}^{M} k \cdot \rho_k$$

#### 🎯 Justification économétrique de la pondération par $k$ :
Pourquoi multiplier chaque autocorrélation par son décalage $k$ ? Parce qu'une dépendance qui persiste loin dans le temps (un grand $k$) témoigne d'une inertie boursière beaucoup plus lourde et complexe à répliquer qu'une dépendance qui s'éteint immédiatement après un jour. Si $\rho_8 = 0.10$, l'impact sur le système financier est plus profond que si $\rho_1 = 0.10$. Multiplier par $k$ permet de donner un poids géométrique dominant aux mémoires longues, ce qui influencera à la hausse la taille moyenne des blocs de simulation.

#### 🔢 Application numérique de $G_{\text{PW}}$ :
Calculons la force de la mémoire cumulée sur notre fenêtre de 8 jours (en considérant que les corrélations des jours 5 à 8 oscillent près de zéro et s'annulent) :
$$\sum_{k=1}^{8} k \cdot \rho_k = (1 \times 0.40) + (2 \times 0.25) + (3 \times 0.18) + (4 \times 0.08) + 0 + 0 + 0 + 0$$
$$\sum_{k=1}^{8} k \cdot \rho_k = 0.40 + 0.50 + 0.54 + 0.32 = 1.76$$

Le coefficient de structure de mémoire globale $G_{\text{PW}}$ s'établit donc à :
$$G_{\text{PW}} = 2 \times 1.76 = \mathbf{3.52}$$

---

### 5. L'Impact de la Variabilité Interne ($\sigma_Y^2$)
Le calcul du $p$ optimal fait intervenir une seconde force : la dispersion des données d'origine.

#### Définition et construction mathématique :
Au numérateur de la formule finale se trouve $\sigma_Y^2$, qui correspond à la variance brute de notre série de volatilité calculée à l'Expérience 1.

#### 🎯 Justification statistique du paramètre $\sigma_Y^2$ :
La variance brute mesure l'instabilité de fond de votre échantillon. Dans l'architecture asymptotique de minimisation de la MSE développée par Politis et White, plus la variance brute d'une série est élevée, plus le risque d'instabilité des simulations est grand (variance du bootstrap élevée). Pour stabiliser l'estimateur, la formule réagit en augmentant la valeur du numérateur, ce qui va tirer le paramètre $p$ vers le haut (donc raccourcir la taille moyenne des blocs) afin d'augmenter le nombre de blocs combinables et de faire baisser la variance globale du modèle simulé. Pour notre exemple, elle est évaluée à :
$$\sigma_Y^2 = \mathbf{1.50}$$

---

### 6. Le Facteur d'Échelle Temporelle ($T^{-1/3}$)
L'algorithme doit ajuster la sensibilité de son calcul à la quantité totale d'information historique disponible.

#### Définition et construction mathématique :
Le dernier paramètre multiplicatif de l'équation est le facteur d'échelle $T^{-1/3}$, où $T$ représente la taille totale de l'échantillon.

#### 🎯 Justification statistique de la puissance cubique :
Pourquoi appliquer une puissance de -1/3 ? C'est la vitesse de convergence mathématique optimale établie pour les processus dépendants par blocs lissés. Plus votre historique $T$ grandit, plus la valeur de $T^{-1/3}$ diminue. Cette décroissance asymptotique garantit que lorsque vous disposez de très grandes bases de données, le paramètre $p$ diminue de manière contrôlée, ce qui allonge la taille moyenne des blocs ($1/p$). Avec notre historique de $T = 1\ 000$ jours :
$$T^{-1/3} = 1000^{-1/3} = \mathbf{0.10}$$

---

### 7. La Synthèse Algébrique Finale pour extraire $p_{\text{optimal}}$
La dernière étape consiste à assembler l'ensemble de ces sous-composants numériques dans l'équation d'optimisation de Politis et White pour minimiser la MSE.

#### Définition et construction mathématique :
$$p_{\text{optimal}} = \left( \frac{\sigma_Y^2}{G_{\text{PW}}^2} \right)^{1/3} \times T^{-1/3}$$

#### 🔢 Application numérique d'assemblage final :
Injectons les valeurs chiffrées trouvées pas à pas au cours de notre protocole :
*   $\sigma_Y^2 = 1.50$
*   $G_{\text{PW}} = 3.52 \implies G_{\text{PW}}^2 = 3.52 \times 3.52 = 12.3904$
*   $T^{-1/3} = 0.10$

1. **Rapport de structure interne :** $\frac{1.50}{12.3904} \approx 0.12106$
2. **Extraction de la racine cubique :** $(0.12106)^{1/3} \approx 0.4947$
3. **Ajustement final par l'échelle :** $p_{\text{optimal}} = 0.4947 \times 0.10 \approx \mathbf{0.0495}$

*Verdict mathématique :* L'algorithme extrait une probabilité de coupure optimale de **0.0495** (4.95 % de chances de briser le bloc). La longueur moyenne idéale de nos blocs variables sera de : $\mathbb{E}(L) = \frac{1}{0.0495} \approx \mathbf{20.2 \text{ jours}}$. Le modèle prouve que des blocs moyens de 20 jours minimisent la MSE.
