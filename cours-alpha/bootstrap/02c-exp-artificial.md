### 🔬 Laboratoire Expérimental de la Partie 2 — Expérience 4 : Prouver la périodicité artificielle du CBB

L'objectif de cette expérience est de mettre en évidence la distorsion temporelle générée par le Bootstrap Circulaire par Blocs (CBB). Nous allons démontrer que l'utilisation de blocs à taille fixe introduit des "frontières de rupture" déterministes à intervalles réguliers. Ce comportement crée un cycle artificiel qui détruit la stationnarité de la structure de corrélation.

---

### 1. La Construction Méthodologique de l'Expérience

Pour traquer l'apparition d'un cycle artificiel, nous allons analyser la probabilité que deux jours consécutifs dans notre simulation soient également consécutifs dans la réalité.

#### Étape A : Création du vecteur de suivi
Comme pour l'expérience précédente, nous utilisons un vecteur d'indices temporels séquentiels allant de 1 à $T = 1\ 000$ jours :

$$I = [1, \ 2, \ 3, \ \dots, \ 1\ 000]$$

#### Étape B : Paramétrage du CBB Circularisé
Fixons une longueur de bloc rigide de **$b = 10$ jours**. Grâce au mécanisme de circularité (*wrapping*), le nombre de blocs disponibles est strictement égal à $T$, soit 1 000 blocs.
*   $B_1 = [1, \ 2, \ \dots, \ 10]$
*   ...
*   $B_{1\ 000} = [1\ 000, \ 1, \ \dots, \ 9]$ *(Bloc circulaire)*

---

### 2. L'Indicateur de Continuité Temporelle ($C_s$) : Définition et Construction

Pour mesurer le comportement de la mémoire aux jointures des blocs, nous devons créer un capteur mathématique capable de détecter si le fil chronologique est préservé ou brisé d'un emplacement à un autre dans la série simulée. Ce capteur est l'**indicateur binaire de continuité temporelle**, noté $C_s$.

Pour une série simulée notée $I^* = [i^*_1, \ i^*_2, \ \dots, \ i^*_s, \ i^*_{s+1}, \ \dots, \ i^*_T]$, la construction algébrique de cet indicateur à l'emplacement $s$ (où $s$ varie de $1$ à $T-1$) est régie par la fonction logique suivante :

$$C_s = \begin{cases} 1 & \text{si } i^*_{s+1} = i^*_s + 1 \\ 1 & \text{si } i^*_s = T \text{ et } i^*_{s+1} = 1 \\ 0 & \text{dans tous les autres cas} \end{cases}$$

#### Décomposition pas à pas de la formule :
*   **La condition nominale ($i^*_{s+1} = i^*_s + 1$) :** L'indicateur renvoie $1$ si la donnée située juste après $i^*_s$ dans la simulation est précisément son successeur naturel dans le calendrier réel (par exemple, si le jour réel 45 est suivi du jour réel 46).
*   **La condition de circularité ($i^*_s = T$ et $i^*_{s+1} = 1$) :** L'indicateur renvoie également $1$ si le jour simulé est le tout dernier jour de l'échantillon d'origine ($1\ 000$) et qu'il est suivi du tout premier jour ($1$). C'est la validation de la soudure circulaire.
*   **La condition de rupture ($0$ partout ailleurs) :** L'indicateur renvoie $0$ dès que la chronologie subit un saut artificiel provoqué par l'algorithme (par exemple, si le jour réel 10 est immédiatement suivi du jour réel 742 parce que l'ordinateur a pigé un nouveau bloc au hasard).

---

### 3. Pourquoi utiliser la Continuité Temporelle ? (Motivation Économétrique)

#### Ce que cet indicateur cherche à expliquer :
Dans une série temporelle réelle et stationnaire, le risque de subir un changement de dynamique ou une rupture d'information est **homogène dans le temps**. La mémoire ne s'interrompt pas selon un calendrier fixe déterminé par un utilisateur.

L'indicateur $C_s$ a pour mission de mesurer si l'algorithme CBB injecte une signature mathématique invisible mais déterministe dans la série synthétique. En suivant la proportion de $1$ et de $0$ à chaque emplacement spécifique $s$, on vérifie si la structure de la mémoire dépend — ou non — du moment où on l'observe.

#### Pourquoi cette construction binaire est-elle légitime ?
La mémoire d'une série financière (comme l'autocorrélation de Pearson étudiée en Partie 1) est directement portée par la contiguïté des observations. Si $C_s = 1$, la dépendance temporelle est transmise à 100% au point suivant. Si $C_s = 0$, la dépendance est instantanément coupée, et la mémoire est réinitialisée à zéro.

Étudier la moyenne de $C_s$ revient à cartographier le "système nerveux" de la simulation : on localise avec exactitude où se trouvent les zones de transmission fluide et où se trouvent les barrières de déconnexion.

---

### 4. Le Protocole Algorithmique de Mesure Accumulée

Pour lisser les fluctuations aléatoires et obtenir une valeur probabiliste robuste, la mesure doit être répétée à grande échelle à travers un vecteur de stockage.

1.  Initialisez un vecteur de compteurs globaux de taille $T-1 = 999$ rempli de zéros : $F = [0_1, \ 0_2, \ \dots, \ 0_{999}]$. Chaque case de ce vecteur correspond à un emplacement géographique de notre future série simulée.
2.  Lancez une boucle de simulation de **$M = 10\ 000$ réplications** indépendantes de bootstrap CBB.
3.  À chaque réplication, l'ordinateur construit une série $I^*$ complète en pigeant au hasard avec remise $\frac{T}{b} = \frac{1\ 000}{10} = 100$ blocs fixes de taille 10.
4.  L'algorithme parcourt la série générée du premier emplacement ($s=1$) jusqu'au 999ème ($s=999$). À chaque position $s$, il calcule la valeur de l'indicateur $C_s$. Si $C_s = 1$, l'ordinateur incrémente de $+1$ la case correspondante du vecteur de compteurs global : $F_s = F_s + 1$.

---

### 5. Le Choix de la Métrique de Validation et Règle de Décision

Une fois les 10 000 réplications achevées, la métrique de validation finale est la **probabilité empirique de continuité**, notée $P_s$. Elle s'obtient en divisant chaque compteur accumulé par le nombre total de simulations :

$$P_s = \frac{F_s}{10\ 000}$$

#### Règle de décision :
*   **Si la méthode préserve la stationnarité :** Le risque de rupture doit être uniformément réparti. Le graphique de $P_s$ en fonction de la position $s$ doit tracer une ligne horizontale parfaitement plate, située à la hauteur théorique de la proportion moyenne de continuité de l'algorithme (ici, comme il y a 9 transitions internes pour 1 transition de rupture par bloc, la ligne devrait être stable à $P_s = \frac{9}{10} = 0.90$).
*   **Si la méthode souffre de périodicité artificielle :** Le graphique affichera des variations régulières et répétitives calées sur la longueur fixe $b$. L'hypothèse de stationnarité temporelle sera alors formellement rejetée.

---

### 6. Ce que vous allez observer

En traçant la courbe des probabilités $P_s$ de la position $s=1$ à $s=999$, vous allez observer un motif géométrique en **dents de scie d'une régularité absolue**, répété de manière cyclique exactement 100 fois :
*   Aux emplacements $s = 1, 2, 3, 4, 5, 6, 7, 8, 9$, la probabilité de continuité est rigoureusement égale à **100%** ($P_s = 1.00$). La chronologie n'est jamais rompue à ces endroits.
*   À l'emplacement exact **$s = 10$**, ainsi qu'à tous ses multiples exacts ($20, 30, 40, \dots, 990$), la probabilité de continuité s'effondre brutalement pour atteindre une valeur quasi-nulle de **0%** ($P_{10} \approx 0.001$).

---

### 7. Analyse des Résultats

La détection de ce cycle de période 10 apporte la preuve mathématique de la non-stationnarité du modèle CBB.

**Pourquoi ce motif apparaît-il ?**
Parce qu'en figeant la taille des blocs à $b=10$, l'utilisateur a programmé une frontière de rupture déterministe. Les 9 premiers points proviennent obligatoirement de l'intérieur du bloc pioché (continuité absolue), tandis que le 10ème point correspond obligatoirement à la charnière de transition vers un autre bloc tiré au hasard (rupture quasi-totale).

Dans la série simulée par CBB, la structure de la mémoire dépend donc entièrement du temps $s$. Deux points adjacents sont parfaitement corrélés si on les observe au milieu d'un bloc, mais totalement indépendants si on les observe à la frontière d'un multiple de 10. Ce comportement cyclique artificiel est absent des séries financières réelles. Pour gommer ces charnières prévisibles, il devient indispensable de rendre le point de rupture totalement imprévisible à chaque pas de temps, ce qui motive l'introduction de la loi géométrique dans le bootstrap stationnaire.
