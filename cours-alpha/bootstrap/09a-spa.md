## Partie 9 : Le Test de Supériorité Prédictive de Hansen (SPA — 2005)

Le *Reality Check* de Halbert White (2000) a posé les jalons de la lutte contre le biais de surapprentissage. Cependant, Peter Hansen démontre en 2005 que l'approche de White souffre d'un défaut structurel majeur : elle est asymptotiquement **conservatrice** et manque de puissance statistique en présence de stratégies non pertinentes [🌐-0].

Hansen introduit le test de **Supériorité Prédictive (SPA — *Superior Predictive Ability*)**, qui réajuste dynamiquement le recentrage de la distribution pour offrir un diagnostic infiniment plus précis [🌐-0].

---

### 1. La faille du Reality Check de White : La sensibilité aux modèles non pertinents

La faiblesse du modèle de White (2000) réside dans la manière dont il gère la centralisation sous l'Hypothèse Nulle $H_0$. Pour rappel, White décale toutes les stratégies du catalogue de manière uniforme en soustrayant leur moyenne empirique ($\bar{X}_k$).

#### Le problème économétrique :
Imaginez qu'un chercheur teste un catalogue de $K = 1\ 000$ stratégies. Parmi elles :
*   La stratégie 1 est excellente et affiche une moyenne empirique de $+0.05\%$.
*   Les stratégies 2 à 1 000 sont catastrophiques (bogues de code, mauvaise logique) et affichent une moyenne de $-0.45\%$.

En appliquant la centralisation brute de White, les 999 mauvaises stratégies sont artificiellement rehaussées de $+0.45\%$ pour forcer leur moyenne bootstrapée à valoir $0.00\%$. Ce faisant, on injecte une quantité massive de variance (de volatilité) artificielle au dénominateur du test global.

Lors de l'étape du Bootstrap Simultané, la variable pivot $\max_k \bar{X}_k^{*(m)}$ va capter les oscillations de ces 999 mauvaises stratégies réadaptées. Le hasard va fréquemment produire de faux plafonds très élevés (par exemple $+0.08\%$), écrasant la performance réelle de notre champion ($+0.05\%$). La $\text{P-value}_{\text{White}}$ va alors exploser de manière artificielle (par exemple à $31\%$), conduisant au rejet d'un algorithme potentiellement excellent. Le test de White est biaisé par l'inclusion de « déchets » statistiques.

### 2. Le fonctionnement du filtre conditionnel de recentrage de Hansen (2005)

Pour éliminer la pollution des modèles médiocres qui fait exploser la variance du test de White, Peter Hansen (2005) modifie radicalement la règle de recentrage de l'Hypothèse Nulle $H_0$ [🌐-0]. Au lieu d'imposer une moyenne de $0.00\%$ à l'ensemble du catalogue sans distinction, le test SPA évalue la viabilité statistique de chaque stratégie $k$ à l'aide d'un **filtre conditionnel de recentrage**, noté $\mu_k^c$ [🌐-0].

La moyenne théorique assignée à la stratégie $k$ lors du bootstrap dépend de sa performance historique réelle face à un seuil de pénalisation dynamique basé sur la Loi du Logarithme Itéré (LIL) :

$$\mu_k^c = \begin{cases} 0 & \text{si } \bar{X}_k > - \text{Seuil}_{\text{LIL}, k} \\ \bar{X}_k & \text{sinon} \end{cases}$$

Où le filtre de détection du bruit et de tri des données s'établit par l'équation suivante :

$$\text{Seuil}_{\text{LIL}, k} = \sqrt{\frac{2 \times \ln(\ln(T))}{T}} \times \omega_k$$

#### Paramètre A : Le Seuil de Pénalisation Asymptotique LIL
Ce paramètre dicte la frontière d'exclusion économique en deçà de laquelle une stratégie est déclarée hors-jeu. Sa vitesse de resserrement est gouvernée par le numérateur doublement logarithmique $\ln(\ln(T))$, divisé par la taille de l'échantillon $T$.

*   **🎯 Justification physique et économétrique :** Pourquoi s'appuyer sur la Loi du Logarithme Itéré plutôt que sur la règle de Bartlett classique ? Hansen cherche un outil mathématique capable de tracer une frontière de démarcation asymptotique stricte entre les modèles dont la moyenne est négative par pur hasard (bruit blanc local qui s'équilibrera avec le temps) et les modèles dont la moyenne est structurellement négative (mauvaise logique intrinsèque). La LIL offre la vitesse de convergence idéale : elle resserre le seuil de manière asymétrique pour s'assurer qu'aucune stratégie structurellement déficitaire ne soit requalifiée par erreur en concurrent légitime sous $H_0$.
*   **🔢 Application numérique du Paramètre A :** Analysons un historique financier de $T = 1\ 000$ jours pour une stratégie donnée. Calculons la composante d'échelle temporelle de la LIL :
    1. Logarithme népérien de la taille : $\ln(1000) \approx 6.90775$
    2. Second logarithme imbriqué : $\ln(6.90775) \approx 1.93267$
    3. Multiplication du numérateur : $2 \times 1.93267 = 3.86534$
    4. Division par l'échelle de temps : $\frac{3.86534}{1000} = 0.00386534$
    5. Extraction de la racine carrée : $\sqrt{0.00386534} \approx \mathbf{0.06217}$

#### Paramètre B : L'Écart-Type de Long Terme de la Stratégie ($\omega_k$)
Ce paramètre représente l'écart-type de la distribution de long terme des rendements de la stratégie $k$. Il est calculé en extrayant la racine carrée de l'estimateur spectral à noyau plat à fréquence zéro, selon la géométrie de Patton (2009) :
$$\omega_k = \sqrt{D_{\text{SB}, k}}$$

*   **🎯 Justification physique et économétrique :** Chaque stratégie possède sa propre signature de risque. Un algorithme de trading haute fréquence à faible levier peut afficher des rendements très stables (faible $\omega_k$), tandis qu'un modèle de suivi de tendance macroéconomique sur options affichera des mouvements violents (grand $\omega_k$). Hansen refuse d'appliquer un seuil uniforme à tout le monde. En multipliant le coefficient LIL par $\omega_k$, l'algorithme adapte la sévérité du filtre à la nature de la stratégie : un modèle très volatil se verra attribuer un seuil d'exclusion large pour ne pas être éliminé au premier faux mouvement aléatoire, tandis qu'un modèle à haute régularité subira un contrôle resserré et intransigeant.
*   **🔢 Application numérique du Paramètre B :** Prenons deux algorithmes déficitaires concurrents au sein de notre catalogue, ayant tous deux obtenu la même performance historique négative de $\bar{X} = -0.05\%$.
    *   *Stratégie A (Paul, Régulière) :* Son estimateur à noyau plat évalue sa variance de long terme à $D_{\text{SB}, A} = 0.16 \implies \omega_A = \sqrt{0.16} = \mathbf{0.40\%}$.
    *   *Stratégie B (Virginie, Volatile) :* Son estimateur évalue sa variance à $D_{\text{SB}, B} = 1.44 \implies \omega_B = \sqrt{1.44} = \mathbf{1.20\%}$.

Calculons les deux seuils critiques d'exclusion personnalisés en reprenant notre coefficient LIL de $0.06217$ :
$$\text{Seuil}_{\text{LIL}, A} = - (0.06217 \times 0.40\%) = \mathbf{-0.0248\%}$$
$$\text{Seuil}_{\text{LIL}, B} = - (0.06217 \times 1.20\%) = \mathbf{-0.0746\%}$$

#### Règle de Décision Conditionnelle du Filtre SPA :
Une fois les barrières fixées, l'ordinateur applique le tri exclusif de Hansen avant de lancer les simulations :
*   **Cas de Paul (Stratégie A) :** L'ordinateur compare sa moyenne historique ($\bar{X}_A = -0.05\%$) face à sa barrière ($\text{Seuil}_{\text{LIL}, A} = -0.0248\%$). Comme $-0.05\% \le -0.0248\%$, la condition d'exclusion est activée. L'algorithme déclare la stratégie de Paul comme structurellement déficitaire. Son verdict de recentrage est $\mu_A^c = \bar{X}_A = \mathbf{-0.05\%}$. La série de Paul ne sera **pas** rehaussée à $0.00\%$. Elle reste ancrée dans sa zone de perte réelle.
*   **Cas de Virginie (Stratégie B) :** L'ordinateur compare sa moyenne historique ($\bar{X}_B = -0.05\%$) face à sa barrière élargie ($\text{Seuil}_{\text{LIL}, B} = -0.0746\%$). Comme $-0.05\% > -0.0746\%$, la condition d'exclusion échoue. Bien que Virginie affiche une perte en apparence identique à celle de Paul, sa forte volatilité démontre que cette baisse a une probabilité d'être un simple accident dû au hasard. L'algorithme lui accorde le bénéfice du doute. Son verdict de recentrage est $\mu_B^c = \mathbf{0.00\%}$. La série de Virginie est centralisée à zéro et participera activement aux tirages du bootstrap simultané.

### 3. Les trois P-values du Test SPA

Grâce à ce filtre conditionnel dépendant de la sévérité du seuil, l'algorithme de Hansen ne calcule pas une seule P-value, mais encadre la vérité statistique à l'aide de trois indicateurs :

1.  **$\text{P-value}_{\text{CONS}}$ (Conservatrice) :** Équivalente à la P-value de White (2000). On suppose que toutes les stratégies sont viables ($\mu_k^c = 0 \ Bowen \forall k$). C'est la borne supérieure de l'erreur.
2.  **$\text{P-value}_{\text{SPA}}$ (Ajustée/Conditionnelle) :** Utilise le filtre conditionnel de Hansen décomposé ci-dessus. C'est l'estimateur le plus précis, épuré des modèles non pertinents.
3.  **$\text{P-value}_{\text{MIN}}$ (Libérale) :** Borne inférieure stricte. On applique $\mu_k^c = \bar{X}_k$ pour toutes les stratégies dont la moyenne est négative, même celles proches du seuil, maximisant les chances de validation.

---

### 4. Application Numérique Comparative : White vs Hansen SPA

Soumettons un catalogue de $K = 1\ 000$ algorithmes testés sur $T = 1\ 000$ jours aux deux méthodologies ($T^{1/3}=10$). Notre meilleure stratégie affiche un score réel de $\bar{X}_{\text{best}} = +0.05\%$. Le catalogue contient 950 stratégies médiocres dont la moyenne historique plonge à $-0.45\%$. Le seuil critique moyen de Hansen l'échantillon est évalué à $-0.03\%$.

#### Étape A : Le traitement par le Reality Check de White
L'ordinateur centralise les 1 000 stratégies de manière uniforme. Les 950 mauvaises stratégies subissent toutes un décalage massif de $+0.45\%$ pour atteindre $0.00\%$.
Lors des 10 000 réplications simultanées, ces séries artificiellement dopées produisent de violents artefacts de performance. Le maximum de la chance dépasse $+0.05\%$ dans 3 100 simulations.
$$\text{P-value}_{\text{White}} = \frac{3100}{10000} = \mathbf{0.3100} \quad (\text{soit } 31.00\%)$$
*Verdict de White :* $0.3100 \ge 0.05 \implies$ Acceptation de $H_0$. La stratégie est rejetée, jugée comme un produit du Data Mining.

#### Étape B : Le traitement par le Test SPA de Hansen
L'ordinateur applique le filtre conditionnel $\mu_k^c$ :
*   Pour les 50 stratégies viables : $\bar{X}_k \ge -0.03\% \implies \mu_k^c = \mathbf{0}$. Elles sont centralisées à $0.00\%$ et participent au jeu.
*   Pour les 950 stratégies médiocres : $\bar{X}_k = -0.45\% < -0.03\% \implies \mu_k^c = \mathbf{-0.45\%}$. Elles ne sont **pas** centralisées et restent figées dans leur zone de perte.

L'ordinateur lance les 10 000 réplications de bootstrap stationnaire synchrone. Libéré de la pollution des 950 bruits blancs artificiels, le maximum du hasard ($\max_k \bar{X}_k^{*(m)}$) est calculé uniquement sur la base des 50 vrais concurrents. Le hasard réel ne parvient à égaler ou dépasser $+0.05\%$ dans 210 simulations.
$$\text{P-value}_{\text{SPA}} = \frac{210}{10000} = \mathbf{0.0210} \quad (\text{soit } 2.10\%)$$
*Verdict de Hansen :* $0.0210 < 0.05 \implies$ Rejet de $H_0$. La stratégie est validée.

---

### 5. Ce que l'on interprète (Le verdict ultime)

L'analyse comparative de cet arbitrage met en évidence la supériorité économétrique de l'approche de Hansen (2005) [🌐-0] :

$$\text{P-value}_{\text{SPA}} \ (\mathbf{0.0210}) \ < \ \alpha \ (\mathbf{0.05}) \ \le \ \text{P-value}_{\text{White}} \ (\mathbf{0.3100})$$

Ce test démontre que le conservatisme de White peut paralyser la recherche quantitative en générant des **faux négatifs** (rejeter une stratégie valide sous prétexte qu'elle a été découverte au milieu d'un grand nombre de mauvais modèles) [🌐-0].

En neutralisant l'influence des algorithmes non pertinents via son filtre de recentrage, Hansen élimine le bruit de fond de la sur-optimisation sans altérer la rigueur du contrôle [🌐-0]. Le test SPA prouve que la stratégie à $+0.05\%$ possède un authentique pouvoir prédictif : sa performance survit au biais du Data Mining une fois le catalogue expurgé de ses déchets statistiques, offrant aux institutions financières un outil de validation d'une précision absolue [🌐-0].
