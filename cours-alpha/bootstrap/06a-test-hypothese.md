## Partie 6 : Application pratique — Les Tests d'Hypothèses et la validation de stratégies

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
Pour forcer le monde simulé à obéir à $H_0$, l'algorithme takes la série des rendements historiques de la stratégie ($X_t$), calcule sa moyenne historique ($\bar{X}$), et la soustrait à chaque point de la série. On obtient une série modifiée, dite "centralisée", notée $X^\dagger_t$ :

$$X^\dagger_t = X_t - \bar{X}$$

*   **Motivation :** Par construction, la moyenne arithmétique de cette nouvelle série $X^\dagger$ est **exactement égale à 0.00%**. Elle représente mathématiquement un gérant qui ne gagne rien (le hasard pur), mais elle conserve l'intégralité de la structure de volatilité, d'autocorrélation et de mémoire de la série d'origine.

#### 🔢 Application numérique de la Centralisation :
Supposons que sur un historique de $T = 1\ 000$ jours, l'algorithme du gérant affiche un rendement moyen quotidien de $\bar{X} = +0.05\%$.
*   Si au jour 12, le fonds a gagné $+0.12\%$, sa valeur centralisée devient : $X^\dagger_{12} = 0.12\% - 0.05\% = \mathbf{+0.07\%}$.
*   Si au jour 13, le fonds a perdu $-0.08\%$, sa valeur centralisée devient : $X^\dagger_{13} = -0.08\% - 0.05\% = \mathbf{-0.13\%}$.

---

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
