## Partie 3 : L'avènement du Bootstrap Stationnaire (Politis & Romano, 1994)

Pour surmonter définitivement le dilemme biais-variance mis en lumière par l'Expérience 5, Dimitris Politis et Joseph Romano introduisent en 1994 une modification majeure de la géométrie du rééchantillonnage : le **Bootstrap Stationnaire**.

Cette méthode n’impose plus une taille de bloc fixe et arbitraire. Elle s'appuie sur la **circularité temporelle** et la **loi géométrique** pour fluidifier la structure des simulations.

---

### 1. La Loi Géométrique comme variable de lissage

Dans le bootstrap stationnaire, la longueur de chaque bloc (que l'on note désormais $L$) devient une variable aléatoire. Elle est régie par une loi géométrique de paramètre $p$, où $p$ représente la **probabilité d'arrêt ou de coupure** du bloc (avec $0 < p \le 1$).

#### Motivation mathématique :
La loi géométrique est la seule loi de probabilité discrète à posséder la propriété de **perte de mémoire** (*memoryless property*). Algébriquement, pour deux longueurs de pas de temps $n$ et $m$, on écrit :

$$\mathbb{P}(L > n + m \mid L > n) = \mathbb{P}(L > m)$$

*   **Justification :** Cette formule signifie que peu importe si un bloc est en cours depuis 1 jour, 10 jours ou 50 jours, la probabilité que ce bloc s'arrête à l'étape suivante reste strictement fixe et égale à $p$. L'algorithme efface la périodicité artificielle du CBB (Partie 2) car le moment de la rupture devient totalement imprévisible d'un pas de temps à un autre.

#### Espérance et longueur moyenne :
L'utilisateur ne choisit plus la taille exacte d'un bloc, mais calibre son espérance mathématique $\mathbb{E}(L)$ en ajustant le paramètre de coupure $p$ :

$$\mathbb{E}(L) = \frac{1}{p}$$

---

### 2. Mécanique de Tirage et de Décision Temporelle

Pour bien comprendre comment l'ordinateur construit la série, il faut détailler les deux types de tirages aléatoires indépendants qu'il doit exécuter :

1.  **Le Tirage de Position (Uniforme Discret) :** Lorsque l'algorithme doit initier ou réinitialiser un bloc, il doit piger un point de départ dans la série d'origine. Chaque jour boursier possède rigoureusement la même probabilité d'être choisi : $P = \frac{1}{T}$. Si $T=1\ 250$, l'ordinateur tire un entier au hasard entre $1$ et $1\ 250$.
2.  **Le Tirage de Continuité (Loi de Bernoulli) :** À chaque fois que l'ordinateur s'apprête à passer au jour suivant ($t+1$), il doit décider s'il continue à faire défiler l'historique ou s'il le brise. Il réalise alors une simulation de Bernoulli, qui est l'équivalent d'un lancer de pièce asymétrique :
    *   **Succès (Coupure) avec probabilité $p$ :** On tire la valeur $1$. Le bloc actuel est détruit.
    *   **Échec (Prolongation) avec probabilité $1-p$ :** On tire la valeur $0$. Le bloc continue.

---

### 3. L'Algorithme d'Exécution Numérique Détaillé

Pour construire une série rééchantillonnée complète $Y^* = [Y^*_1, \ Y^*_2, \ \dots, \ Y^*_T]$ de même taille $T$ que l'échantillon d'origine, l'ordinateur utilise une variable pivot (un pointeur de lecture) notée $K$. Ce pointeur mémorise l'indice du jour réel que l'on est en train de copier.

Voici la décomposition de la structure logique de la boucle :

#### Étape A : Initialisation du premier point ($t = 1$)
L'algorithme démarre obligatoirement par la création d'un nouveau bloc.
1. L'ordinateur effectue le premier *Tirage de Position* et génère un nombre entier aléatoire compris entre $1$ et $T$.
2. Supposons qu'il obtienne la valeur $342$. Le pointeur prend cette valeur : $K = 342$.
3. La première case de notre simulation reçoit la valeur correspondante : $Y^*_1 = Y_{342}$.

#### Étape B : Itération de l'axe du temps (de $t = 2$ à $T$)
Pour chaque position temporelle suivante, l'ordinateur doit exécuter le *Tirage de Continuité* (Loi de Bernoulli de paramètre $p$) pour choisir entre deux branches logiques exclusives :

*   **Branche LOGIQUE 1 : Le déclenchement d'une rupture (Probabilité $p$)**
    Si le test de Bernoulli renvoie $1$, le bloc actuel s'arrête immédiatement.
    1. L'ordinateur réinitialise le pointeur en effectuant un tout nouveau *Tirage de Position* uniforme entre $1$ et $T$. Supposons qu'il pige $812$.
    2. Le pointeur est mis à jour : $K = 812$.
    3. On enregistre la donnée : $Y^*_t = Y_{812}$.

*   **Branche LOGIQUE 2 : La prolongation chronologique (Probabilité $1-p$)**
    Si le test de Bernoulli renvoie $0$, la mémoire doit être transmise de manière fluide.
    1. L'ordinateur fait avancer le pointeur d'une unité vers le futur : $K = K + 1$.
    2. **Le filtre de circularité (*Wrapping*) :** On vérifie si le pointeur dépasse la fin de l'échantillon réel. Si $K = T + 1$, alors l'ordinateur écrase cette valeur et force un retour au tout début de la série : $K = 1$.
    3. On enregistre la donnée : $Y^*_t = Y_K$.

L'algorithme répète cette double vérification à chaque pas de temps jusqu'à ce que la case $Y^*_T$ soit remplie.

---

### 4. Application Numérique Théorique

Appliquons cette mécanique pas à pas pour générer les 5 premiers jours d'une simulation ($T=5$) sur notre série historique. Fixons le paramètre de coupure à **$p = 0.40$**.
La longueur moyenne théorique de nos blocs sera de : $\mathbb{E}(L) = \frac{1}{0.40} = 2.5 \text{ jours}$.

*   **Étape $t = 1$ (Démarrage obligatoire) :** L'ordinateur effectue un tirage uniforme entre $1$ et $5$. Supposons qu'il sélectionne l'indice $K = 3$.
    *   *Résultat :* $Y^*_1 = Y_3$
*   **Étape $t = 2$ (Continuer ou Couper ?) :** L'algorithme simule une loi de Bernoulli avec une probabilité de coupure $p=0.40$. Le sort décide de "Prolonger" (60 % de chance). L'ordinateur incrémente l'indice précédent : $K = 3 + 1 = 4$.
    *   *Résultat :* $Y^*_2 = Y_4$
*   **Étape $t = 3$ (Continuer ou Couper ?) :** Nouveau jet de Bernoulli. Le sort décide à nouveau de "Prolonger". L'ordinateur incrémente l'indice : $K = 4 + 1 = 5$.
    *   *Résultat :* $Y^*_3 = Y_5$
*   **Étape $t = 4$ (Application du Wrapping circulaire) :** Le sort décide encore de "Prolonger". L'ordinateur incrémente l'indice : $K = 5 + 1 = 6$. Comme l'indice $6$ dépasse la taille de la série ($T=5$), la règle du wrapping circulaire ramène automatiquement l'indice au point de départ : $K = 1$.
    *   *Résultat :* $Y^*_4 = Y_1$
*   **Étape $t = 5$ (Déclenchement d'une rupture) :** Nouveau jet de Bernoulli. Cette fois, le sort tire "Coupure" (40 % de chance). Le bloc actuel se brise. L'ordinateur réinitialise le processus et effectue un tout nouveau tirage uniforme indépendant entre $1$ et $5$. Supposons qu'il pige l'indice $K = 2$.
    *   *Résultat :* $Y^*_5 = Y_2$

#### Structure finale de l'échantillon simulé :
Notre série synthétique $Y^*$ est composée de deux blocs distincts nés du hasard : un premier bloc de longueur $L_1 = 4$ jours, suivi d'un second bloc de longueur $L_2 = 1$ jour.

$$Y^* = [\underbrace{Y_3, \ Y_4, \ Y_5, \ Y_1}_{\text{Bloc circulaire de taille 4}}, \quad \underbrace{Y_2}_{\text{Bloc de taille 1}}]$$

---

### 5. Justification Théorique de la Stationnarité

Pourquoi cette formulation géométrique garantit-elle la stationnarité de la série rééchantillonnée $Y^*$, là où le MBB et le CBB échouaient ?

1.  **Homogénéité spatiale :** Grâce au wrapping circulaire, chaque observation de la série originale possède exactement la même probabilité ($\frac{1}{T}$) d'être sélectionnée comme point de départ lors d'une initialisation ou d'une coupure. Aucune zone (milieu ou bord) n'est favorisée.
2.  **Homogénéité temporelle :** La probabilité de transition entre deux observations dans la simulation est constante. Pour n'importe quel emplacement $t$, la probabilité que la donnée suivante $Y^*_{t+1}$ soit le successeur chronologique réel de $Y^*_t$ vaut toujours $1 - p$. La probabilité qu'elle subit une cassure vaut toujours $p$.

La structure de corrélation et de covariance est ainsi lissée et uniformément distribuée le long de l'échantillon simulé. Elle ne dépend plus du temps $t$, ce qui valide mathèmatique la propriété de stationnarité stricte du bootstrap de Politis et Romano.
