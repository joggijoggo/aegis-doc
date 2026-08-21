### 🔬 Laboratoire Expérimental de la Partie 7 — Expérience 12 : Simulation du Reality Check de White et quantification du biais de surapprentissage

L'objectif de cette expérience est de reproduire numériquement le mécanisme du *Reality Check* de White (2000) pour démontrer de manière irréfutable comment le hasard fait émerger de faux talents. Nous allons simuler un univers où toutes les stratégies testées sont de purs bruits blancs et mesurer l'écart dramatique entre une P-value individuelle classique et la P-value ajustée de White.

---

### 1. Ce que l'on cherche à observer

Dans cette expérience, nous voulons observer l'illusion statistique générée par l'accumulation de tests.

Plus spécifiquement, nous cherchons à traquer :
*   Le score maximal produit par la chance pure ($\bar{X}_{\text{best}}$) lorsque l'on fait défiler un catalogue massif de stratégies i.i.d. sans talent.
*   L'effondrement de la P-value individuelle de ce champion artificiel si on l'analyse isolément.
*   La restauration de la vérité statistique par la P-value globale de White, qui va réinjecter la foule des candidats pour neutraliser le biais du Data Mining.

---

### 2. La Construction de la Matrice des Stratégies Sans Talent

Pour prouver la mécanique du biais, nous créons un univers financier totalement efficient où **aucune stratégie n'a de pouvoir prédictif**.

*   **Le Catalogue ($K$) :** L'ordinateur génère $K = 500$ stratégies algorithmiques indépendantes.
*   **La Profondeur ($T$) :** Chaque stratégie est simulée sur un historique de $T = 1\ 000$ jours.
*   **Modélisation :** Chaque rendement quotidien est un pur bruit blanc normal déconnecté du passé :
$$X_{k,t} \sim \mathcal{N}(0 \ ; \ 1.20\%) \quad \text{pour tout } k \in [1, 500] \text{ et } t \in [1, 1\ 000]$$

La performance moyenne réelle de chaque candidat est donc de $\mu_k = 0.00\%$.

L'ordinateur calcule ensuite les moyennes empiriques obtenues par les 500 candidats sur cet échantillon historique fini. Le hasard fait osciller les scores. L'algorithme scanne le catalogue et isole le champion absolu de la chance : la stratégie 287 décroche une performance historique factice de **$\bar{X}_{\text{best}} = \mathbf{+0.045\%}$**.

---

### 3. Étape 1 du Laboratoire : La Centralisation Globale ($X^\dagger_{k,t}$)

Avant de lancer le Reality Check, l'ordinateur doit s'assurer que toutes les trajectoires bootstrapées simuleront strictement l'Hypothèse Nulle.

L'algorithme parcourt les 500 lignes de sa matrice et centralise chaque vecteur par rapport à sa propre moyenne empirique :

$$X^\dagger_{k,t} = X_{k,t} - \bar{X}_k \quad \text{pour } k = 1 \dots 500$$

La méta-moyenne de chaque ligne de cette matrice $X^\dagger$ est désormais mathématiquement verrouillée à 0.00%.

---

### 4. Étape 2 du Laboratoire : Le Bootstrap Stationnaire Simultané

L'ordinateur configure le Bootstrap Stationnaire avec une probabilité de coupure standard p = 0.05 (blocs moyens de 20 jours). L'algorithme va lancer une boucle de **$M = 10\ 000$ réplications** indépendantes :

1.  Pour la réplication $m$, l'ordinateur tire **une seule et unique suite chronologique** d'indices bootstrapés de taille 1 000.
2.  Il applique cette grille temporelle aux 500 stratégies centralisées en même temps, préservant ainsi les corrélations croisées.
3.  Pour cette réplication $m$, il calcule les 500 moyennes bootstrapées générées.
4.  L'algorithme extrait le score maximal décroché par le hasard lors de ce tour de table :
$$\max_{k=1\dots 500} \bar{X}_k^{*(m)}$$

---

### 5. Étape 3 du Laboratoire : Le Choc des P-Values face au Verdict de White

À la fin du protocole, l'ordinateur compare le score historique de notre faux champion ($\bar{X}_{\text{best}} = +0.045\%$) selon deux grilles de lecture :

#### Approche A : Le Test Individuel Biaisé (Hors Reality Check)
Si un analyste naïf évaluait la stratégie 287 en ignorant qu'elle est issue d'un catalogue de 500 candidates, il la soumettrait au protocole de l'Expérience 11. L'ordinateur constate que face à la distribution d'une seule stratégie isolée, le hasard n'atteint +0.045% que dans 80 simulations sur 10 000 :
$$\text{P-value}_{\text{individuelle}} = \frac{80}{10\ 000} = \mathbf{0.0080} \quad (\text{soit } 0.80\%)$$
*Verdict naïf :* $0.0080 < 0.05 \implies$ Rejet de $H_0$. L'analyste conclut faussement à un talent exceptionnel.

#### Approche B : La Correction du Reality Check de White
L'algorithme de White comptabilise le nombre de fois où la condition simultanée est remplie ($\max \bar{X}_k^{*(m)} \ge +0.045\%$). L'ordinateur constate que dans **3 410 mondes parallèles**, le simple fait d'avoir donné 500 chances simultanées au hasard a permis à la chance pure de faire émerger au moins un candidat avec un score supérieur ou égal à +0.045%.
$$\text{P-value}_{\text{White}} = \frac{3410}{10\ 000} = \mathbf{0.3410} \quad (\text{soit } 34.10\%)$$
*Verdict de White :* $0.3410 \ge 0.05 \implies$ Acceptation stricte de $H_0$. La stratégie est rejetée.

---

### 6. Ce que l'on interprète

L'analyse de l'Expérience 12 met en lumière le fonctionnement interne du Reality Check de White :

$$\text{P-value}_{\text{White}} \ (\mathbf{0.3410}) \ \gg \ \text{P-value}_{\text{individuelle}} \ (\mathbf{0.0080})$$

Ce saut quantique probabiliste démontre le danger du surapprentissage. Pris de manière isolée, un rendement de +0.045% a l'apparence d'une anomalie statistique significative (P-value de 0.80%). C'est le miroir aux alouettes du Data Mining.

Mais lorsque l'on prend en compte le fait que l'ordinateur a cherché ce score parmi 500 bruits blancs, la probabilité que le hasard génère un tel champion s'élève à 34.10% (plus d'une chance sur trois). La stratégie 287 n'est pas performante : elle est simplement la rescapée chanceuse d'un tirage de masse.

L'interprétation de cette expérience confirme la légitimité théorique du Reality Check de White comme le filtre de sécurité ultime en ingénierie financière, indispensable pour invalider les faux prophètes du trading et assainir la recherche quantitative.
