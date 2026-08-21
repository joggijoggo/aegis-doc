### 🔬 Laboratoire Expérimental de la Partie 2 — Expérience 3 : Mettre en évidence la distorsion spatiale du MBB

L'objectif de cette expérience est de prouver et de quantifier numériquement l'effet de bord asymétrique généré par le Bootstrap par Blocs Mouvants (MBB) de Künsch. Nous allons démontrer que les observations situées aux extrémités de l'échantillon d'origine ont une probabilité plus faible d'apparaître dans les données rééchantillonnées, violant ainsi directement la propriété de stationnarité.

---

### 1. La Construction Méthodologique de l'Expérience

Pour isoler uniquement l'effet géométrique du découpage, nous n'avons pas besoin d'utiliser les valeurs de volatilité réelles. Nous allons travailler directement sur un **vecteur d'indices de position temporelle**, ce qui nous permettra de suivre à la trace la trajectoire de chaque jour d'historique.

#### Étape A : Création du vecteur de suivi
Créez une série temporelle factice représentée par un vecteur d'indices séquentiels allant de $1$ à $T = 1\ 000$ jours :

$$I = [1, \ 2, \ 3, \ \dots, \ 1\ 000]$$

#### Étape B : Paramétrage du MBB
Fixons une longueur de bloc standard de **$b = 50$ jours**.
L'algorithme calcule le nombre de blocs fixes disponibles via notre formule :
$$N = T - b + 1 = 1\ 000 - 50 + 1 = 951 \text{ blocs}$$

L'ordinateur stocke numériquement ces 951 morceaux :
*   $B_1 = [1, \ 2, \ \dots, \ 50]$
*   $B_2 = [2, \ 3, \ \dots, \ 51]$
*   ...
*   $B_{951} = [951, \ 952, \ \dots, \ 1\ 000]$

---

### 2. Le Cœur de la Mesure : La Fréquence Empirique d'Apparition

Pour valider l'existence de la distorsion, la métrique retenue est le **compteur empirique d'échantillonnage**, noté $F_t$.

#### Le Protocole Algorithmique :
1. Initialisez un vecteur de compteurs de taille $T=1\ 000$ rempli de zéros : $F = [0_1, \ 0_2, \ \dots, \ 0_{1\ 000}]$.
2. Lancez une boucle de simulation de **$M = 10\ 000$ réplications** de bootstrap MBB.
3. À chaque réplication, pour reconstruire une série de taille $1\ 000$, l'ordinateur tire au hasard avec remise $\frac{T}{b} = \frac{1\ 000}{50} = 20$ blocs parmi les 951 disponibles.
4. Pour chaque bloc pioché, l'algorithme examine les indices temporels qu'il contient et incrémente de $+1$ le compteur correspondant dans notre vecteur $F$.

---

### 3. Le Choix de la Métrique de Validation : L'Écart de Probabilité Théorique

Si l'algorithme préservait la stationnarité et l'équité temporelle, chaque jour $t$ devrait avoir une probabilité uniforme d'être sélectionné. Sur les 20 blocs de 50 jours tirés par simulation, l'ordinateur sélectionne au total $20 \times 50 = 1\ 000$ indices. Sur les $M = 10\ 000$ simulations, le nombre total d'indices cumulés est de $10\ 000 \times 1\ 000 = 10\ 000\ 000$.

La **fréquence théorique attendue** pour chaque jour $t$ si la méthode était stationnaire s'élève à :

$$\mathbb{E}(F_t) = \frac{\text{Nombre total d'indices piochés}}{T} = \frac{10\ 000\ 000}{1\ 000} = \mathbf{10\ 000 \text{ apparitions}}$$

---

### 4. Application Numérique et Règle de Décision

Une fois les 10 000 simulations terminées, l'ordinateur divise le vecteur des compteurs $F$ par le nombre total de réplications $M$ pour obtenir la probabilité empirique d'échantillonnage de chaque jour : $P_t = \frac{F_t}{10\ 000}$.

*   **Règle de décision :** Si le graphique de $P_t$ en fonction du temps dessine une ligne parfaitement plate à la hauteur de $P_t = 1.00$ (avec de micro-oscillations dues au hasard), le modèle respecte la stationnarité. Si la courbe s'affaisse ou se déforme aux extrémités de l'axe du temps, la stationnarité géométrique est infirmée.

---

### 5. Ce que vous allez observer

En analysant la courbe des fréquences empiriques accumulées dans le vecteur $F$, vous allez observer une déformation majeure en forme de **trapèze symétrique** :
*   Pour toutes les données centrales situées entre le jour $t = 50$ et le jour $t = 951$, la courbe est parfaitement plate. Les compteurs affichent de manière stable la valeur théorique : $F_t \approx \mathbf{10\ 000}$.
*   Pour les données des bordures extrêmes, les compteurs s'effondrent de manière linéaire. Le jour $t=1$ et le jour $t=1\ 000$ n'affichent plus qu'environ $F_1 \approx F_{1\ 000} \approx \mathbf{200}$ apparitions.

---

### 6. Analyse des Résultats

La distorsion spatiale est numériquement flagrante. Le jour initial $t=1$ a été sous-échantillonné d'un facteur 50 par rapport aux jours du milieu.

**Pourquoi ce phénomène se produit-il ?**
Parce que l'indice $1$ n'appartient qu'au tout premier bloc ($B_1$). Pour qu'il apparaisse dans la simulation, l'ordinateur doit obligatoirement piger le bloc $B_1$ (soit 1 chance sur 951). En revanche, l'indice central $500$ appartient à 50 blocs différents (du bloc $B_{451}$ au bloc $B_{500}$). Il a donc 50 fois plus de chances d'être sélectionné par le hasard.

Le MBB crée ainsi artificiellement une baisse de la variance et du poids des données aux extrémités de vos séries temporelles simulées. La structure statistique dépend de l'emplacement géographique des points sur l'axe chronologique : le postulat de stationnarité est détruit.
