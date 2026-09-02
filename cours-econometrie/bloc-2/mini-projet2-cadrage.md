# Mini-Projet 2 : Le Filtre à Alpha Multi-Indicateurs

### 🎯 Objectif du Projet
L'objectif de ce livrable de synthèse est d'assembler l'intégralité des briques mathématiques, géométriques et stochastiques développées tout au long du **Bloc 2**. Vous devez construire un script d'ingénierie quantitative complet, modulaire et industrialisé. Ce programme chargera un échantillon de marché réel, calibrera un modèle d'évaluation multi-indices, exécutera la purge géométrique pour isoler l'Alpha résiduel d'une devise, et appliquera le pare-feu d'inférence pour valider ou rejeter les signaux d'arbitrage.

---

### 📋 Spécifications Techniques & Cahier des Charges

Le projet doit être structuré autour d'un pipeline d'exécution strict divisé en quatre phases logiques. Tout le code (fichiers, classes, fonctions, documentations et commentaires) doit être rédigé **exclusivement en anglais**.

#### Phase 1 : Data Alignment & Matrix Specification
Le programme doit charger un fichier de données contenant l'historique de clôture d'un actif cible $Y$ (ex: `EURUSD`) et de $K-1$ indices macroéconomiques mondiaux (ex: `DXY`, `SPY`, `US10Y`).
- Implémenter une routine de synchronisation temporelle et de gestion des données manquantes.
- Construire la matrice de design globale $X$ en y intégrant dynamiquement une colonne d'unités (vecteur de $1$) pour modéliser l'intercept (Alpha structurel).

#### Phase 2 : The Geometric Washing Machine
Développer un moteur d'algèbre linéaire brute exploitant uniquement les opérations primitives de `numpy`.
- Calculer le vecteur des coefficients MCO $\hat{\beta} = (X^T X)^{-1} X^T Y$.
- Générer de manière analytique la matrice chapeau $H = X(X^T X)^{-1}X^T$ et la matrice résiduelle $M = I - H$.
- Extraire le vecteur d'Alpha résiduel purement in-sample $\hat{\epsilon} = M Y$.
- Implémenter un test unitaire interne vérifiant la condition de purge stricte : $\|X^T \hat{\epsilon}\| \le 10^{-10}$ (perpendicularité géométrique).

#### Phase 3 : The Stochastic Firewall
Développer une classe de validation statistique exploitant les distributions théoriques de `scipy.stats`.
- Calculer l'estimateur sans biais de la variance résiduelle $s^2 = \text{SCR}/(N-K)$.
- Déduire la matrice de variance-covariance estimée $\hat{\mathbb{V}}(\hat{\beta}) = s^2(X^T X)^{-1}$ et extraire les erreurs-types individuelles.
- Calculer les statistiques de Student ($t$-stat) et les deux queues de $p$-values associées pour chaque coefficient.
- Calculer la statistique de Fisher globale ($F$-stat) et sa $p$-value conjointe en exploitant la décomposition de la variance dictée par le théorème de Cochran ($\text{SCT} = \text{SCE} + \text{SCR}$).
- Calculer le coefficient de détermination ajusté $\bar{R}^2$ pour quantifier la parcimonie du modèle.

#### Phase 4 : Systematic Routing & Risk Management
Traduire les métriques d'inférence en règles d'engagement de capital.
- **Élagage dynamique (Feature Selection) :** Le script doit automatiquement identifier et exclure du modèle final les indices macroéconomiques dont la $p$-value individuelle est supérieure à $5\%$ ($p > 0.05$).
- **Coupe-circuit central (Kill Switch) :** Si la $p$-value du test de Fisher global est supérieure à $5\%$, le script doit lever une alerte de sécurité, suspendre l'exécution et interdire toute transaction.
- **Position Sizing :** Si le modèle est validé par Fisher, le script doit calculer un Z-score instantané sur le dernier résidu $\hat{\epsilon}_t$ et moduler la taille de l'ordre inversement proportionnellement à la variance résiduelle sans biais $s^2$.

---

### 📦 Livrables Attendus

Pour valider le projet, vous devez restituer deux fichiers distincts :

1. **`alpha_filter_engine.py` :** Le script Python de production contenant les classes d'isolation et de validation statistique. Ce fichier doit intégrer une section de test (`if __name__ == "__main__":`) déroulant le pipeline complet sur un jeu de données simulé ou réel, affichant un rapport d'inférence clair dans le terminal.
2. **`validation_report.md` :** Une note de synthèse rédigée en français analysant les résultats du script. Vous devrez y expliquer si votre modèle multi-indices est robuste, quelles variables ont été purgées par le pare-feu de Student, et comment votre robot s'est comporté face au risque de surapprentissage.
