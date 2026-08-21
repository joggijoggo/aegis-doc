### 🔬 Laboratoire Expérimental de la Partie 3 — Expérience 7 : Prouver le lissage de la continuité temporelle par la loi géométrique

L'objectif de cette expérience est de démontrer numériquement que le Bootstrap Stationnaire (SB) résout le problème de la périodicité artificielle mis en évidence lors de l'Expérience 4 avec le modèle CBB. Nous allons prouver qu'en rendant la longueur des blocs aléatoire via la loi géométrique, les "frontières de rupture" déterministes (le piège des dents de scie) disparaissent au profit d'un risque de cassure parfaitement homogène dans le temps.

---

### 1. La Construction Méthodologique de l'Expérience

Pour assurer une cohérence analytique absolue avec l'Expérience 4, nous conservons le même cadre de suivi chronologique sur un vecteur d'indices séquentiels.

#### Étape A : Création du vecteur de suivi
Nous initialisons un vecteur d'indices de position temporelle allant de 1 à $T = 1\ 000$ jours :

$$I = [1, \ 2, \ 3, \ \dots, \ 1\ 000]$$

#### Étape B : Paramétrage du Bootstrap Stationnaire
Dans l'Expérience 4, la longueur de bloc fixe était de $b = 10$ jours. Pour que la comparaison soit mathématiquement rigoureuse, nous configurons notre loi géométrique pour obtenir une **longueur moyenne de bloc identique**, soit $\mathbb{E}(L) = 10$ jours.

La probabilité de coupure $p$ à instancier pour notre tirage de Bernoulli est donc :

$$p = \frac{1}{\mathbb{E}(L)} = \frac{1}{10} = \mathbf{0.10}$$

---

### 2. L'Indicateur de Continuité Temporelle Appliqué au SB

Nous réutilisons l'**indicateur binaire de continuité temporelle**, noté $C_s$, développé à l'Expérience 4. Pour chaque emplacement $s$ de la série simulée (de $s=1$ à $T-1$), cet indicateur vérifie si la chronologie réelle est conservée ($C_s = 1$) ou brisée par un saut vers un autre bloc ($C_s = 0$) :

$$C_s = \begin{cases} 1 & \text{si } i^*_{s+1} = i^*_s + 1 \\ 1 & \text{si } i^*_s = T \text{ et } i^*_{s+1} = 1 \\ 0 & \text{dans tous les autres cas} \end{cases}$$

#### Motivation du capteur dans le cadre du SB :
Dans le modèle CBB, cet indicateur a révélé que le risque de rupture était prisonnier d'un calendrier rigide (0% de continuité à chaque multiple exact de 10). Dans le Bootstrap Stationnaire, la loi géométrique ajoute une probabilité de coupure constante à chaque étape. L'indicateur $C_s$ va nous permettre de vérifier si les points de rupture sont effectivement lissés de manière imprévisible tout au long de la simulation.

---

### 3. Le Protocole Algorithmique de Mesure

1. Initialisez un vecteur de compteurs globaux de taille $T-1 = 999$ rempli de zéros : $F = [0_1, \ 0_2, \ \dots, \ 0_{999}]$.
2. Lancez une boucle de simulation de **$M = 10\ 000$ réplications** indépendantes de bootstrap stationnaire avec $p=0.10$.
3. À chaque réplication, l'ordinateur construit une série $I^*$ complète de taille 1 000 en exécutant les tirages de position et de continuité (Bernoulli).
4. L'algorithme parcourt la série générée de la position $s=1$ à $s=999$. À chaque emplacement, il calcule l'indicateur $C_s$. Si $C_s = 1$ (continuité préservée), la case $s$ du vecteur de compteurs global est incrémentée : $F_s = F_s + 1$.

---

### 4. Le Choix de la Métrique de Validation et Règle de Décision

Une fois les 10 000 simulations achevées, nous calculons la probabilité empirique de continuité à chaque emplacement $s$ :

$$P_s = \frac{F_s}{10\ 000}$$

#### Règle de décision :
*   **Si la périodicité artificielle persiste (Échec) :** Le graphique de $P_s$ affichera le motif en dents de scie calé sur les multiples de 10.
*   **Si l'homogénéité temporelle est atteinte (Succès) :** Le risque de rupture étant identique à chaque étape, la probabilité de continuité doit être stable et uniforme tout au long de la série. La cible idéale est une ligne horizontale parfaitement plate située à la hauteur mathématique de la probabilité de prolongation du bloc, soit :

$$P_s = 1 - p = 1 - 0.10 = \mathbf{0.90}$$

---

### 5. Application Numérique Théorique

Pour comprendre l'effet de lissage, analysons la probabilité de continuité à la position $s=10$, là où le CBB s'effondrait à 0%.

*   **Dans le modèle CBB :** La taille de bloc étant fixe à 10, le 10ème point correspondait *systématiquement* à la fin du bloc. Le passage au 11ème point forçait une rupture. La continuité était mathématiquement impossible.
*   **Dans le modèle Stationnaire :** Le fait d'avoir atteint la position $s=10$ n'influence pas la loi de Bernoulli (propriété de perte de mémoire). La probabilité que l'ordinateur décide de prolonger le bloc à cet endroit reste strictement égale à $1 - p = 0.90$. La rupture peut avoir eu lieu au jour 3, au jour 7, ou ne pas avoir lieu du tout.

---

### 6. Ce que vous allez observer

En traçant le graphique des probabilités $P_s$ de la position $s=1$ à $s=999$, vous constaterez un lissage absolu :
*   Le motif en dents de scie répétitif du CBB a intégralement disparu.
*   La courbe dessine une **ligne horizontale parfaitement plate**, sans aucune baisse cyclique ni déformation régulière.
*   Chaque case du vecteur affiche une valeur stable oscillant de manière infime autour de la cible théorique : $P_s \approx \mathbf{0.90} \quad (\pm 0.005)$.

---

### 7. Analyse des Résultats

La disparition du cycle de période 10 apporte la preuve mathématique de l'**homogénéité temporelle** du Bootstrap Stationnaire.

En libérant la taille des blocs grâce à la loi géométrique, les frontières de rupture déterministes ont été effacées. À n'importe quel emplacement $s$ de la simulation, la probabilité de transmettre la mémoire au point suivant est rigoureusement identique (0.90). La structure de corrélation de la série simulée ne dépend plus du temps, validant ainsi la propriété de stationnarité stricte. L'algorithme est désormais capable de répliquer fidèlement le comportement lissé d'une série financière réelle, ouvrant la voie à la sélection mathématique rigoureuse du paramètre $p$.
