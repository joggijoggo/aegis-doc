# 📊 Laboratoire Économétrique : Rapport Récapitulatif Global
## Modélisation, Calibration et Validation des Processus Dépendants par Bootstraps Avancés

---

## 🗒️ PARTIE 1 : RÉCAPITULATIF DES ENJEUX THÉORIQUES ET FONDATIONS

L'analyse des séries temporelles financières (rendements, volatilité, spreads) se heurte à une caractéristique universelle : la **dépendance temporelle**. Contrairement aux postulats des modèles classiques de marche aléatoire, les observations successives ne sont pas indépendantes. Elles présentent des structures d'autocorrélation et des phénomènes de grappes de volatilité (*volatility clustering*).

Si l'on applique le bootstrap d'Efron classique (tirage point par point avec remise) sur ces séries, le fil chronologique est rompu, ce qui détruit la mémoire du processus et conduit à une sous-estimation systématique des risques et de la variance de long terme.

### 1. La taxonomie des architectures de rééchantillonnage

Pour préserver la structure dynamique d'une série de taille $T$, l'économétrie a développé le concept de **rééchantillonnage par blocs contigus**. Trois géométries majeures fondées sur des blocs de taille fixe ont jalonné cette évolution avant l'introduction des approches à taille aléatoire :

*   **Moving Block Bootstrap (MBB) - Blocs Mouvants (Künsch, 1989) :** Une fenêtre glissante de taille $b$ déterministe se déplace d'un jour à la fois. L'algorithme s'arrête strictement à la barrière physique de la fin de l'échantillon $t_T$.
*   **Circular Block Bootstrap (CBB) - Blocs Circulaires (Politis & Romano, 1992) :** Les indices temporels sont calculés modulo $T$. Le temps est enroulé en anneau afin d'éliminer mathématiquement les barrières du début et de la fin.
*   **Stationary Bootstrap (SB) - Blocs Stationnaires (Politis & Romano, 1994) :** Structure circulaire combinée à une longueur de bloc $L$ aléatoire régie par une loi géométrique de paramètre $p$. Ce modèle introduit la propriété de perte de mémoire numérique pour préserver la stationnarité.

### 2. Synthèse Comparative des Propriétés Géométriques

| Propriété Spatio-Temporelle | Moving Block (MBB) | Circular Block (CBB) | Stationary Bootstrap (SB) |
| :--- | :--- | :--- | :--- |
| **Longueur du bloc ($b$)** | Fixe et déterministe | Fixe et déterministe | Aléatoire : Loi Géométrique ($p$) |
| **Traitement des Bordures** | Arrêt linéaire à $t_T$ | Circularité ($t_T \to t_1$) | Circularité ($t_T \to t_1$) |
| **Équité Spatiale** | **Inégale** : sous-échantillonne les extrémités | **Uniforme** : chaque point pèse le même poids | **Uniforme** : chaque point possède la même probabilité |
| **Stationnarité de la série** | **Détruite** (Effets de bord asymétriques) | **Détruite** (Points de rupture cycliques) | **Parfaitement préservée** par construction mathématique |

---

## 🔬 PARTIE 2 : COMPENDIUM COMPLET DES EXPÉRIENCES DE LABORATOIRE (1 À 12)

### 📈 Expérience 1 : Cartographie de l'autocorrélation linéaire de Pearson
*   **Objectif :** Quantifier et valider statistiquement l'existence d'une mémoire temporelle au sein d'une série financière réelle.
*   **Méthodologie :** Extraction de la série des carrés des rendements quotidiens ($R_t^2$) du S&P 500 sur $T = 1\ 250$ jours. Calcul du coefficient d'autocorrélation d'ordre 1 ($\rho_1$) via le ratio de la covariance décalée sur la variance globale.
*   **Application Numérique & Résultats :** L'ordinateur calcule $\rho_1 = \mathbf{0.22}$.
*   **Interprétation :** Le résultat $0.22 \neq 0$ prouve la présence d'une persistance de court terme. Un choc de volatilité aujourd'hui a 22 % de chances de se propager directement sur la journée de demain.

### 📊 Expérience 2 : L'effondrement de la mémoire sous le Bootstrap d'Efron
*   **Objectif :** Démontrer empiriquement l'incapacité du bootstrap point par point à reproduire les structures de dépendance.
*   **Méthodologie :** Rééchantillonnage de la série précédente par tirages uniformes i.i.d. point par point ($M = 10\ 000$ réplications). Calcul de la moyenne des autocorrélations simulées ($\bar{\rho}_1^*$).
*   **Application Numérique & Résultats :** La mémoire s'effondre à $\bar{\rho}_1^* = \mathbf{0.0002}$.
*   **Interprétation :** Le bootstrap d'Efron détruit instantanément la dynamique temporelle. Il génère un monde alternatif sans mémoire boursière, biaisant toute évaluation ultérieure de la persistance des risques.

### 📐 Expérience 3 : Mise en évidence de la distorsion spatiale du MBB (Le piège du trapèze)
*   **Objectif :** Quantifier l'effet de bord asymétrique généré par la rigidité linéaire du découpage en blocs fixes du MBB.
*   **Méthodologie :** Suivi d'un vecteur d'indices séquentiels $[1, 2 \dots 1\ 000]$ soumis à $M = 10\ 000$ simulations MBB avec un bloc fixe $b = 50$ jours. Calcul de la probabilité empirique d'apparition de chaque jour $P_t = F_t / 10\ 000$. Target uniforme idéale : $P_t = 1.00$.
*   **Application Numérique & Résultats :** Les points centraux affichent $P_t \approx \mathbf{1.00}$. Les extrémités s'effondrent de manière linéaire : $P_1 = P_{1000} \approx \mathbf{0.02}$.
*   **Interprétation :** L'indice $1$ n'appartient qu'au bloc $B_1$, tandis que l'indice $500$ appartient à 50 blocs glissants différents. Le MBB applique une distorsion géométrique en trapèze qui pénalise les extrémités de l'historique, brisant la stationnarité.

### ⛓️ Expérience 4 : Preuve de la périodicité artificielle du CBB (Le piège des dents de scie)
*   **Objectif :** Démontrer que la circularité du temps combinée à des blocs de taille fixe engendre des points de rupture prévisibles non stationnaires.
*   **Méthodologie :** Utilisation d'un indicateur binaire de continuité temporelle $C_s$ ($C_s=1$ si les jours se suivent dans le calendrier réel, $0$ s'il y a un saut de bloc). Simulation de $10\ 000$ réplications CBB avec $T = 1\ 000$ et $b = 10$ jours fixes.
*   **Application Numérique & Résultats :** Pour les positions $s = 1 \dots 9$, la probabilité de continuité vaut $P_s = \mathbf{1.00}$ (100 %). Aux charnières exactes $s = \{10, 20, 30 \dots 990\}$, la probabilité s'effondre à $P_s \approx \mathbf{0.001}$.
*   **Interprétation :** Le CBB redistribue équitablement le poids des points grâce à l'enroulement circulaire, mais introduit un cycle artificiel de rupture déterministe tous les $b$ jours. La structure de covariance dépend du temps boursier, ce qui est incompatible avec le postulat de stationnarité.

### 📉 Expérience 5 : Quantification du dilemme Biais-Variance sur blocs fixes
*   **Objectif :** Mesurer numériquement l'impact du choix de la taille du bloc déterministe sur la qualité globale de l'estimateur (MSE).
*   **Méthodologie :** Test de trois configurations CBB sur la série S&P 500 ($\rho_1 = 0.22$, $\sigma_Y^2 = 1.50$) : Blocs courts ($b=2$), intermédiaires ($b=20$), longs ($b=400$). Calcul de la $\text{MSE} = \text{Biais}^2 + \text{Variance}_{\rho_1^*}$.
*   **Application Numérique & Résultats :**
    *   *Configuration A ($b=2$) :* $\bar{\rho}_1^* = 0.01$ (Biais max), $\sigma^2_{\rho_1^*} = 0.002 \implies \text{MSE} = \mathbf{0.0461}$
    *   *Configuration B ($b=20$) :* $\bar{\rho}_1^* = 0.20$ (Biais faible), $\sigma^2_{\rho_1^*} = 0.005 \implies \text{MSE} = \mathbf{0.0054}$
    *   *Configuration C ($b=400$) :* $\bar{\rho}_1^* = 0.22$ (Biais nul),  $\sigma^2_{\rho_1^*} = 0.038 \implies \text{MSE} = \mathbf{0.0380}$
*   **Interprétation :** Des blocs trop courts détruisent la mémoire (biais élevé). Des blocs trop longs figent l'échantillonnage en quelques morceaux massifs copier-coller, ce qui fait exploser l'instabilité de l'estimateur d'une simulation à l'autre (variance élevée). La configuration intermédiaire ($b=20$) minimise la MSE.

### 🌟 Expérience 6 : Validation de l'équité spatiale du Bootstrap Stationnaire
*   **Objectif :** Prouver empiriquement l'architecture de Politis et Romano (SB) neutralise définitivement le piège du trapèze du MBB.
*   **Méthodologie :** Répétition stricte du protocole de l'Expérience 3 sur $T = 1\ 000$ jours avec un Bootstrap Stationnaire configuré sur une longueur moyenne équivalente $\mathbb{E}(L) = 50$ jours, soit une probabilité d'arrêt Bernoulli $p = 1/50 = 0.02$.
*   **Application Numérique & Résultats :** La courbe des probabilités d'échantillonnage $P_t$ dessine une ligne horizontale parfaitement plate. Pour tout $t \in [1, 1\ 000]$, $P_t = \mathbf{1.00} \ (\pm 0.01)$.
*   **Interprétation :** La combinaison du wrapping circulaire et de la flexibilité de la loi géométrique redonne exactement les mêmes chances géométriques à chaque observation. Le critère d'homogénéité spatiale est validé.

### ⚡ Expérience 7 : Validation du lissage de la continuité temporelle par le SB
*   **Objectif :** Prouver que la loi géométrique gomme le motif cyclique en dents de scie du CBB.
*   **Méthodologie :** Répétition stricte du protocole de l'Expérience 4 sur $T = 1\ 000$ jours avec un Bootstrap Stationnaire réglé sur $\mathbb{E}(L) = 10$ jours, soit une probabilité d'arrêt constante $p = 0.10$. Suivi de l'indicateur de continuité $C_s$.
*   **Application Numérique & Résultats :** Les oscillations déterministes disparaissent. Le graphique de $P_s$ affiche une ligne horizontale stable à la hauteur exacte de la probabilité théorique de prolongation : $P_s = 1 - p = \mathbf{0.90} \ (\pm 0.005)$.
*   **Interprétation :** Grâce à la propriété de perte de mémoire de la loi géométrique, le risque de subir une cassure est homogène et identique à chaque pas de temps. La périodicité artificielle est éliminée, garantissant la stationnarité temporelle stricte de la série rééchantillonnée.

### ⚙️ Expérience 8 : Sensibilité de l'algorithme Politis-White aux régimes de marché
*   **Objectif :** Vérifier la capacité de l'algorithme automatique de Politis-White (2004) à adapter le paramètre $p$ à la nature physique des données.
*   **Méthodologie :** Simulation de deux marchés de $T = 1\ 000$ jours face à un filtre de bruit $\text{Seuil}_{\text{PW}} = 0.1662$. Un Marché A de crise (persistance AR(1), $\phi=0.80$, $\sigma_A^2 = 3.20$) et un Marché B efficient (bruit blanc i.i.d., $\sigma_B^2 = 0.20$).
*   **Application Numérique & Résultats :**
    *   *Marché A (Crise) :* Le scan détecte $k^* = 7 \implies M = 14 \implies G_{\text{PW}} = 15.78 \implies p_{\text{optimal}} = \mathbf{0.0234} \implies \mathbb{E}(L) = \mathbf{42.7 \text{ jours}}$.
    *   *Marché B (Calme) :* Le scan détecte $k^* = 1 \implies M = 2 \implies G_{\text{PW}} = 0.08 \implies p_{\text{optimal}} = \mathbf{0.3150} \implies \mathbb{E}(L) = \mathbf{3.17 \text{ jours}}$.
*   **Interprétation :** En période de stress, l'algorithme détecte une mémoire longue et ordonne de déployer de très grands blocs (43 jours) pour capturer l'inertie du risque. En période calme, il constate l'absence de signal, réduit les blocs à 3 jours pour maximiser le mélange des données et optimiser la diversité.

### 🔍 Expérience 9 : Impact du facteur d'échelle temporel asymptotique sur le SB
*   **Objectif :** Quantifier comment l'algorithme ajuste sa sévérité et sa calibration lorsque le volume d'historique ($T$) augmente.
*   **Méthodologie :** Analyse d'une structure de marché fixe ($\rho_1=0.40, \rho_2=0.25, \rho_3=0.18, \rho_4=0.08$, $\sigma_Y^2=1.50$) sous deux échelles de profondeur : $T = 500$ jours et $T = 8\ 000$ jours.
*   **Application Numérique & Résultats :**
    *   *Échelle Courte ($T=500$) :* Seuil de bruit large = $0.2229 \implies k^*=3 \implies M=6 \implies p_{\text{optimal}} = \mathbf{0.0975} \implies \mathbb{E}(L) = \mathbf{10.25 \text{ jours}}$.
    *   *Échelle Longue ($T=8\ 000$) :* Seuil resserré = $0.0670 \implies k^*=5 \implies M=10 \implies p_{\text{optimal}} = \mathbf{0.0247} \implies \mathbb{E}(L) = \mathbf{40.5 \text{ jours}}$.
*   **Interprétation :** Disposant d'un catalogue massif de 8 000 points, l'ordinateur réduit la marge d'erreur tolérée, détecte des traces de mémoire fine cachées à basse fréquence (les jours 3 et 4) et déploie de très grands blocs (40 jours) sans risque d'étouffer la diversité statistique.

### 📐 Expérience 10 : Preuve de l'efficacité du correctif à Noyau Plat de Patton (2009)
*   **Objectif :** Mesurer la réduction de la MSE obtenue par l'approche de Patton (2009) face à une série présentant une résurgence de mémoire.
*   **Méthodologie :** Comparaison du simulateur PW ($b = 3.3$ jours) et du simulateur Patton ($b = 29.2$ jours) sur une série ($T = 1\ 000, \sigma_Y^2 = 1.50$) affichant une résurgence tardive au 5ème jour ($\rho_5 = 0.19$). Mesure de la $\text{MSE}$ sur l'autocorrélation simulée d'ordre 5.
*   **Application Numérique & Résultats :**
    *   *Simulateur Politis-White :* $\bar{\rho}_{5}^* = 0.021 \implies \text{Biais}^2 = 0.02856$, $\text{Variance} = 0.0011 \implies \text{MSE}_{\text{PW}} = \mathbf{0.02966}$.
    *   *Simulateur Patton :* $\bar{\rho}_{5}^* = 0.178 \implies \text{Biais}^2 = 0.00014$, $\text{Variance} = 0.0048 \implies \text{MSE}_{\text{Patton}} = \mathbf{0.00494}$.
*   **Interprétation :** Le modèle de Patton réduit l'erreur totale d'un **facteur 6**. En refusant l'arrêt prématuré au jour 4 grâce au lissage du noyau à toit plat, Patton préserve la mémoire du jour 5. Le gain massif sur le biais écrase largement la légère hausse de variance.

### 🎯 Expérience 11 : Simulation empirique du Test d'Hypothèse individuel
*   **Objectif :** Valider statistiquement la performance isolée d'un gérant de fonds (Virginie) via le calcul de sa P-value bootstrapée.
*   **Méthodologie :** Modélisation de la stratégie de Virginie sur $T = 1\ 000$ jours avec un rendement moyen de $\bar{X} = +0.05\%$. Centralisation algébrique de la série ($X^\dagger_t = X_t - 0.05\%$) pour créer le monde de l'Hypothèse Nulle $H_0$ (moyenne stricte = 0.00%). Exécution de $10\ 000$ simulations stationnaires calibrées par Patton ($p=0.0342$).
*   **Application Numérique & Résultats :** Le hasard pur centralisé parvient à égaler ou dépasser le score de $+0.05\%$ dans seulement 12 simulations sur 10 000. L'ordinateur extrait : $\text{P-value} = 12/10000 = \mathbf{0.0012}$ (0.12 %).
*   **Interprétation :** La P-value (0.12 %) étant inférieure au seuil alpha de 5 %, l'Hypothèse Nulle est rejetée. Il n'y a qu'une chance sur 833 pour que le hasard imite Virginie. Son talent de gestionnaire est validé.

### ⛓️ Expérience 12 : Simulation du Reality Check de White et preuve du biais de Data Mining
*   **Objectif :** Démontrer l'illusion de performance générée par la sur-optimisation de masse (tests multiples).
*   **Méthodologie :** Création d'une matrice de $K = 500$ stratégies algorithmiques indépendantes de pur bruit blanc (aucun talent, $\mu_k = 0.00\%$) sur $T = 1\ 000$ jours. Repérage du champion de la chance ($\bar{X}_{\text{best}} = +0.045\%$). Application du bootstrap simultané de White ($M = 10\ 000$) sur la matrice centralisée pour extraire la probabilité que le hasard génère un tel score parmi 500 candidats.
*   **Application Numérique & Résultats :**
    *   *P-value Individuelle (naïve) :* Évaluée isolément, la stratégie affiche une $\text{P-value} = \mathbf{0.0080}$ (0.80 % $\to$ Rejet erroné de $H_0$).
    *   *P-value Globale de White :* Le maximum du hasard simultané surpasse $+0.045\%$ dans 3 410 mondes parallèles $\implies \text{P-value}_{\text{White}} = \mathbf{0.3410}$ (34.10 % $\to$ Acceptation stricte de $H_0$).
*   **Interprétation :** Pris isolément, le score semble exceptionnel. Mais lorsque l'on prend en compte le fait que l'ordinateur a donné 500 chances au hasard, la probabilité de voir émerger un faux champion s'élève à 34.10 % (plus d'une chance sur trois). Le Reality Check neutralise l'illusion du surapprentissage.

---

## 📈 PARTIE 3 : SYNTHÈSE MATHÉMATIQUE DES ÉQUATIONS ET DU PIPELINE LOGIQUE

Pour calibrer et exécuter ce protocole de pointe, l'ingénieur quantitatif doit assembler les équations validées selon l'enchaînement structurel décrit ci-dessous.

### 🌐 Étape 1 : Le Filtrage Initial du Bruit (Règle de Bartlett)
L'algorithme détermine la ligne de tolérance critique pour séparer les vraies corrélations du hasard.

$$\text{Seuil}_{\text{Bartlett}} = \pm \frac{1.96}{\sqrt{T}} \quad \xrightarrow{\text{Ajustement Asymptotique de Politis-White}} \quad \text{Seuil}_{\text{PW}} = \pm 2 \times \sqrt{\frac{\ln(T)}{T}}$$

### 🛠️ Étape 2 : Détection du Point de Rupture de la Mémoire ($k^*$)
Le scan identifie le premier décalage temporel où la corrélation s'écrase dans la zone grise du bruit :

$$k^* = \min \{ k \ge 1 \mid \vert \rho_k \vert < \text{Seuil}_{\text{PW}} \}$$

### 📏 Étape 3 : Délimitation de la Fenêtre Utile et Force du Signal ($G_{\text{PW}}$)
L'algorithme double l'horizon pour englober les effets de traîne et calcule la somme pondérée par le décalage temporel $k$ :

$$M = 2 \times k^* \quad \implies \quad G_{\text{PW}} = 2 \times \sum_{k=1}^{M} k \cdot \rho_k$$

### ⚙️ Étape 4 : Extraction Asymptotique de la Longueur du Bloc ($b_{\text{opt}}$)
Mise en concurrence de la variance de fond de la série ($\sigma_Y^2$) face à la force géométrique de sa mémoire ($G_{\text{PW}}^2$) :

$$b_{\text{PW, opt}} = \left( \frac{\sigma_Y^2}{G_{\text{PW}}^2} \right)^{1/3} \times T^{1/3} \quad \implies \quad p_{\text{optimal}} = \frac{1}{b_{\text{PW, opt}}}$$

---

### 🛡️ Étape 5 : L'Architecture de Correction d'Andrew Patton (2009)
En présence de structures complexes de résurgence, le pipeline intègre le **Noyau à Toit Plat** ($\lambda(k)$) and la **Variance de Long Terme** ($D_{\text{SB}}$) pour redéfinir la longueur :

$$\lambda(k) = \begin{cases} 1 & \text{si } 0 \le \frac{k}{M} < 0.5 \\ 2 \left(1 - \frac{k}{M}\right) & \text{si } 0.5 \le \frac{k}{M} \le 1 \\ 0 & \text{si } \frac{k}{M} > 1 \end{cases} \quad \implies \quad G_{\text{Patton}} = 2 \times \sum_{k=1}^{M} k \cdot \lambda(k) \cdot \rho_k$$

$$D_{\text{SB}} = 4 \times \left( \rho_0 \cdot \sigma_Y^2 + 2 \times \sum_{k=1}^{M} \lambda(k) \cdot \gamma_k \right) \quad \implies \quad b_{\text{SB, opt}} = \left( \frac{2 \cdot G_{\text{Patton}}^2}{D_{\text{SB}}} \right)^{1/3} \times T^{1/3}$$

---

### 🧪 Étape 6 : Le Pipeline de Test d'Hypothèse de Masse (Reality Check de White)
Pour valider un catalogue de $K$ stratégies face au surapprentissage, les données sont centralisées, parallélisées et soumises au verdict de la P-value cumulée :

$$\text{Matrice sous } H_0: \ X^\dagger_{k,t} = X_{k,t} - \bar{X}_k \quad \forall k \in [1, K]$$

$$\text{Exécution Asymptotique Simultanée} \implies \text{Extraction du maximum du Hasard} : \ \max_{k=1\dots K} \bar{X}_k^{*(m)}$$

$$\text{Calcul Final du Pouvoir de Décision} \implies \text{P-value}_{\text{White}} = \frac{1}{M} \sum_{m=1}^M \mathbb{I}\left(\max_{k=1\dots K} \bar{X}_k^{*(m)} \ge \bar{X}_{\text{best}}\right)$$

$$\text{Règle de décision : } \begin{cases} \text{P-value}_{\text{White}} < 0.05 \implies \text{Rejet de } H_0 \implies \mathbf{\text{Stratégie Validée (Vrai Talent)}} \\ \text{P-value}_{\text{White}} \ge 0.05 \implies \text{Acceptation de } H_0 \implies \mathbf{\text{Stratégie Rejetée (Mirage Statistique)}} \end{cases}$$
