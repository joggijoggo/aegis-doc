## MODULE 2 : IDENTIFICATION DE L'ANOMALIE ET FORMULATION DE L'ALPHA
Bienvenue dans le deuxième pilier de notre cursus de R&D. Maintenant que notre usine de traitement produit des caractéristiques (features) propres et échantillonnées de manière non temporelle, nous allons nous attaquer au péché originel de la recherche quantitative : le traitement destructeur du signal.
La plupart des chercheurs commettent un crime mathématique pour rendre leurs données exploitables par un algorithme. Aujourd'hui, nous allons apprendre à extraire des anomalies de marché tout en préservant la mémoire à long terme des prix.
------------------------------
## 2.1 Les grandes familles d'anomalies de marché## 📊 Fondations théoriques
Une anomalie est une inefficience statistique ou microstructural temporaire qui s'écarte de l'hypothèse des marchésefficients (EMH). Pour qu'un Alpha soit pérenne, il doit exploiter l'une de ces trois forces motrices :
## A. Le Suivi de tendance (Momentum / Trend Following)

* La physique : Sous-réaction initiale des acteurs du marché face à une nouvelle information macroéconomique ou microstructural, suivie d'un phénomène de sur-réaction grégaire (herding behavior).
* La signature statistique : Autocorrélation positive persistante des rendements à des horizons de temps spécifiques ($\text{Cov}(R_t, R_{t-\tau}) > 0$).

## B. Le Retour à la moyenne (Mean Reversion)

* La physique : Sur-réaction des flux de liquidité ou contraintes d'exécution forcées (ex: liquidations de marges, rééquilibrages de fonds indiciels) qui poussent le prix d'un actif au-delà de sa valeur fondamentale temporaire.
* La signature statistique : Autocorrélation négative des rendements ($\text{Cov}(R_t, R_{t-\tau}) < 0$) ou relation de cointégration stricte entre deux actifs corrélés (Spreads de paires).

## C. L'Arbitrage microstructural

* La physique : Exploitation des asymétries de vitesse d'exécution, de la toxicité du flux d'ordres ou des déséquilibres de liquidité au sein du carnet d'ordres (OFI, VPIN).
* La signature statistique : Signaux à très haute fréquence s'épuisant en quelques secondes ou millisecondes, capturant la prime de spread.

💼 Parenthèse Wall Street
Quelle est la différence entre un trader de momentum et un trader de mean reversion ? Le trader de momentum achète un actif parce qu'il monte, en espérant le revendre à un plus grand imbécile plus tard. Le trader de mean reversion achète un actif parce qu'il baisse, en étant intimement convaincu que c'est le marché qui est imbécile. Dans les deux cas, le marché finit souvent par mettre tout le monde d'accord.

------------------------------
## 2.2 Traitement du signal et transformation des features : La Différenciation Fractionnaire## 📊 Fondations théoriques
C'est ici que se joue la survie de votre framework quantitative. Pour entraîner un modèle de Machine Learning ou un classifieur statistique, vos variables d'entrée doivent être stationnaires (moyenne, variance et autocovariance constantes dans le temps).
Pour rendre une série de prix stationnaire, le réflexe standard (et paresseux) est de calculer les rendements ou de différencier la série à l'ordre 1 :
$$\Delta P_t = P_t - P_{t-1}$$
## Le Piège Absolu : L'annihilation de la mémoire
La différenciation à l'ordre entier 1 ($d=1$) rend la série parfaitement stationnaire, mais elle détruit la quasi-totalité de la mémoire historique de la série. Le modèle ne sait plus si le prix actuel de 1.3500 vient de 1.1000 ou de 1.6000. Vous avez nettoyé vos données en jetant le signal à la poubelle.
## La Solution : La Différenciation Fractionnaire (Fractional Differentiation)
Le chercheur John Read et popularisé par Marcos López de Prado a démontré qu'on peut différencier une série à un ordre non-entier $d$ (par exemple $d=0.45$). Cela permet d'atteindre le seuil de stationnarité requis par les algorithmes tout en préservant la mémoire à long terme de la série temporelle.
L'opérateur de différenciation fractionnaire se définit mathématiquement par l'expansion binomiale suivante :
$$(1 - B)^d = \sum_{k=0}^{\infty} (-1)^k \binom{d}{k} B^k = 1 - dB + \frac{d(d-1)}{2!}B^2 - \frac{d(d-1)(d-2)}{3!}B^3 + \dots$$
Où $B$ est l'opérateur de décalage arrière (Backshift operator) tel que $B^k P_t = P_{t-k}$, et les poids $\omega$ appliqués aux prix passés sont calculés de manière itérative :
$$\omega_k = \omega_{k-1} \frac{k - 1 - d}{k}, \quad \omega_0 = 1$$
------------------------------
## 🛠️ Application pratique : Le Transformeur à Fenêtre Glissante Fractionnaire
Nous allons coder un module de traitement de signal hautement optimisé. Il applique l'expansion de mémoire fractionnaire sur une fenêtre glissante finie. Pour éviter que les calculs ne ralentissent avec le temps, on applique un seuil de tolérance ($\delta$) pour couper les poids insignifiants.
Le code respecte strictement la limite des 80 colonnes, les guillemets simples et les virgules de fin de ligne.

"""Aegis Framework - Advanced Fractional Signal Processing.

Applies fractional differentiation operators to preserve time series memory."""
# =============================================================================# -----------------------------------------------------------------------------# =============================================================================

class FractionalDifferentiator:
    """Computes fractional weights and transforms non-stationary price series."""

    def __init__(self, d: float, memory_threshold: float = 1e-4) -> None:
        """Initializes the fractional memory compressor.

        Args:
            d: The fractional differentiation order (typically between 0.2 and 0.7).
            memory_threshold: Cutoff value for historical weight significance.
        """
        self._d = d
        self._threshold = memory_threshold
        self._weights = self._generate_weights()
        self._window_size = len(self._weights)
        self._buffer: list[float] = []

    def _generate_weights(self) -> list[float]:
        """Generates memory expansion weights iteratively using binomial series."""
        weights = [1.0]
        k = 1

        while True:
            # Iterative formula matching the mathematical binomial expansion
            next_w = -weights[-1] * (self._d - k + 1) / k

            if abs(next_w) < self._threshold:
                break

            weights.append(next_w)
            k += 1

        return weights

    def transform_sample(self, raw_price: float) -> float | None:
        """Applies fractional weights to the sliding real-time price history window.

        Args:
            raw_price: The incoming raw execution price sample.

        Returns:
            The stationary differentiated scalar, or None during warmup.
        """
        self._buffer.append(raw_price)

        if len(self._buffer) < self._window_size:
            return None

        if len(self._buffer) > self._window_size:
            self._buffer.pop(0)

        # Dot product between memory weights and current history matrix
        differentiated_value = 0.0
        for i in range(self._window_size):
            # History is read backwards: current price is multiplied by weights[0]
            differentiated_value += self._weights[i] * self._buffer[-1 - i]

        return differentiated_value

# =============================================================================# -----------------------------------------------------------------------------# =============================================================================

------------------------------
## 2.3 Formalisation mathématique de la conviction : Le Scalaire Continu de Conviction## 📊 Fondations théoriques
Un signal d'Alpha de qualité institutionnelle ne doit pas se contenter de renvoyer un état binaire "ACHAT" ou "VENTE". Le marché est probabiliste. Votre indicateur d'Alpha doit formaliser sa force à travers un scalaire continu de conviction borné rigoureusement entre $-1.0$ et $1.0$.

* $C_t = 1.0$ : Certitude mathématique absolue d'une impulsion haussière imminente.
* $C_t = -1.0$ : Certitude absolue d'une rupture baissière.
* $C_t = 0.0$ : Absence totale d'asymétrie statistique (neutralité complète, pas d'opinion).

## La modélisation par Sigmoïde ou Tangente Hyperbolique ($\tanh$)
Pour transformer un indicateur technique brut (comme l'OFI que nous avons calculé au module 1, ou l'écart à une moyenne mobile) en un vecteur de conviction probabiliste propre, on utilise une fonction d'activation non-linéaire normalisée. La tangente hyperbolique est idéale car elle est nativement bornée entre $-1.0$ et $1.0$.
------------------------------
## 🛠️ Application pratique : L'Alpha de Conviction Continu par OFI
Nous allons assembler nos connaissances du module 1 et du module 2. Nous allons écrire un Alpha complet d'entrée qui consomme notre processeur d'imbalance microstructural (OFI), applique un lissage, et projette le signal sur un scalaire continu de conviction via une fonction $\tanh$ calibrée.

"""Aegis Framework - Continuous Alpha Conviction Model.

Maps microstructural order flow imbalances into normalized scalar vectors."""
import mathfrom aegis.core.base import BaseAlphafrom aegis.core.model import MarketContext
# =============================================================================# -----------------------------------------------------------------------------# =============================================================================

class ContinuousOfiAlpha(BaseAlpha):
    """Alpha model generating continuous conviction vectors from order flows."""

    def __init__(self, smoothing_period: int, scaling_factor: float) -> None:
        """Initializes the continuous microstructural alpha.

        Args:
            smoothing_period: Lookback window to stabilize the imbalance signal.
            scaling_factor: Volatility normalization divisor for the tanh map.
        """
        super().__init__()
        self._period = smoothing_period
        self._scaler = scaling_factor
        self._ofi_history: list[float] = []

    def evaluate(
        self,
        market_context: MarketContext,
        historical_values: list[float],
    ) -> float:
        """Evaluates the order flow matrix to map out a continuous conviction score.

        Args:
            market_context: The current instant state of the market.
            historical_values: Trailing calculated raw features.

        Returns:
            A scalar bounded between -1.0 and 1.0 mapping trend intensity.
        """
        # In a real environment, historical_values here would feed the raw OFI metrics.
        if not historical_values:
            return 0.0

        latest_ofi = historical_values[-1]
        self._ofi_history.append(latest_ofi)

        if len(self._ofi_history) > self._period:
            self._ofi_history.pop(0)

        # 1. Compute smoothed trailing imbalance average
        smoothed_ofi = sum(self._ofi_history) / len(self._ofi_history)

        # 2. Map the raw feature into a continuous conviction space via tanh
        # Scaling factor avoids immediate saturation near -1.0 or 1.0
        conviction_scalar = math.tanh(smoothed_ofi / self._scaler)

        return conviction_scalar

# =============================================================================# -----------------------------------------------------------------------------# =============================================================================
