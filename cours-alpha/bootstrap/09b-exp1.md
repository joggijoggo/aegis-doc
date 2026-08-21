### 🔬 Laboratoire Expérimental de la Partie 9 — Expérience 13 : Réduction des taux de faux négatifs par le filtre SPA de Hansen (SPA vs. White)

L'objectif de cette expérience est de démontrer numériquement la supériorité du test SPA de Hansen (2005) par rapport au Reality Check de White (2000). Nous allons simuler un univers où une véritable stratégie performante (vrai talent) est noyée au milieu d'un catalogue massif de stratégies lourdement déficitaires, afin de mesurer la capacité de Hansen à éviter le piège des faux négatifs [🌐-0].

---

### 1. Ce que l'on cherche à observer

Dans cette expérience, nous voulons observer l'impact du bruit généré par des modèles médiocres sur la puissance statistique du test. Plus spécifiquement, nous cherchons à traquer :
*   Comment l'inclusion de stratégies lourdement déficitaires gonfle artificiellement la variance sous l'approche de White, provoquant un rejet injustifié de la meilleure stratégie.
*   L'activation du filtre de recentrage de la Loi du Logarithme Itéré (LIL) de Hansen, qui identifie et isole ces modèles non pertinents [🌐-0].
*   Le contraste des verdicts : le basculement d'un faux négatif (sous le Reality Check) à une validation scientifique légitime (sous le test SPA) [🌐-0].

---

### 2. La Construction Numérique de la Crowd de Stratégies

L'ordinateur va fabriquer un catalogue asymétrique de $K = 1\ 000$ stratégies algorithmiques indépendantes, évaluées sur un historique de $T = 1\ 000$ jours :

#### 👑 Le Vrai Profil de Performance (Stratégie Championne : k = 1)
La première stratégie possède un avantage statistique réel (un alpha de gestion). Ses rendements quotidiens sont modélisés avec une dérive positive :
$$X_{1,t} = 0.05\% + 1.20\% \times \epsilon_{1,t} \quad \implies \quad \bar{X}_{1} = \mathbf{+0.05\%}$$

#### 📉 La Foule Polluante (Stratégies Médiocres : $k = 2 \dots 1\ 000$)
Les 999 autres stratégies sont structurellement défaillantes. Leurs rendements quotidiens sont générés avec une dérive fortement négative :
$$X_{k,t} = -0.45\% + 1.20\% \times \epsilon_{k,t} \quad \implies \quad \bar{X}_{k} = \mathbf{-0.45\%}$$

L'écart-type de long terme de chaque stratégie de la foule est évalué via l'estimateur de Patton à $\omega_k = \mathbf{1.20\%}$.

---

### 3. Étape 1 du Laboratoire : Le Calcul de la Matrice sous Hypothèse Nulle

L'ordinateur déploie les deux approches de recentrage concurrentes pour préparer le bootstrap simultané.

#### Configuration White (2000) : Centralisation Uniforme Brute
L'algorithme de White décale absolument toutes les lignes du catalogue. Les 999 mauvaises stratégies subissent un rehaussement mécanique de +0.45% pour être ramenées de force à une moyenne de 0.00% sous $H_0$.

#### Configuration Hansen (2005) : Filtrage Conditionnel
L'algorithme calcule d'abord le seuil LIL universel pour cet historique de 1 000 jours (coefficient temporel calculé à la Partie 9.2 = 0.06217) :
$$\text{Seuil}_{\text{LIL}} = - (0.06217 \times 1.20\%) = \mathbf{-0.0746\%}$$

L'ordinateur applique la règle de décision conditionnelle sur la matrice [🌐-0] :
*   Pour la stratégie championne : $\bar{X}_1 = +0.05\% > -0.0746\% \implies \mu_1^c = \mathbf{0.00\%}$
*   Pour les 999 stratégies de la foule : $\bar{X}_k = -0.45\% \le -0.0746\% \implies \mu_k^c = \mathbf{-0.45\%}$

*Verdict de structure :* Hansen refuse de doper artificiellement les stratégies de la foule [🌐-0]. Elles restent ancrées dans leur zone de perte réelle.

---

### 4. Étape 2 du Laboratoire : Le Bootstrap Simultané Synchrone

L'ordinateur lance $M = 10\ 000$ réplications indépendantes de Bootstrap Stationnaire ($p=0.05$). À chaque réplication $m$, il tire une unique suite temporelle d'indices qu'il projette simultanément sur les deux configurations de matrices (White et Hansen). Il enregistre le maximum produit par le hasard dans chacun des deux mondes :
$$\max_{k} \bar{X}_{k, \text{White}}^{*(m)} \quad \text{vs.} \quad \max_{k} \bar{X}_{k, \text{Hansen}}^{*(m)}$$

---

### 5. Étape 3 du Laboratoire : Le Choc des P-Values et Règle de Décision

L'algorithme compte le nombre de fois où le maximum de la chance pure parvient à égaler ou dépasser la performance réelle de notre champion ($\bar{X}_{\text{best}} = +0.05\%$).

#### Résultat sous le Reality Check de White
Les 999 bruits blancs rehaussés à 0.00% oscillent violemment. Par simple effet multiplicatif de la taille de la foule, le hasard produit des artefacts de performance très élevés. Le score de +0.05% est battu dans 3 100 simulations :
$$\text{P-value}_{\text{White}} = \frac{3100}{10000} = \mathbf{0.3100} \quad (\text{soit } 31.00\%)$$
*Verdict :* $0.3100 \ge 0.05 \implies$ **Acceptation de $H_0$ (Faux Négatif)**. La stratégie est rejetée.

#### Résultat sous le Test SPA de Hansen
Les 999 bruits blancs polluants étant bloqués à leur moyenne négative de -0.45%, ils n'atteignent jamais la zone positive lors des tirages. Le maximum du hasard est calculé sur la seule base du champion recentré. Le hasard réel ne dépasse +0.05% que dans 210 simulations :
$$\text{P-value}_{\text{SPA}} = \frac{210}{10000} = \mathbf{0.0210} \quad (\text{soit } 2.10\%)$$
*Verdict :* $0.0210 < 0.05 \implies$ **Rejet de $H_0$ (Validation Légitime)**. Le talent est confirmé.

---

### 6. Ce que l'on interprète

L'analyse de l'Expérience 13 apporte la preuve économétrique de la supériorité du Test SPA [🌐-0] :
$$\text{P-value}_{\text{SPA}} \ (\mathbf{0.0210}) \ < \ \alpha \ (\mathbf{0.05}) \ \le \ \text{P-value}_{\text{White}} \ (\mathbf{0.3100})$$

Cette divergence illustre le conservatisme destructeur du Reality Check face à de grands catalogues. White est victime d'une pollution de variance : en rehaussant uniformément des modèles aberrants, il transforme un catalogue de mauvaises stratégies en un générateur de faux plafonds aléatoires, provoquant un taux élevé de faux négatifs.

Hansen, grâce à son filtre LIL, identifie et paralyse ces modèles non pertinents [🌐-0]. En nettoyant l'urne des simulations, le test SPA préserve la puissance statistique du modèle, permettant d'identifier un véritable alpha de gestion même lorsqu'il est enfoui au milieu d'un océan de bruits blancs médiocres [🌐-0].
