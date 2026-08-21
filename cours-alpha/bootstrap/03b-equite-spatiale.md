### 🔬 Laboratoire Expérimental de la Partie 3 — Expérience 6 : Validation numérique de l'équité spatiale du Bootstrap Stationnaire

L'objectif de cette expérience est de prouver empiriquement que le Bootstrap Stationnaire (SB) de Politis et Romano élimine définitivement la distorsion spatiale des bordures (le piège du trapèze) mise en évidence lors de l'Expérience 3 avec le modèle MBB. Nous allons démontrer que chaque jour de l'historique possède rigoureusement la même probabilité d'échantillonnage, validant ainsi la stationnarité géométrique du modèle.

---

### 1. La Construction Méthodologique de l'Expérience

Pour assurer une comparaison directe et incontestable avec l'Expérience 3, nous conservons un protocole strictement identique en travaillant exclusivement sur un **vecteur d'indices de position temporelle**.

#### Étape A : Création du vecteur de suivi
Créez la même série temporelle factice représentée par un vecteur d'indices séquentiels allant de $1$ à $T = 1\ 000$ jours :

$$I = [1, \ 2, \ 3, \ \dots, \ 1\ 000]$$

#### Étape B : Paramétrage du Bootstrap Stationnaire (SB)
Lors de l'Expérience 3, nous avaient fixé une longueur de bloc rigide de $b = 50$ jours. Pour obtenir une base de comparaison équitable, nous devons configurer notre loi géométrique pour qu'elle génère des blocs d'une **longueur moyenne équivalente**.

La probabilité de coupure $p$ (le paramètre de Bernoulli) se calcule donc par inversion de l'espérance :

$$p = \frac{1}{\mathbb{E}(L)} = \frac{1}{50} = \mathbf{0.02}$$

*   **Motivation du paramètre :** À chaque pas de temps, l'ordinateur aura 2 % de chances de briser le bloc actuel et 98 % de chances de le prolonger en copiant le jour chronologique suivant.

---

### 2. Le Cœur de la Mesure : Le Compteur d'Échantillonnage Uniformisé

La métrique retenue reste le **compteur empirique d'échantillonnage**, noté $F_t$, qui comptabilise le nombre total de fois où chaque indice temporel est sélectionné au cours de l'ensemble des simulations.

#### Le Protocole Algorithmique :
1.  Initialisez un vecteur de compteurs de taille $T=1\ 000$ rempli de zéros : $F = [0_1, \ 0_2, \ \dots, \ 0_{1\ 000}]$.
2.  Lancez une boucle de simulation de **$M = 10\ 000$ réplications** de bootstrap stationnaire.
3.  À chaque réplication, l'ordinateur construit une nouvelle série de taille $1\ 000$ en appliquant l'algorithme détaillé à la Partie 3 (mélange de tirages uniformes et de tirages de Bernoulli avec $p=0.02$).
4.  À chaque fois qu'un indice temporel réel est copié dans la simulation, la case correspondante dans le vecteur de compteurs globaux $F$ est incrémentée de $+1$.

---

### 3. Le Choix de la Métrique de Validation : L'Indice d'Équité Spatiale

Pour valider le succès du lissage géométrique, nous calculons à la fin des simulations la probabilité empirique d'échantillonnage de chaque point $t$, définie par :

$$P_t = \frac{F_t}{M} = \frac{F_t}{10\ 000}$$

#### Règle de décision :
*   **Si la distorsion persiste (Échec) :** La courbe de $P_t$ doit reproduire l'affaissement aux extrémités observé avec le MBB (chute vers $0.20$ pour $t=1$ et $t=1\ 000$).
*   **Si l'équité spatiale est atteinte (Succès) :** Chaque observation ayant la même chance d'être tirée, la probabilité empirique doit être mathématiquement stable et égale à la performance moyenne cumulée par simulation ($1\ 000$ points générés pour $1\ 000$ points d'historique). La cible idéale est une ligne parfaitement plate à la hauteur de :

$$P_t = \mathbf{1.00}$$

---

### 4. Application Numérique Théorique

Pour comprendre pourquoi la loi géométrique rétablit l'équilibre aux bordures, analysons le calcul de probabilité d'apparition du premier jour ($t=1$) au tout premier emplacement de notre simulation.

*   **Dans le modèle MBB (Expérience 3) :** L'indice $1$ ne pouvait apparaître que si l'ordinateur sélectionnait le bloc $B_1$. La probabilité était bridée à $\frac{1}{951} \approx 0.00105$.
*   **Dans le modèle Stationnaire (SB) :** L'indice $1$ peut apparaître de deux manières indépendantes :
    1.  Il est pioché directement comme point de départ lors du tirage uniforme d'initialisation : Probabilité = $\frac{1}{1000} = 0.001$.
    2.  L'ordinateur était en train de lire le dernier jour de la série ($t=1\ 000$), le test de Bernoulli a décidé de prolonger le bloc (Probabilité = $0.98$), et la règle du wrapping circulaire a ramené le pointeur au début : Probabilité associée = $\mathbb{P}(Y^*_{t-1} = Y_{1000}) \times 0.98$.

En additionnant toutes les trajectoires probabilistes circulaires sur les 10 000 simulations, la somme des chances géométriques de chaque point finit par s'équilibrer de manière rigoureuse.

---

### 5. Ce que vous allez observer

En traçant la courbe des probabilités empiriques $P_t$ de la position $t=1$ à $t=1\ 000$ à la sortie de vos simulations de Bootstrap Stationnaire, vous allez observer un changement radical par rapport au trapèze du MBB :
*   Le trapèze a totalement disparu. La courbe ne subit plus aucun affaissement ou déformation aux extrémités.
*   Le graphique dessine une **ligne horizontale parfaitement plate** qui traverse tout l'axe du temps de part en part.
*   Chaque case du vecteur affiche une valeur unitaire oscillant de manière infime autour de la cible théorique : $P_t \approx \mathbf{1.00} \quad (\pm 0.01)$.

---

### 6. Analyse des Résultats

La correction géométrique est totale. Le jour initial $t=1$ et le jour final $t=1\ 000$ affichent désormais une probabilité d'échantillonnage de $1.00$, strictement identique à celle du jour central $t=500$.

L'alliance du wrapping circulaire (qui ferme l'axe du temps en anneau) et de la flexibilité de la loi géométrique a neutralisé l'effet destructeur des bordures fixes. L'emplacement d'une donnée dans le calendrier d'origine ne lui confère plus aucun privilège ni aucune pénalité statistique au cours du rééchantillonnage. Le critère d'homogénéité spatiale est validé : la stationnarité géométrique de la série est préservée.
