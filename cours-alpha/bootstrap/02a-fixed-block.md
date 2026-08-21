### 📐 Partie 2 : La géométrie du rééchantillonnage — L'évolution des blocs fixes (MBB et CBB)

Pour préserver la mémoire temporelle d'une série chronologique, le rééchantillonnage ne doit plus se faire point par point, mais par **groupes d'observations consécutives** (des blocs). Avant d'aboutir au bootstrap stationnaire, la recherche en économétrie est passée par deux modèles géométriques majeurs fondés sur des blocs de taille fixe.

---

### 1. Le Bootstrap par Blocs Mouvants (Moving Block Bootstrap - MBB)

Introduit par Künsch (1989), le MBB est la méthode pionnière du rééchantillonnage dépendant. Son principe repose sur le découpage de la série originale en morceaux contigus d'une longueur strictement fixe, notée $b$.

#### Algorithme de construction et décomposition mathématique :
Soit une série originale de taille $T$. Pour une longueur de bloc $b$ choisie par l'utilisateur, l'algorithme fait glisser une fenêtre de lecture d'un jour à la fois, du début à la fin de l'historique.

Le nombre total de blocs fixes disponibles dans les données est déterminé par la formule :
$$N = T - b + 1$$

Chaque bloc individuel, noté $B_i$ (ou l'indice de position $i$ varie de $1$ à $N$), est un vecteur défini par :
$$B_i = [Y_i, \ Y_{i+1}, \ \dots, \ Y_{i+b-1}]$$

#### 🔢 Application numérique du MBB :
Reprenons notre échantillon de volatilité sur $T = 6$ jours :
$$Y = [t_1: 1\%, \ t_2: 4\%, \ t_3: 9\%, \ t_4: 4\%, \ t_5: 2\%, \ t_6: 5\%]$$

Fixons une taille de bloc rigide de **$b = 3$ jours**. L'algorithme calcule le nombre de blocs disponibles : $N = 6 - 3 + 1 = 4$ blocs.
L'ordinateur construit et stocke numériquement les 4 blocs de l'urne :
*   **Bloc $B_1$ (commence en $t_1$) :** $[1\%, \ 4\%, \ 9\%]$
*   **Bloc $B_2$ (commence en $t_2$) :** $[4\%, \ 9\%, \ 4\%]$
*   **Bloc $B_3$ (commence en $t_3$) :** $[9\%, \ 4\%, \ 2\%]$
*   **Bloc $B_4$ (commence en $t_4$) :** $[4\%, \ 2\%, \ 5\%]$

Pour simuler une nouvelle série de 6 jours, l'ordinateur tire au hasard avec remise $\frac{T}{b} = \frac{6}{3} = 2$ blocs dans cette urne. Si le sort sélectionne le Bloc $B_3$ puis le Bloc $B_1$, la série bootstrapée finale sera :
$$Y^*_{\text{MBB}} = [\underbrace{9\%, \ 4\%, \ 2\%}_{B_3}, \ \underbrace{1\%, \ 4\%, \ 9\%}_{B_1}]$$

#### Le défaut théorique du MBB : La distorsion géométrique des bordures
Bien que le MBB conserve l'enchaînement des jours à l'intérieur de chaque bloc, **il détruit la stationnarité de la série**. La stationnarité exige que la probabilité d'apparition d'une donnée ne dépende pas de sa position initiale dans le temps. Or, la géométrie des blocs fixes crée un effet de bord asymétrique.

Analysons la fréquence d'apparition de chaque jour dans notre urne de 4 blocs :
*   Le jour initial $t_1$ ($1\%$) n'est présent **que dans le Bloc $B_1$**. Il se situe obligatoirement en première position du bloc. Sa probabilité globale d'être tiré est faible.
*   Le jour final $t_6$ ($5\%$) n'est présent **que dans le Bloc $B_4$**. Il se situe obligatoirement en dernière position du bloc.
*   Le jour central $t_3$ ($9\%$) est présent **dans le Bloc $B_1$, le Bloc $B_2$ et le Bloc $B_3$**. Il peut occuper la première, la deuxième ou la troisième place d'un bloc.

**Conséquence numérique :** Les données situées au cœur de la série temporelle d'origine ont géométriquement plus de poids dans les simulations que les données des extrémités. La série rééchantillonnée change de comportement statistique sur ses bords : la stationnarité est perdue.

---

### 2. Le Bootstrap Circulaire par Blocs (Circular Block Bootstrap - CBB)

Pour corriger cette anomalie des bordures, Politis et Romano ont introduit en 1992 le CBB. L'idée fondamentale est d'éliminer mathématiquement les barrières du début ($t_1$) et de la fin ($t_T$) en rendant l'axe du temps circulaire.

#### Algorithme de construction et mécanisme de circularité (*Wrapping*) :
Dans le CBB, le temps est modélisé comme un anneau où le successeur immédiat du dernier jour $t_T$ devient le premier jour $t_1$. Algébriquement, les indices temporels sont calculés **modulo $T$**.

Grâce à cette rotation, l'algorithme peut faire glisser sa fenêtre de lecture de taille fixe $b$ au-delà du jour $t_T$. Le nombre de blocs disponibles dans l'urne augmente et devient **strictement égal à la taille de l'échantillon $T$** :
$$N_{\text{CBB}} = T$$

#### 🔢 Application numérique du CBB :
Reprenons notre série de volatilité sur $T = 6$ jours avec une taille de bloc fixe de $b = 3$ jours. Grâce au bouclage circulaire, l'ordinateur génère 6 blocs équiprobables :
*   **Bloc $B_1$ (départ $t_1$) :** $[1\%, \ 4\%, \ 9\%]$
*   **Bloc $B_2$ (départ $t_2$) :** $[4\%, \ 9\%, \ 4\%]$
*   **Bloc $B_3$ (départ $t_3$) :** $[9\%, \ 4\%, \ 2\%]$
*   **Bloc $B_4$ (départ $t_4$) :** $[4\%, \ 2\%, \ 5\%]$
*   **Bloc $B_5$ (départ $t_5$, circulaire) :** $[2\%, \ 5\%, \ 1\%]$ *(le jour $t_6$ est suivi de $t_1$)*
*   **Bloc $B_6$ (départ $t_6$, circulaire) :** $[5\%, \ 1\%, \ 4\%]$ *(le jour $t_6$ est suivi de $t_1$ puis $t_2$)*

#### Le bénéfice géométrique du CBB :
Si vous comptez le nombre d'apparitions de chaque jour dans cette nouvelle urne, vous constaterez qu'ils apparaissent tous exactement **3 fois** ($b$ fois de manière générale). Chaque point de votre historique de départ possède désormais la même probabilité mathématique d'être échantillonné ($\frac{b}{T}$). L'asymétrie spatiale des bordures du MBB est intégralement corrigée.

#### Le problème persistant du CBB : Le piège de la périodicité artificielle
Bien que le CBB redistribue équitablement les chances de tirage, il échoue à garantir la stationnarité de la série rééchantillonnée à cause d'une **dépendance structurelle forcée**.

Regardez ce qui se passe à la jonction entre deux blocs lors d'une simulation CBB. L'ordinateur pioche des blocs entiers de taille fixe $b=3$. Dans la série simulée, deux éléments adjacents aux positions $t$ et $t+1$ ont :
*   **99% de chances** d'être parfaitement consécutifs dans le temps réel (s'ils se trouvent à l'intérieur d'un même bloc).
*   **100% de chances** d'être totalement indépendants et déconnectés s'ils se situent exactement à la charnière entre la fin du premier bloc et le début du suivant.

Cette rupture de mémoire se produit de manière déterministe et rigide **tous les 3 jours**. La série simulée présente une cassure systématique à intervalles réguliers. Statistiquement, la structure de covariance de la série générée varie en fonction du temps : elle présente une dépendance cyclique artificielle de période $b$. Elle n'est donc toujours pas stationnaire, ce qui a rendu nécessaire l'introduction d'un second paramètre de lissage : la longueur de bloc aléatoire.

---

### Synthèse de la Partie 2 (Étape Intermédiaire)
*   **MBB (1989) :** Introduit des blocs de taille fixe, mais crée une asymétrie spatiale (le milieu de la série pèse plus lourd que les bords).
*   **CBB (1992) :** Corrige l'effet de bord en rendant le temps circulaire, mais introduit des points de rupture déterministes tous les $b$ jours, créant un cycle artificiel non stationnaire.
