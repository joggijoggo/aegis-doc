# RAPPORT TECHNIQUE 3 : SPÉCIFICATIONS TECHNIQUES ET ALGORITHMIQUES DU PROJET 2 – LE CONSTRUCTEUR DE DATASET (BAR ENGINE)

---

## PARTIE 1 : CADRE ENCADRANT, SYNTHÈSE MANAGÉRIALE ET SCHÉMA DE DONNÉES DE FLUX

**Statut du Document :** Spécifications d'Ingénierie de la Structure des Données (In-Sample 2019-2022)  
**Composant Logiciel :** Projet 2 (Couche Intermédiaire / Moteur de Segmentation et d'Agrégation)  
**Données d'Entrée :** Flux de Ticks Virtuels Ajustés (Schéma Parquet journalier avec volume fractionnaire réel provenant du Projet 1)  

---

## Synthèse Managériale

Le Projet 2 (Bar Engine) opère comme le compresseur microstructurel passif de notre écosystème quantitatif. Placé sous la dépendance stricte du Projet 1, sa mission unique est de regrouper le flux continu de ticks virtuels ajustés sous forme de barres structurelles discrètes. Pour garantir une neutralité scientifique absolue et éliminer tout risque de biais de surapprentissage à ce niveau, le Projet 2 est épuré de tout lissage ou estimateur glissant (pas de calcul d'ATR, pas de moyennes mobiles glissantes, pas d'attente statistique d'information).

Le module reçoit une configuration de paramètres fixes et explicites (ex: un volume fractionnaire fixe, un range fixe ou un seuil d'imbalance fixe) et applique des règles de clôture strictes et asynchrones. De plus, il résout le problème du dépassement de seuil (overshooting) par une approche d'absorption complète et de réinitialisation à zéro, préservant ainsi l'intégrité économique des transactions institutionnelles. Le livrable final est une matrice de barres brutes augmentées à 18 dimensions, constituant une base de données "totale" et non dégradée que le Projet 3 pourra analyser et transformer.

---

## 1. Introduction

### 1.1. Contexte
Dans le domaine du trading quantitatif et de la microstructure des marchés, la transition entre un flux de ticks bruts et une série temporelle exploitable nécessite une étape d'échantillonnage (sampling). Alors que les plateformes de détail se limitent à agréger les données selon des fenêtres de temps rigides, l'ingénierie moderne exige de pouvoir segmenter le marché selon des critères liés à l'activité réelle, à la géométrie du prix ou au flux d'information, afin de stabiliser les propriétés statistiques des rendements.

### 1.2. Motivation
La motivation principale du Projet 2 est de fournir un moteur de calcul ultra-rapide capable de segmenter le marché de manière pure. En limitant le périmètre de ce module à la simple compression microstructurelle (Double OHLCV + Métriques de friction + Volumes fractionnaires), nous assurons une performance algorithmique optimale lors du balayage de la grille de paramètres. Cela préserve également la totale liberté du chercheur qui pourra concevoir, modifier ou tester n'importe quelle transformation mathématique en aval (Projet 3) sans jamais avoir à réécrire la logique de segmentation des ticks.

### 1.3. Problématique
La problématique technique du Projet 2 se définit ainsi :
> **« Comment concevoir un moteur d'échantillonnage asynchrone purement passif et paramétrable, capable de compresser un flux de ticks vectorisés sous forme de matrice multi-dimensionnelle sans destruction d'information sur la liquidité (Dual-Pricing) ni sur la masse des volumes réels, et sans introduire de corrélations sérielles artificielles lors des dépassements de seuils ? »**

---

## 2. Intégration du Flux et Respect du Schéma Parquet

Le Projet 2 consomme en continu les données issues de la virtualisation en RAM du Projet 1. Le flux respecte le schéma strict de vos fichiers Parquet quotidiens, garantissant une compatibilité descendante totale :
* `timestamp` : datetime64[ns] (Axe temporel fictif, monotone et strictement croissant)
* `bidprice` / `askprice` / `price_mid` : float64 (Cours translatés récursivement par l'ajustement cumulé du Projet 1)
* `spread` : float64 (Écart absolu askprice - bidprice original et invariant)
* `bidvolume` / `askvolume` : int32 (Volumes d'achat et de vente historiques originaux)
* `volume` : float64 (Volume de transaction réel exprimé en fraction de millions d'unités)

---

## PARTIE 2 : LES TROIS BRANCHES DE LA TAXONOMIE MÉCANIQUE ET GESTION DE L'OVERSHOOTING

---

## 3. Les Algorithmes de Construction des Barres (Bar Builders)

Pour chaque Univers Alternatif, le Projet 2 initialise une structure de barre au tick t₀. Il accumule les ticks de manière vectorisée jusqu'à ce que la condition de clôture soit validée au tick t_clôture. La grille de paramètres testée définit des seuils fixes et explicites répartis en trois familles :

### 3.1. La Famille de l'Activité (Seuils Métriques Fixes)
*   **Time Bars (Temporelles) :** Clôture de la barre dès que la colonne `timestamp` franchit le pas de temps défini par le paramètre P_time (ex: 60s, 300s, 3600s).
*   **Tick Bars :** Clôture dès que le nombre de lignes de ticks accumulées (le décompte physique des lignes reçues) atteint le seuil fixe N.
*   **Volume Bars (Broker Activity) :** Clôture dès que la somme accumulée de la colonne volume réel fractionnaire (\sum volume_t) franchit le seuil fixe V.
*   **Dollar / Notional Bars :** Clôture de la barre dès que la somme de la valeur notionnelle brute (\sum price_mid_t) dépasse le seuil fixe D. L'effet d'échelle de la constante de lot standard est directement absorbé par la cible D.
*   **Dollar Volume Bars :** Clôture de la barre dès que la somme cumulée du prix pondéré par le volume fractionnaire brut (\sum price_mid_t * volume_t) dépasse le seuil fixe DV. Les constantes d'échelle (fraction en millions et taille de lot) sont intégralement absorbées par la valeur de la cible DV.

### 3.2. La Famille de la Géométrie (Mouvement Pur du Prix)
*   **Range Bars :** Une barre reste ouverte tant que l'amplitude extrême du prix moyen n'a pas franchi le seuil de pips fixe R. La condition de clôture est :
    \max(price_mid) - \min(price_mid) >= R
*   **Renko Bars :** Une brique est générée uniquement si le `price_mid` actuel s'écarte du prix de clôture de la brique précédente (`close_mid`) d'un montant fixe de pips B dans une direction spécifique.

### 3.3. La Famille de l'Information (Déséquilibre du Flux Réel)
*   **Imbalance Bars (Tick / Volume / Dollar Volume) Brutes :** Le module calcule la règle du signe du tick (b_t = +1 si Delta price_mid > 0, -1 si Delta price_mid < 0, et b_t-1 si inchangé). La barre se clôture dès que la valeur absolue de la somme cumulée de ces signes, pondérée selon la configuration (soit par 1 pour le Tick, soit par la colonne `volume` pour le Volume, soit par la valeur brute `price_mid * volume` pour le Dollar Volume), franchit le seuil fixe d'information brute I.
*   **Run Bars (Tick / Volume / Dollar Volume) Brutes :** La barre se clôture dès que l'accumulation associée à une séquence directionnelle continue et unilatérale de signes identiques (+1 consécutifs ou -1 consécutifs) franchit le seuil fixe K.

---

## 4. Algorithme de Gestion de l'Overshooting Microstructurel

Lorsque la granularité d'un tick individuel apporte une quantité d'activité ou d'information supérieure au seuil restant pour fermer la barre, le Projet 2 applique l'Algorithme d'Absorption Complète et de Réinitialisation.

1.  **Interdiction de Fractionnement :** Le Projet 2 s'interdit de scinder le tick ou de reporter le surplus dans un buffer pour la barre suivante. Diviser un ordre bloc institutionnel détruirait la réalité économique de la transaction et introduirait une corrélation sérielle artificielle (barres à rendement nul consécutives).
2.  **Absorption Totale :** Tout tick provoquant un dépassement de la valeur cible est intégralement absorbé par la barre courante. Le volume ou le déséquilibre réel de la barre émise sera donc mathématiquement supérieur ou égal au seuil théorique de la configuration.
3.  **Réinitialisation Synchrone :** À l'instant précis du déclenchement, la barre est scellée et émise. Le compteur d'accumulation de la barre suivante redémarre à une valeur strictement égale à zéro. Aucun mécanisme de buffer n'est toléré.

--- 

## PARTIE 3 : FORMAT COMPACT À 18 DIMENSIONS CONTRACTUEL ET EXAMEN DES PERSPECTIVES

---

## 5. Format de Sortie Contractuel (Livrable P2 ➔ P3 : Matrice Totale)

Chaque fois qu'une condition de clôture est validée, le Projet 2 compile les informations du segment de ticks écoulé et écrit une ligne de données standardisée. L'information est structurée selon le schéma exhaustif à 18 dimensions suivant :

*   `virtual_timestamp` : datetime64[ns] (Horodatage exact du tick de clôture)
*   `open_mid` / `high_mid` / `low_mid` / `close_mid` : float64 (Géométrie du prix moyen)
*   `open_bid` / `high_bid` / `low_bid` / `close_bid` : float64 (Géométrie de la barrière acheteuse)
*   `open_ask` / `high_ask` / `low_ask` / `close_ask` : float64 (Géométrie de la barrière vendeuse)
*   `min_spread` / `max_spread` : float64 (Bornes absolues de friction)
*   `avg_spread` : float64 (Métrique de densité et de durée du risque de liquidité)
*   `tick_count` : int32 (Nombre de lignes de ticks accumulées dans la barre)
*   `total_volume` : float64 (Somme cumulative de la colonne fractionnaire 'volume' dans la barre)
*   `total_bidvolume` : int64 (Somme des 'bidvolume' accumulés dans la barre)
*   `total_askvolume` : int64 (Somme des 'askvolume' accumulés dans la barre)

---

## 6. Pistes Futures

*   **Vectorisation SIMD Compilée (C++ / Rust Extension) :** Implémenter le moteur d'accumulation des barres sous forme d'extension binaire compilée pour Python, exploitant les instructions vectorielles des processeurs (AVX-512) afin de maximiser la vitesse de traitement du flux de ticks.
*   **Analyse de l'Overshooting Volumétrique :** Étudier au sein du Projet 3 si l'écart entre le volume réel d'une barre après absorption et son seuil théorique constitue un descripteur d'agressivité institutionnelle exploitable par les modèles.

---

## 7. Conclusion

Le Projet 2 remplit sa mission technique en convertissant le flux continu de ticks ajustés en une série discrète et structurée de barres à haute densité d'information. En se cantonnant à un rôle d'exécuteur paramétrique pur et passif, débarrassé de toute logique temporelle glissante ou d'analyse statistique, il garantit la stabilité et la rapidité du pipeline de données. Le dataset émis offre une mine d'or microstructurelle non dégradée pour le Projet 3, qui pourra débuter l'évaluation scientifique du *Feature Space*.

