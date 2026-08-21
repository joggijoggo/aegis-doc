### 🔬 Laboratoire Expérimental de la Partie 4 — Expérience 9 : L'impact du facteur d'échelle temporel (La surveillance de l'échantillon $T$)

L'objectif de cette expérience est d'isoler et de quantifier le rôle du facteur d'échelle asymptotique ($T^{-1/3}$) combiné au durcissement logarithmique du seuil de Politis-White. Nous allons Démontrer comment l'ordinateur réagit lorsque la quantité d'information historique augmente sur un marché ayant une structure de mémoire identique, illustrant ainsi la convergence asymptotique de l'algorithme.

---

### 1. Ce que l'on cherche à observer

Dans cette expérience, nous cherchons à observer comment l'ordinateur exploite l'augmentation de la taille de la base de données ($T$).

Plus spécifiquement, nous voulons traquer l'évolution de deux barrières mathématiques :
*   Le resserrement du filtre de bruit ($\text{Seuil}_{\text{PW}}$) : devient-il plus discriminant à mesure que $T$ grandit, permettant de détecter des signaux de mémoire auparavant masqués par le hasard ?
*   L'ajustement du facteur d'échelle ($T^{-1/3}$) : comment cette force mathématique allonge-t-elle la taille moyenne des blocs $\mathbb{E}(L)$ pour capturer les queues de mémoire lointaines sans détruire la diversité des simulations ?

---

### 2. La Construction Méthodologique et Numérique des Échantillons

Pour cette expérience, nous allons figer la structure mathématique sous-jacente du marché. Nous utilisons un processus autorégressif AR(1) identique à celui de la Partie 4.2 ($\rho_1 = 0.40, \rho_2 = 0.25, \rho_3 = 0.18, \rho_4 = 0.08$, et les jours suivants oscillent près de zéro). La variance historique de la volatilité est fixée dans les deux cas à $\sigma_Y^2 = 1.50$.

Nous allons soumettre cette structure à deux échelles d'historique différentes :

#### 📊 Échantillon 1 : L'Échelle Courte (Petite base de données)
Ce vecteur représente une base de données restreinte, par exemple un jeune actif financier ou un historique de court terme.
*   **Taille de l'échantillon :** $T_{\text{court}} = \mathbf{500 \text{ jours}}$

#### 📊 Échantillon 2 : L'Échelle Longue (Grande base de données)
Ce vecteur simule un historique profond, typique d'un indice boursier majeur suivi sur plusieurs décennies.
*   **Taille de l'échantillon :** $T_{\text{long}} = \mathbf{8\ 000 \text{ jours}}$

---

### 3. Application Numérique Pas à Pas sur l'Échelle Courte ($T = 500$)

L'ordinateur applique l'algorithme de Politis-White sur la base de données réduite :

#### Étape A : Calcul du seuil et détection du point de rupture $k^*$
L'ordinateur calibre sa ligne de tolérance au bruit pour 500 jours :
$$\text{Seuil}_{\text{PW, court}} = 2 \times \sqrt{\frac{\ln(500)}{500}} = 2 \times \sqrt{\frac{6.2146}{500}} = 2 \times \sqrt{0.01243} \approx \mathbf{0.2229}$$

L'algorithme scanne les autocorrélations réelles face à ce seuil large :
*   $k=1 \implies \rho_1 = 0.40$ (Supérieur à $0.2229 \rightarrow$ Signal)
*   $k=2 \implies \rho_2 = 0.25$ (Supérieur à $0.2229 \rightarrow$ Signal)
*   $k=3 \implies \rho_3 = 0.18$ (Inférieur à $0.2229 \rightarrow$ **Coupure immédiate !**)

Le seuil étant trop haut en raison du manque de données, le signal du jour 3 est rejeté comme du bruit. L'algorithme enregistre : $\mathbf{k^* = 3}$.

#### Étape B : Calcul de la fenêtre globale $M$
L'ordinateur applique le multiplicateur de sécurité :
$$M = 2 \times 3 = \mathbf{6 \text{ jours}}$$

#### Étape C : Calcul de la force de la mémoire cumulée $G_{\text{PW}}$
L'ordinateur calcule la somme pondérée tronquée sur 6 jours :
$$\sum_{k=1}^{6} k \cdot \rho_k = (1 \times 0.40) + (2 \times 0.25) + 0 + 0 + 0 + 0 = 0.40 + 0.50 = 0.90$$
$$G_{\text{PW}} = 2 \times 0.90 = \mathbf{1.80} \implies G_{\text{PW}}^2 = \mathbf{3.24}$$

#### Étape D : Extraction cubique de $p_{\text{optimal}}$
L'ordinateur calcule le facteur d'échelle $500^{-1/3} \approx 0.1260$ et résout l'équation :
$$p_{\text{optimal}} = \left( \frac{1.50}{3.24} \right)^{1/3} \times 0.1260 = (0.4629)^{1/3} \times 0.1260 \approx 0.7735 \times 0.1260 = \mathbf{0.0975}$$

---

### 4. Application Numérique Pas à Pas sur l'Échelle Longue ($T = 8\ 000$)

L'ordinateur déploie le même algorithme sur la base de données profonde :

#### Étape A : Calcul du seuil et détection du point de rupture $k^*$
L'ordinateur recalibre sa ligne de tolérance au bruit pour 8 000 jours :
$$\text{Seuil}_{\text{PW, long}} = 2 \times \sqrt{\frac{\ln(8000)}{8000}} = 2 \times \sqrt{\frac{8.9872}{8000}} = 2 \times \sqrt{0.001123} \approx \mathbf{0.0670}$$

Disposant de beaucoup plus d'informations, le bruit de fond statistique diminue. L'algorithme scanne à nouveau la même structure de marché face à ce filtre resserré :
*   $k=1 \implies \rho_1 = 0.40$ (Supérieur à $0.0670 \rightarrow$ Signal)
*   $k=2 \implies \rho_2 = 0.25$ (Supérieur à $0.0670 \rightarrow$ Signal)
*   $k=3 \implies \rho_3 = 0.18$ (Supérieur à $0.0670 \rightarrow$ Signal)
*   $k=4 \implies \rho_4 = 0.08$ (Supérieur à $0.0670 \rightarrow$ Signal)
*   $k=5 \implies \rho_5 = 0.03$ (Inférieur à $0.0670 \rightarrow$ **Coupure actée !**)

Les jours 3 et 4 sont désormais identifiés comme du signal légitime. L'algorithme enregistre : $\mathbf{k^* = 5}$.

#### Étape B : Calcul de la fenêtre globale $M$
$$M = 2 \times 5 = \mathbf{10 \text{ jours}}$$

#### Étape C : Calcul de la force de la mémoire cumulée $G_{\text{PW}}$
L'ordinateur calcule la somme pondérée étendue sur 10 jours :
$$\sum_{k=1}^{10} k \cdot \rho_k = (1 \times 0.40) + (2 \times 0.25) + (3 \times 0.18) + (4 \times 0.08) + 0 \dots = 0.40 + 0.50 + 0.54 + 0.32 = 1.76$$
$$G_{\text{PW}} = 2 \times 1.76 = \mathbf{3.52} \implies G_{\text{PW}}^2 = \mathbf{12.3904}$$

#### Étape D : Extraction cubique de $p_{\text{optimal}}$
L'ordinateur calcule le nouveau facteur d'échelle $8000^{-1/3} = 0.0500$ et résout :
$$p_{\text{optimal}} = \left( \frac{1.50}{12.3904} \right)^{1/3} \times 0.0500 = (0.12106)^{1/3} \times 0.0500 \approx 0.4947 \times 0.0500 = \mathbf{0.0247}$$

---

### 5. Ce que l'on interprète

L'analyse de ce saut d'échelle révèle la puissance de la convergence asymptotique de l'algorithme :

*   **Interprétation sur l'historique court ($T=500$) :** La probabilité de coupure s'établit à **0.0975**, ce qui restreint la longueur moyenne des blocs à $\mathbb{E}(L) = \frac{1}{0.0975} \approx \mathbf{10.25 \text{ jours}}$. Le manque de données a forcé le filtre à être très large ($0.2229$), ce qui a tronqué prématurément l'analyse de la mémoire. L'ordinateur a choisi d'utiliser des blocs courts pour s'assurer que le bootstrap puisse mélanger suffisamment de cartes et préserver la diversité (variance) des simulations, quitte à subir un léger biais.
*   **Interprétation sur l'historique long ($T=8\ 000$) :** La probabilité de coupure chute à **0.0247**, ce qui permet d'allonger la longueur moyenne des blocs à $\mathbb{E}(L) = \frac{1}{0.0247} \approx \mathbf{40.5 \text{ jours}}$. Disposant d'un catalogue massif de 8 000 jours, le bruit statistique s'effondre. L'algorithme resserre ses critères ($0.0670$), capture des nuances de mémoire plus fines et déploie de très grands blocs (40 jours) pour restituer parfaitement la dynamique de long terme de la série, sans jamais craindre d'étouffer la diversité du bootstrap car le stock de données initial est gigantesque.

**Conclusion du Laboratoire :** L'expérience démontre que l'algorithme de Politis & White est mathématiquement programmé pour être de plus en plus performant et précis à mesure que l'historique grandit, optimisant de manière dynamique le compromis entre la fidélité temporelle et la richesse statistique.
