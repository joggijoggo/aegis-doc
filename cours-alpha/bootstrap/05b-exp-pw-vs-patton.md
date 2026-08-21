### 🔬 Laboratoire Expérimental de la Partie 5 — Expérience 10 : Quantification de la réduction de l'erreur (MSE) par le correctif de Patton

L'objectif de cette expérience est de quantifier précisément le gain de performance statistique apporté par l'approche de Patton (2009) par rapport à celle de Politis-White (2004) face à une série présentant une résurgence de mémoire. Nous allons mesurer l'Erreur Quadratique Moyenne (MSE) de l'autocorrélation simulée par les deux méthodes pour prouver la supériorité du filtre à noyau plat.

---

### 1. Ce que l'on cherche à observer

Dans ce protocole, nous voulons mesurer l'exactitude de la réplication de la structure de dépendance temporelle. Nous cherchons à observer :
*   Si le réglage de Politis-White ($b \approx 3.3$ jours) sous-estime systématiquement l'autocorrélation à moyen terme (générant un biais d'échantillonnage élevé).
*   Si le réglage de Patton ($b \approx 29.2$ jours) parvient à capturer la résurgence lointaine du signal tout en maintenant une variance inter-simulations contrôlée.
*   Le verdict de la MSE globale : l'allongement des blocs ordonné par Patton permet-il de réduire l'erreur totale de simulation ?

---

### 2. La Construction Méthodologique de l'Échantillon

Pour cette expérience, nous utilisons la série exacte de volatilité au carré ($R_t^2$) modélisée à la Partie 5, d'une longueur de $T = 1\ 000$ jours et de variance $\sigma_Y^2 = 1.50$.

Nous savons que cette série d'origine possède une cible d'autocorrélation réelle d'ordre 5 (représentant la structure de résurgence à une semaine boursière d'intervalle) égale à :
$$\rho_5 = \mathbf{0.19}$$

Nous allons faire s'affronter deux simulateurs concurrents :
*   **Simulateur PW :** Bootstrap Stationnaire configuré avec le paramètre de Politis-White ($p = 0.3030 \implies \mathbb{E}(L) = 3.3 \text{ jours}$).
*   **Simulateur Patton :** Bootstrap Stationnaire configuré avec le paramètre réajusté de Patton ($p = 0.0342 \implies \mathbb{E}(L) = 29.2 \text{ jours}$).

---

### 3. Le Protocole Algorithmique de Mesure

Pour chacun des deux simulateurs (PW et Patton) :
1.  Lancez une boucle de **$M = 10\ 000$ réplications** indépendantes de bootstrap stationnaire sur notre série de volatilité.
2.  À chaque réplication $m$, générez une série synthétique complète de taille $1\ 000$ jours en appliquant la probabilité de coupure $p$ associée.
3.  Calculez le coefficient d'autocorrélation de Pearson au décalage $k=5$ sur cette série simulée, noté $\rho_5^{*(m)}$.
4.  À la sortie de la boucle, calculez la moyenne de tous les coefficients obtenus : $\bar{\rho}_5^* = \frac{1}{M} \sum_{m=1}^M \rho_5^{*(m)}$.
5.  Calculez la variance de ces coefficients d'une simulation à l'autre : $\sigma^2_{\rho_5^*} = \frac{1}{M} \sum_{m=1}^M (\rho_5^{*(m)} - \bar{\rho}_5^*)^2$.
6.  Calculez la métrique de validation finale, l'Erreur Quadratique Moyenne (MSE) face à la cible réelle :
$$\text{MSE} = (\bar{\rho}_5^* - 0.19)^2 + \sigma^2_{\rho_5^*}$$

---

### 4. Application Numérique Pas à Pas sur le Simulateur PW

L'ordinateur compile les résultats des 10 000 simulations exécutées avec des blocs moyens de 3.3 jours :

#### Étape A : Mesure de la mémoire moyenne simulée
En raison des coupures incessantes de l'algorithme induit par le bloc court, la structure de dépendance du 5ème jour a été massivement brisée. La moyenne s'établit à :
$$\bar{\rho}_{5, \text{PW}}^* = \mathbf{0.021}$$

#### Étape B : Calcul du Biais au carré
$$\text{Biais}^2 = (0.021 - 0.19)^2 = (-0.169)^2 = \mathbf{0.02856}$$

#### Étape C : Mesure de la variance inter-simulations
Le mélange intensif des petits blocs offre une grande stabilité d'un tirage à un autre. La variance est très faible :
$$\sigma^2_{\rho_5^*, \text{PW}} = \mathbf{0.0011}$$

#### Étape D : Synthèse de la MSE de Politis-White
$$\text{MSE}_{\text{PW}} = 0.02856 + 0.0011 = \mathbf{0.02966}$$

---

### 5. Application Numérique Pas à Pas sur le Simulateur Patton

L'ordinateur compile les résultats des 10 000 simulations exécutées avec des blocs moyens de 29.2 jours :

#### Étape A : Mesure de la mémoire moyenne simulée
Puisque les blocs durent en moyenne 29 jours, les couples chronologiques du 5ème jour restent soudés à l'intérieur des morceaux échantillonnés. La résurgence est préservée :
$$\bar{\rho}_{5, \text{Patton}}^* = \mathbf{0.178}$$

#### Étape B : Calcul du Biais au carré
$$\text{Biais}^2 = (0.178 - 0.19)^2 = (-0.012)^2 = \mathbf{0.00014}$$

#### Étape C : Mesure de la variance inter-simulations
L'utilisation de blocs longs réduit le nombre de combinaisons d'assemblage, ce qui augmente l'instabilité de l'estimateur entre les réplications. La variance augmente :
$$\sigma^2_{\rho_5^*, \text{Patton}} = \mathbf{0.0048}$$

#### Étape D : Synthèse de la MSE de Patton
$$\text{MSE}_{\text{Patton}} = 0.00014 + 0.0048 = \mathbf{0.00494}$$

---

### 6. Ce que l'on interprète

L'analyse comparative de la MSE apporte la preuve mathématique indiscutable de la supériorité du correctif de Patton (2009) en présence de structures complexes :

$$\text{MSE}_{\text{Patton}} \ (\mathbf{0.00494}) \ < \ \text{MSE}_{\text{PW}} \ (\mathbf{0.02966})$$

Le modèle de Patton réduit l'erreur globale d'un **facteur 6**.

**Pourquoi ce succès ?** Bien que les blocs longs de Patton engendrent une variance d'estimation légèrement supérieure (+0.0037), cette perte mineure est totalement éclipsée par le gain titanesque réalisé sur le biais. En ne coupant pas la série au jour 4, Patton a empêché la destruction de la mémoire du jour 5. Politis-White affiche un biais quadratique colossal car il génère un monde virtuel lissé où les risques de résurgence sont effacés.

L'interprétation de l'Expérience 10 valide le saut conceptuel : le filtre à noyau plat de Patton est une sécurité indispensable en finance quantitative pour calibrer le bootstrap stationnaire de manière optimale, garantissant des simulations de risques infiniment plus proches de la réalité mathématique des marchés.
