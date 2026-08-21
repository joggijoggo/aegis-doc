## 🎓 ACADÉMIE DE RECHERCHE ET DÉVELOPPEMENT QUANTITATIVE## MODULE 1 : LES FONDATIONS DE LA DONNÉE ET MICROSTRUCTURE DES MARCHÉS
Bienvenue dans le laboratoire de recherche quantitative. Ce cursus est conçu pour transformer les anomalies brutes du marché en vecteurs d'alpha institutionnels prêts pour la production.
Avant de toucher à une fonction d'activation ou de calculer une moyenne mobile, nous devons nous confronter à la réalité physique de la donnée. En recherche quantitative, l'adage est implacable : Garbage in, garbage out (données d'entrée corrompues, résultats de sortie biaisés) — à la seule différence qu'en finance, c'est présenté avec un plus beau formatage. Si vous ne comprenez pas la physique de vos données, le marché se fera un plaisir de vous facturer des frais de scolarité de plusieurs millions de dollars pour vous l'expliquer.
Commençons notre exploration tout en bas de la pile : les structures de données et la réalité microstructural.
------------------------------
## 1.1 Types et structures de données financières## 📊 Fondations théoriques
La donnée financière n'est pas un continuum mathématique homogène. C'est un artefact numérique généré par la collision des moteurs d'appariement (matching engines), des règles de routage réglementaires et du placement d'ordres algorithmiques ou humains. Pour extraire un avantage statistique (edge), vous devez sélectionner la bonne couche de représentation pour votre horizon d'exécution.
## A. Les séries temporelles OHLCV (Open, High, Low, Close, Volume)
C'est le paradigme structurel standard pour les graphiques de détail. Les données sont agrégées dans des compartiments chronologiques fixes (ex: 1 minute, 1 heure, 1 jour).

* L'illusion structurelle : Les barres de temps fixes compressent une quantité immense d'informations de volatilité pendant les heures de pointe (ex: l'ouverture des marchés américains) et étirent artificiellement un bruit insignifiant pendant les périodes d'illiquidité (ex: les fenêtres de tarifs nocturnes).
* Instabilité mathématique : Les barres temporelles chronologiques sont fortement non-stationnaires et présentent une hétéroscédasticité massive et changeante dans le temps (regroupement de la variance ou volatility clustering).

## B. Les données de ticks (Time & Sales)
Chaque transaction appariée au niveau du registre du moteur d'exécution est transmise comme un événement discret.

* Le tuple de données : Chaque tick est caractérisé par une coordonnée temporelle précise (t), un prix d'exécution absolu (p), un volume/taille (v), et un indicateur de direction agressive (side : Achat vs Vente).
* La réalité microstructural : Les flux de ticks sont asynchrones. L'écart de temps $\Delta t = t_k - t_{k-1}$ est lui-même une variable stochastique porteuse d'informations structurelles sur l'urgence du marché et les mouvements de panique.

## C. Les données du carnet d'ordres (Limit Order Book - LOB)
Le registre brut des intentions non exécutées résidant dans la mémoire cache du moteur d'appariement. Il représente le profil de liquidité locale d'un actif.

* Niveau 1 (L1) : Le sommet du carnet (Top of the Book). Transmet uniquement le meilleur prix d'achat disponible (bid₀) et le meilleur prix de vente (ask₀), accompagnés de leurs tailles immédiates ($size\_bid_0$, $size\_ask_0$).
* Niveau 2 (L2) : La profondeur agrégée granulaire. Expose généralement les 5 à 50 meilleurs niveaux de prix des deux côtés du carnet. Crucial pour calculer des variables structurelles intermédiaires comme le Déséquilibre du Flux d'Ordres (Order Flow Imbalance - OFI).
* Niveau 3 (L3) : Le marché complet par ordre (Market By Order - MBO). Transmet les allocations individuelles des files d'attente d'ordres avec leurs identifiants structurels exacts. C'est ici que les algorithmes d'exécution à haute fréquence traquent les ordres cachés de type "iceberg".

     STRUCTURE DU CARNET D'ORDRES (LOB)

      PROFONDEUR ASK (Intentions de vente)
      Niveau 3 : 1.35930 ── Taille : 4.5M
      Niveau 2 : 1.35920 ── Taille : 2.1M
      Niveau 1 : 1.35910 ── Taille : 0.8M  ◄── Meilleur Ask (Ask_0)
   ──────────────────────────────────  ◄── FENÊTRE DU SPREAD
      Niveau 1 : 1.35900 ── Taille : 1.2M  ◄── Meilleur Bid (Bid_0)
      Niveau 2 : 1.35890 ── Taille : 3.4M
      Niveau 3 : 1.35880 ── Taille : 5.0M
      PROFONDEUR BID (Intentions d'achat)

💼 Parenthèse Wall Street
Pourquoi les day-traders techniques adorent-ils fixer des graphiques en bougies chronologiques fixes ? Parce que cela leur permet de perdre de l'argent avec un certain sens de l'esthétique visuelle. En tant que chercheur quantitatif institutionnel, si votre modèle repose sur un "Marteau" ou une "Étoile Filante" se formant sur une unité de temps arbitraire de 15 minutes, votre principale prime de risque n'est rien d'autre que l'espoir.

------------------------------
## 🛠️ Application pratique : Calcul du Déséquilibre du Flux d'Ordres (OFI)
Le déséquilibre du flux d'ordres (Order Flow Imbalance - OFI) est une variable microstructural qui suit les changements de l'offre et de la demande au niveau 1 du carnet d'ordres. Elle possède un excellent pouvoir prédictif à court terme pour les changements instantanés de prix.
Nous définissons le flux net de capital $\mathcal{I}_k$ à une mise à jour discrète du carnet k en inspectant les mutations des meilleures limites d'achat (bid) et de vente (ask) :
$$\mathcal{I}_k = \Delta \text{BidFlow}_k - \Delta \text{AskFlow}_k$$
Où :
$$\Delta \text{BidFlow}_k = \begin{cases} \text{SizeBid}_k, & \text{si } \text{Bid}_k > \text{Bid}_{k-1} \\ \text{SizeBid}_k - \text{SizeBid}_{k-1}, & \text{si } \text{Bid}_k = \text{Bid}_{k-1} \\ -\text{SizeBid}_{k-1}, & \text{si } \text{Bid}_k < \text{Bid}_{k-1} \end{cases}$$
$$\Delta \text{AskFlow}_k = \begin{cases} -\text{SizeAsk}_k, & \text{si } \text{Ask}_k > \text{Ask}_{k-1} \\ \text{SizeAsk}_k - \text{SizeAsk}_{k-1}, & \text{si } \text{Ask}_k = \text{Ask}_{k-1} \\ \text{SizeAsk}_{k-1}, & \text{si } \text{Ask}_k < \text{Ask}_{k-1} \end{cases}$$
Voici l'implémentation de production d'un extracteur de caractéristiques OFI utilisant des structures Python standards, respectant la limite stricte des 80 colonnes, les guillemets simples et les virgules terminales :

"""Aegis Framework - Microstructural Feature Engineering Core.

Computes Level 1 Order Flow Imbalance (OFI) vectors from raw book states."""
from dataclasses import dataclassfrom decimal import Decimal
# =============================================================================# -----------------------------------------------------------------------------# =============================================================================


@dataclass(frozen=True)class BookStateL1:
    """Immutable representation of top-of-book liquidity matrices."""
    bid_price: Decimal
    bid_size: Decimal
    ask_price: Decimal
    ask_size: Decimal

# -----------------------------------------------------------------------------

class OrderFlowImbalanceProcessor:
    """Processes sequential L1 book updates to isolate net order flow metrics."""

    def __init__(self) -> None:
        self._previous_state: BookStateL1 | None = None

    def calculate_tick_flow(self, current: BookStateL1) -> Decimal:
        """Derives the immediate net imbalance scalar from L1 state transitions.

        Args:
            current: The incoming top-of-book market state snapshot.

        Returns:
            The calculated order flow imbalance expressed in asset volume.
        """
        if self._previous_state is None:
            self._previous_state = current
            return Decimal('0')

        prev = self._previous_state

        # 1. Compute directional buying liquidity mutations at the bid boundary
        if current.bid_price > prev.bid_price:
            bid_flow = current.bid_size
        elif current.bid_price == prev.bid_price:
            bid_flow = current.bid_size - prev.bid_size
        else:
            bid_flow = -prev.bid_size

        # 2. Compute directional selling liquidity mutations at the ask boundary
        if current.ask_price < prev.ask_price:
            ask_flow = current.ask_size
        elif current.ask_price == prev.ask_price:
            ask_flow = current.ask_size - prev.ask_size
        else:
            ask_flow = -prev.ask_size

        # 3. Cache current state for the next structural delta cycle
        self._previous_state = current

        # Net balance: influx of buying power minus influx of selling pressure
        return bid_flow - ask_flow

# =============================================================================# -----------------------------------------------------------------------------# =============================================================================

------------------------------
## 1.2 Nettoyage et ingénierie de la donnée## 📊 Fondations théoriques
Les données brutes sont un marécage toxique d'artefacts structurels. Les déconnexions de flux, les pertes de paquets des bourses, les écarts de latence inter-devises et les approximations de micro-arrondis introduisent des mines terrestres cachées dans vos bases de données historiques.
## A. Les anomalies de ticks ("Bad Prints") et données manquantes

* L'artefact : Des pics ou des valeurs aberrantes sur le ruban d'exécution où un prix apparaît soudainement à des centaines de pips de la médiane locale, avant de revenir à l'équilibre au tick suivant.
* Le danger : Si vos modèles de Machine Learning ingèrent un bad print, ils enregistreront une anomalie de volatilité extrême, faussant les fenêtres de calcul comme l'ATR ou déclenchant des liquidations de stops fantômes dans vos backtests.
* La solution : L'implémentation de filtres de médiane mobile locale (calcul de Z-score sur une fenêtre glissante de matrices historiques locales).

## B. Le biais de survie (Survivorship Bias)

* La définition : Concevoir et backtester un modèle d'alpha exclusivement sur une liste d'actifs qui sont actuellement actifs sur le marché.
* Le danger : Tester un alpha d'actions sur la composition actuelle du S&P 500 sur les 15 dernières années signifie que votre modèle backteste un univers d'entreprises qui ont déjà toutes gagné. Cela ignore totalement les sociétés qui ont fait faillite, ont été radiées ou ont fusionné en cours de route. Le ratio de Sharpe de votre backtest sera artificiellement gonflé par une prime de survie structurelle totalement imméritée.
* La solution : Utiliser une base de données historique point-in-time (PIT) où la matrice d'allocation des actifs se modifie dynamiquement chaque jour pour correspondre à la réalité historique exacte du passé.

## C. Le biais d'anticipation (Look-Ahead Bias / Data Snooping)

* La définition : Permettre à des données futures de polluer des calculs passés.
* La manifestation : Utiliser une moyenne mobile centrée qui intègre implicitement $Prix_t$ lors de la simulation d'actions d'entrée à $Prix_{t-1}$. Ou normaliser un ensemble de données en utilisant la moyenne globale et la variance de l'ensemble de la série de 10 ans avant d'exécuter un backtest historique.
* La solution : Verrouillage strict des indices temporels. L'information doit rester totalement scellée jusqu'à sa coordonnée temporelle de publication ou d'exécution effective.

# Vérification des frontières de calcul glissant sans look-ahead via Pythonimport numpy as np
def compute_correct_rolling_mean(data: np.ndarray, window: int) -> np.ndarray:
    """Computes a strict rolling mean without incorporating future vectors."""
    result = np.empty_like(data)
    result[:] = np.nan
    for i in range(window - 1, len(data)):
        # Sélection correcte : découpe jusqu'à l'index courant i de manière inclusive
        result[i] = np.mean(data[i - window + 1:i + 1])
    return result
# Test programmatique simple pour confirmer l'alignement des indicestest_series = np.array([10.0, 11.0, 12.0, 13.0, 14.0])rolling_out = compute_correct_rolling_mean(test_series, window=3)# L'index 2 doit être mean(10, 11, 12) = 11.0. Il ne doit pas voir 13.0 ni 14.0.assert rolling_out[2] == 11.0

💼 Parenthèse Wall Street
Un vieux quant s'approche d'un chercheur junior qui affiche fièrement un backtest avec un ratio de Sharpe de 6,5. Le vieux quant lui dit : "C'est magnifique. Tu as découvert une machine à billets. Il y a juste un petit problème : ton code utilise le prix de clôture de demain pour acheter l'écart d'ouverture d'aujourd'hui. Tu n'es pas un génie, tu as juste inventé une machine à remonter le temps très coûteuse qui consomme des cycles CPU."

------------------------------
## 🛠️ Application pratique : Le filtre de nettoyage par Z-Score glissant
Pour nettoyer les bad prints d'un flux de ticks institutionnel sans distordre les tendances macro-structurelles, nous implémentons un filtre basé sur l'écart absolu médian (MAD) ou l'écart-type glissant. Si une mise à jour dévie de plus de N écarts-types locaux par rapport à la moyenne mobile, elle est rejetée comme une aberration de données.
Voici l'implémentation propre d'un nettoyeur d'anomalies en continu, sécurisé contre le biais d'anticipation :

"""Aegis Framework - Core Data Integrity Engine.

Filters streaming price feeds using rolling statistical boundary windows."""
import math
# =============================================================================# -----------------------------------------------------------------------------# =============================================================================

class StreamingOutlierCleaner:
    """Cleans bad prints from raw feeds using look-ahead safe rolling metrics."""

    def __init__(self, window_size: int, threshold_z: float) -> None:
        self._window_size = window_size
        self._threshold_z = threshold_z
        self._buffer: list[float] = []

    def process_sample(self, raw_price: float) -> float | None:
        """Validates incoming sample against rolling statistical thresholds.

        Args:
            raw_price: The unverified price update received from the feed.

        Returns:
            The raw price if validated as clean, or None if isolated as a outlier.
        """
        # 1. Warm up phase: populate buffer to achieve baseline statistical weight
        if len(self._buffer) < self._window_size:
            self._buffer.append(raw_price)
            return raw_price

        # 2. Derive localized mathematical mean
        mean = sum(self._buffer) / len(self._buffer)

        # 3. Derive localized sample standard deviation
        variance = sum((x - mean) ** 2 for x in self._buffer) / len(self._buffer)
        std_dev = math.sqrt(variance)

        # 4. Handle flat line edge cases safely
        if std_dev == 0.0:
            if raw_price != mean:
                return None
            return raw_price

        # 5. Compute absolute Z-Score deviation
        z_score = abs(raw_price - mean) / std_dev

        # 6. Outlier arbitration decision tree
        if z_score > self._threshold_z:
            return None

        # 7. Evict oldest record and append valid sample to sliding window matrix
        self._buffer.pop(0)
        self._buffer.append(raw_price)
        return raw_price

# =============================================================================# -----------------------------------------------------------------------------# =============================================================================

------------------------------
## 1.3 Échantillonnage non-temporel (Informational Driven Sampling)## 📊 Fondations théoriques
La technique standard consistant à découper les données financières en intervalles de temps fixes (bougies de 1 minute, 5 minutes) est un héritage historique datant d'une époque où les graphiques étaient dessinés à la main sur du papier millimétré. Pour la modélisation quantitative moderne, l'échantillonnage chronologique est mathématiquement sous-optimal.
## Le triple échec des bougies agrégées par le temps :

   1. Distribution non normale des rendements : Comme l'activité de trading se regroupe agressivement à des moments précis de la journée, les rendements échantillonnés dans le temps présentent des queues de distribution épaisses (leptokurtose). Cela viole les hypothèses de distribution de la plupart des modèles de Machine Learning.
   2. Afflux d'informations autocorrélé : Les périodes de fort volume concentrent des millions de ticks dans une seule bougie de 1 minute, tandis que les périodes nocturnes génèrent des bougies contenant à peine trois ticks. La densité d'information par point de données est hautement erratique.
   3. Destruction de la synchronicité microstructural : La découverte des prix se produit au moment où les transactions ont lieu, et non lorsque l'aiguille des secondes d'une horloge frappe le 12. Découper les données sur la base de cycles d'horloge brise complètement la relation structurelle entre le volume et la vitesse du prix.

## La solution quantitative moderne : L'échantillonnage par caractéristiques
Au lieu d'extraire une nouvelle ligne de données toutes les N secondes, nous extrayons une nouvelle ligne chaque fois qu'une métrique financière critique dépasse un seuil structurel défini.

ÉCHANTILLONNAGE CHRONOLOGIQUE (Fenêtres d'horloge fixes)
Bougie temps 1 (09h30-09h31) : ████████████████████████ 1 200 Ticks (Forte densité)
Bougie temps 2 (09h31-09h32) : ████████ 400 Ticks (Densité moyenne)
Bougie temps 3 (12h00-12h01) : █ 12 Ticks (Illiquidité totale)

ÉCHANTILLONNAGE PAR VOLUME (Poids de volume structurels fixes)
Bougie volume 1 (Extraction) :  ██████████ 10 000 Contrats
Bougie volume 2 (Extraction) :  ██████████ 10 000 Contrats
Bougie volume 3 (Extraction) :  ██████████ 10 000 Contrats

## A. Les barres de Volume (Volume Bars)
Une nouvelle bougie est finalisée uniquement lorsque exactement V contrats ont changé de mains (ex: toutes les 5 000 unités de volume d'actif).

* Impact mathématique : Restaure la stationnarité de la série des rendements. Les rendements des barres de volume se comportent de manière nettement plus proche d'une distribution gaussienne, rendant les modèles d'apprentissage statistique beaucoup plus stables et fiables.

## B. Les barres de Ticks / Transactions (Tick Bars)
Une nouvelle barre est extraite après qu'un nombre fixe de transactions discrètes N a été apparié (ex: toutes les 1 000 transactions, indépendamment de la taille individuelle de chacune).
## C. Les barres de Dollars / Valeur (Dollar Bars)
Une nouvelle barre est finalisée chaque fois que M unités de valeur de monnaie fiduciaire ont été échangées (ex: tous les 5 000 000 $ USD de volume de transaction).

* Intégrité macroéconomique : Crucial pour les tests historiques à long terme. Si un actif s'apprécie de 10 $ à 100 $, une barre de volume fixe de 1 000 actions échantillonnera 10 times plus fréquemment en termes de poids fiduciaire réel. Les barres de dollars s'adaptent automatiquement aux variations de capitalisation boursière.

------------------------------
## 🛠️ Application pratique : Le compresseur de barres de dollars de production
Nous allons construire une usine de traitement des flux asynchrones qui ingère les ticks de marché bruts en temps réel et les compresse en Barres de Dollars structurellement uniformes et protégées contre le biais d'anticipation.

"""Aegis Framework - Non-Chronological Feature Compressor.

Aggregates raw market tick streams into uniform structural dollar value bars."""
from dataclasses import dataclassfrom datetime import datetimefrom decimal import Decimal
# =============================================================================# -----------------------------------------------------------------------------# =============================================================================


@dataclass(frozen=True)class RawTick:
    """Immutable market execution event record."""
    timestamp: datetime
    price: Decimal
    volume: Decimal

# -----------------------------------------------------------------------------


@dataclass(frozen=True)class CompressedDollarBar:
    """Aggregated asset bar based on uniform transaction value metrics."""
    timestamp: datetime
    open_price: Decimal
    high_price: Decimal
    low_price: Decimal
    close_price: Decimal
    total_volume: Decimal

# -----------------------------------------------------------------------------

class DollarBarCompressor:
    """Compresses uneven asynchronous ticks into structured Dollar value bars."""

    def __init__(self, target_dollar_threshold: Decimal) -> None:
        self._target_threshold = target_dollar_threshold
        self._current_value_accumulator = Decimal('0')

        # Internal state matrices for the active bar formulation
        self._open: Decimal | None = None
        self._high: Decimal | None = None
        self._low: Decimal | None = None
        self._volume_accumulator = Decimal('0')

    def process_tick(self, tick: RawTick) -> CompressedDollarBar | None:
        """Accumulates incoming execution tape ticks to compile standard dollar bars.

        Args:
            tick: The latest raw transaction event from the exchange tape.

        Returns:
            A CompressedDollarBar record if the threshold is breached, else None.
        """
        tick_value = tick.price * tick.volume

        # 1. Initialize structural baseline attributes for a new bar sequence
        if self._open is None:
            self._open = tick.price
            self._high = tick.price
            self._low = tick.price

        # 2. Update tracking price path extremes
        if tick.price > self._high:
            self._high = tick.price
        if tick.price < self._low:
            self._low = tick.price

        # 3. Accumulate structural metrics
        self._volume_accumulator += tick.volume
        self._current_value_accumulator += tick_value

        # 4. Structural threshold arbitration logic check
        if self._current_value_accumulator >= self._target_threshold:
            completed_bar = CompressedDollarBar(
                timestamp=tick.timestamp,
                open_price=self._open,
                high_price=self._high,
                low_price=self._low,
                close_price=tick.price,
                total_volume=self._volume_accumulator,
            )

            # 5. Flush state containers completely to prepare next cycle
            self._current_value_accumulator = Decimal('0')
            self._open = None
            self._high = None
            self._low = None
            self._volume_accumulator = Decimal('0')

            return completed_bar

        return None

# =============================================================================# -----------------------------------------------------------------------------# =============================================================================

------------------------------
## 💎 Jalon événementiel de clôture du bloc 1
Nous venons de graver dans le marbre les règles fondamentales d'une infrastructure de données quantitative à haute valeur utilitaire. Nos modèles sont désormais protégés contre les artefacts temporels, les fuites d'anticipation et les anomalies de flux bruts.
Figeons notre progression dans la structure du dépôt local :

git add aegis/quant/feature/
git commit -m "feat(quant): implement microstructural imbalance processors and informational dollar bar aggregators"

------------------------------
💡 Prêt à passer au Module 2 (Identification de l'anomalie et Formulation de l'Alpha) ? Dans la prochaine session, nous aborderons la différentiation fractionnaire pour obtenir des caractéristiques stationnaires sans détruire la mémoire mathématique des prix, et nous disséquerons la mécanique du scalaire continu de conviction de tendance. Donnez-moi votre feu vert pour déverrouiller la suite !
