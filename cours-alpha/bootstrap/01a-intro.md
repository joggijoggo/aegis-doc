# Cours : Le Bootstrap Stationnaire (Politis & Romano)

---

## 📌 Introduction Générale

Dans l'analyse statistique moderne, la simulation de données est un outil incontournable. Elle permet d'estimer la précision d'un paramètre, de construire des intervalles de confiance ou de tester des hypothèses complexes sans avoir à postuler des lois de probabilité théoriques souvent trop rigides (comme la loi normale). La méthode reine pour réaliser ces simulations est le **bootstrap**.

Idéalement conçu pour des données indépendantes, le bootstrap a dû évoluer pour s'attaquer aux données temporelles économiques et financières. Ces dernières possèdent une structure complexe : elles sont dépendantes du temps et dynamiques. Le **Bootstrap Stationnaire**, introduit par Dimitris Politis et Joseph Romano en 1994, s'est imposé comme l'une des réponses mathématiques les plus robustes pour simuler des séries temporelles tout en respectant scrupuleusement leurs propriétés d'origine.

L'objectif de ce cours est de décomposer méthodiquement cette technique, de comprendre ses fondements géométriques, ses réglages mathématiques rigoureux et ses applications concrètes contre les pièges de la finance quantitative.

---

## 📘 Partie 1 : Le problème fondamental des données temporelles

### 1. Le postulat des données i.i.d. et les statistiques classiques
La majorité des tests statistiques traditionnels reposent sur une hypothèse fondamentale : les observations de l'échantillon doivent être **Indépendantes et Identiquement Distribuées (i.i.d.)**.
* **Indépendantes :** Le résultat d'une observation n'influence en rien le résultat des observations suivantes.
* **Identiquement distribuées :** Toutes les observations proviennent de la même loi de probabilité (elles partagent la même moyenne, la même variance, etc.).

L'analogie classique est le lancer d'un dé équilibré. Obtenir un $6$ au premier lancer ne modifie pas la probabilité d'obtenir un $6$ au second lancer. Chaque lancer est indépendant et suit la même loi uniforme.

### 2. La réalité des marchés financiers : la mémoire temporelle
En finance, en macroéconomie ou en météorologie, l'hypothèse d'indépendance est systématiquement violée. Les rendements d'un actif financier, les taux d'inflation ou les températures journalières affichent ce que l'on appelle une **dépendance temporelle** ou de la **mémoire**.

Cette mémoire se manifeste sous deux formes principales :
1. **L'autocorrélation :** La valeur d'aujourd'hui est linéairement liée à la valeur d'hier. Par exemple, si le PIB augmente fortement ce trimestre, il y a une forte probabilité qu'il continue de croître le trimestre suivant en raison de l'inertie économique.
2. **Les grappes de volatilité (*Volatility Clustering*) :** Les vagues de forte variabilité s'enchaînent. Les marchés financiers alternent entre de longues périodes de calme et des phases de stress intense (crises) où les fortes variations quotidiennes (positives ou négatives) se succèdent.

### 3. Pourquoi le bootstrap classique (Efron, 1979) échoue
Le bootstrap original, inventé par Bradley Efron, fonctionne par tirage aléatoire individuel avec remise. L'algorithme prend les $T$ observations de départ, les place dans une urne virtuelle, et procède à $T$ tirages successifs. À chaque tirage, la donnée sélectionnée est recopiée dans le nouvel échantillon, puis remise dans l'urne.

Si les données d'origine possèdent de la mémoire, le bootstrap classique commet une erreur grave : il détruit l'ordre chronologique et, par conséquent, efface toute la structure de dépendance temporelle.

#### 🔢 Application numérique de l'échec d'Efron
Imaginons que nous suivons le rendement d'une action sur un horizon de $T = 5$ jours. La série originale montre une tendance claire (les deux premiers jours sont hauts, suivis d'un effondrement les jours 3 et 4, puis d'un début de rebond le jour 5) :

$$X = [t_1: +2\%, \quad t_2: +3\%, \quad t_3: -4\%, \quad t_4: -3\%, \quad t_5: +1\%]$$

L'ordre chronologique $[t_1, t_2, t_3, t_4, t_5]$ témoigne d'une mémoire de court terme (le choc négatif du jour 3 se prolonge le jour 4).

Appliquons le bootstrap classique d'Efron. L'ordinateur effectue 5 tirages indépendants avec remise dans notre urne contenant les 5 valeurs. Supposons que le sort détermine la suite d'indices temporels suivante : $[t_3, t_1, t_4, t_1, t_5]$.

La nouvelle série simulée (échantillon bootstrap) est :

$$X_{\text{Efron}} = [-4\%, \quad +2\%, \quad -3\%, \quad +2\%, \quad +1\%]$$

**Analyse de l'échec :**
* Dans la réalité ($X$), la forte baisse de $-4\%$ is immédiatement suivie d'une autre baisse de $-3\%$. Le marché a mis du temps à absorber le choc.
* Dans la simulation ($X_{\text{Efron}}$), le rendement de $-4\%$ est immédiatement suivi d'une hausse de $+2\%$. La dynamique de persistance de la baisse a été totalement annihilée.

Si vous utilisez $X_{\text{Efron}}$ pour calculer le risque de perte consécutive de votre portefeuille, votre modèle conclura à tort que le marché se retourne instantanément après un krach, sous-estimant ainsi dramatiquement le risque réel. Pour préserver cette structure, il est indispensable de ne plus piger les données point par point, mais par morceaux.

---

### Synthèse de la Partie 1
* **Le constat :** Les séries temporelles financières possèdent une mémoire (dépendance) qui viole l'hypothèse i.i.d.
* **Le problème :** Le bootstrap classique détruit la chronologie en mélangeant les observations de manière isolée.
* **L'impératif :** Concevoir une méthode de rééchantillonnage par blocs de données consécutives.
