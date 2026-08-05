## Part 1: Strategic Context, Managerial Synthesis, and Operational Core Priorities

### 1. Introduction
In the engineering of high-reliability quantitative algorithmic trading platforms, the execution bridge and the broker accounting adapter constitute the most critical interface between abstract mathematical models (the Domain Layer) and the low-level microstructural realities of execution brokers. Any decimal or temporal discrepancy at this boundary results in broker order rejections, cascading duplicate fills (order spamming), or post-mortem telemetry data corruption.

This document formalizes the production specifications and the accelerated, risk-mitigated engineering roadmap for **AEGIS**. Following a microstructural code audit of **Backtrader v1.9.78.123**, which exposed core native constraints of Cerebro's cash ledgers (asymmetric free cash reduction, silent flat margin overrides under stocklike structures, and historical execution price anchoring), this roadmap integrates these empirical facts into a production-first trajectory focused on capital safety and fast alpha deployment.

### 2. Managerial Synthesis
The core engineering philosophy of this roadmap relies on an unyielding architectural trade-off: **sanctuary low-level local accounting logic and bot state determinism before confronting the asynchronous latency and connectivity failure profiles of live exchange networks.**

The organization deliberately rejects the pitfalls of long-term software tunnels (building complex API connectors without tracking tools) as well as cosmetic misallocations (building heavy front-end portals before completing an airtight execution core).

#### The 4 Strategic Pillars of the Trajectory:
1. **Backtest Integration Closing (Immediate Priority)**: Integrates the universal accounting equation into the adapter, separating margin instruments from spot cash allocations to permanently secure a 100% green status across multi-threaded integration suites.
2. **Bot Industrialization (Low but Acceptable Alpha)**: Deploys an abstract vectorized signal factory using fast Polars arrays to block look-ahead bias, coupled with a strict Finite State Machine (FSM) bot template to structurally eliminate in-flight order spamming.
3. **The Risk Shield & Streamlit UI (The Operational Insurance)**: Before connecting any live exchange network, the framework forces the deployment of a pre-trade risk barrier, an in-flight order state tracker, a persistent transactional ledger, and a lightweight **Streamlit Monitoring Dashboard** to grant real-time human supervision over floating portfolios.
4. **The Baptism of Fire (IG API Paper Trading)**: Connects AEGIS to live market data streams and routes execution orders to an IG demo account, fully protected by an infrastructure network fault-tolerance and hot-swap reconnection module.

This decoupled design guarantees a balanced velocity between systems engineering and alpha research, mitigating operational risks at every single milestone.

## Part 2: Concrete Milestone Specifications and Technical Evaluation Matrix

*Planning Note: Every milestone is evaluated using a 3-star matrix (⭐). Timeframes correspond to dedicated, high-intensity working days of pure engineering, including business code, automated unit/integration tests, and technical documentation.*

### 1. Milestone Specifications Catalogue

#### 📌 Phase 1: Infrastructure Backtest & Accounting Extraction (Closing Milestone 5)
*   **Milestone 1.1: `COMPUTATIONAL-SRP` | Risk Discrimination Extraction**: Refactors the active position loop. The adapter natively isolates margin-based contracts (Futures, Forex leverage) evaluated at historical entry cost (`position.price`) from spot cash allocations (Stocks, Forex flat margin overrides) evaluated at upfront acquisition values to wipe out Cerebro's internal ledger biases.
    *   *Timeframe*: **1 day** | *Priority*: ⭐⭐⭐ | *Complexity*: ⭐
*   **Milestone 1.2: `CLOCK-SYNC-TEST` | Integration Test Calibration Gate**: Re-aligns the automated multi-threaded integration test bot assertions on the second cycle of simulation (Bar 2). This fies the capture window directly during the active flotation state of the order, neutralizing false-negative liquidations triggered on closed-bar historical streams at Cycle 3.
    *   *Timeframe*: **1 day** | *Priority*: ⭐⭐⭐ | *Complexity*: ⭐

#### 🤖 Phase 2: Vectorized Alpha Factory (First Stateful Bots)
*   **Milestone 2.1: `SIGNAL-FACTORY` | Abstract Vectorized Indicator Engine**: An abstract engineering layer (Polars/NumPy) allowing quants to code cross-asset mathematical signals (VWAP, rolling standard moments) with high-speed performance, absolute broker independence, and a native lock against look-ahead bias leakage.
    *   *Timeframe*: **1 day** | *Priority*: ⭐⭐⭐ | *Complexity*: ⭐⭐
*   **Milestone 2.2: `STATE-STRATEGY` | Finite State Machine Bot Core**: Squelette bot structure forcing algorithms to transition through explicit, deterministic execution states (`SCANNING`, `ORDER_IN_FLIGHT`, `POSITION_ACTIVE`). This formalization prevents concurrent race conditions where a bot re-emits a buy signal while an identical parent order is still in-flight.
    *   *Timeframe*: **1 day** | *Priority*: ⭐⭐⭐ | *Complexity*: ⭐⭐

#### 🛡️ Phase 3: The Risk Shield, Transactional Persistence & Streamlit UI
*   **Milestone 3.1: `RISK-BARRIER` | Pre-Trade Compliance Coupe-Circuit**: A capital defense module intercepting incoming orders *before* broker submission. Triggers automated cancellations if a requested allocation breaches the remaining liquid free cash threshold.
    *   *Timeframe*: **1 day** | *Priority*: ⭐⭐⭐ | *Complexity*: ⭐⭐
*   **Milestone 3.2: `ORDER-STATE-MACHINE` | In-Flight Order Tracking**: A live register tracking intermediate asynchronous order statuses (`SUBMITTED`, `PENDING_SUBMIT`, `PARTIAL`). Freezes subsequent bot alpha triggers until the active order has received its network matching confirmation.
    *   *Timeframe*: **2 days** | *Priority*: ⭐⭐⭐ | *Complexity*: ⭐⭐
*   **Milestone 3.3: `STATEFUL-LEDGER` | Persistent Transactional Journal**: Commits active position ledger records and account metrics into an append-only transaction journal or a lightweight local SQLite database. This guarantees AEGIS can recover from a hardware crash or power loss without losing track of live exposures running on the exchange.
    *   *Timeframe*: **2 days** | *Priority*: ⭐⭐ | *Complexity*: ⭐⭐
*   **Milestone 3.4: `STREAMLIT-DASHBOARD` | Real-Time Telemetry & Monitoring UI**: A lightweight Streamlit graphic dashboard plugged straight into local persistence files. Displays real-time Equity curves, free available margin levels, and bot activity flags for human oversight.
    *   *Timeframe*: **1 day** | *Priority*: ⭐⭐ | *Complexity*: ⭐

#### 🚀 Phase 4: Live Network Ingestion (IG API Paper Trading)
*   **Milestone 4.1: `IG-LIVE-BRIDGE` | IG Broker Infrastructure Adapter**: Implémente l'adaptateur `IGBrokerAdapter`. Establishes async connectivity to live pricing and execution streams (Lightstreamer Websockets) and integrates REST endpoints for order submission in a safe demo environment.
    *   *Timeframe*: **4 days** | *Priority*: ⭐⭐⭐ | *Complexity*: ⭐⭐⭐
*   **Milestone 4.2: `IG-FAULT-TOLERANCE` | Network Resilience & Hot-Swap Reconnexion**: Captures WebSocket disconnect events, freezes alpha processing loops during network outages, manages automatic retry cycles, and runs automated REST polling to re-reconcile the ledger with the broker upon re-awakening.
    *   *Timeframe*: **4 days** | *Priority*: ⭐⭐⭐ | *Complexity*: ⭐⭐⭐

#### 🔬 Phase 5: Quantitative Acceleration (Advanced Alpha R&D)
*   **Milestone 5.1: `WALK-FORWARD-OPTIMIZER` | Time-Series Cross-Validation**: Anti-overfitting backtesting engine that splits data history into walking In-Sample optimization and Out-of-Sample validation windows to verify parameter robustness.
    *   *Timeframe*: **2 days** | *Priority*: ⭐⭐ | *Complexity*: ⭐⭐
*   **Milestone 5.2: `ANOMALY-DETECTOR` | Microstructural Market Probes**: Statistical probe library capturing Order Book Imbalance (OBI), volume anomalies, or structural liquidity gaps.
    *   *Timeframe*: **2 days** | *Priority*: ⭐⭐ | *Complexity*: ⭐⭐

---

### 2. Technical Evaluation Matrix

*   **Importance / Priority**: ⭐ (Optional) | ⭐⭐ (Highly Important) | ⭐⭐⭐ (Critical System Survival)
*   **Technical Complexity**: ⭐ (Straightforward) | ⭐⭐ (Robust Logic) | ⭐⭐⭐ (Advanced Concurrent / Network Engineering)

| ID | Operational Milestone | Importance / Priority | Technical Complexity | Effort (Working Days) |
| :--- | :--- | :---: | :---: | :---: |
| **1.1** | `COMPUTATIONAL-SRP` (Margin Extraction) | ⭐⭐⭐ | ⭐ | **1 day** |
| **1.2** | `CLOCK-SYNC-TEST` (Test Alignment) | ⭐⭐⭐ | ⭐ | **1 day** |
| **2.1** | `SIGNAL-FACTORY` (Vectorized Indicators)| ⭐⭐⭐ | ⭐⭐ | **1 day** |
| **2.2** | `STATE-STRATEGY` (Bot Finite States) | ⭐⭐⭐ | ⭐⭐ | **1 day** |
| **3.1** | `RISK-BARRIER` (Pre-Trade Coupe-Circuit) | ⭐⭐⭐ | ⭐⭐ | **1 day** |
| **3.2** | `ORDER-STATE-MACHINE` (In-Flight Track) | ⭐⭐⭐ | ⭐⭐ | **2 days** |
| **3.3** | `STATEFUL-LEDGER` (Local Persistence) | ⭐⭐ | ⭐⭐ | **2 days** |
| **3.4** | `STREAMLIT-DASHBOARD` (Telemetry UI) | ⭐⭐ | ⭐ | **1 day** |
| **4.1** | `IG-LIVE-BRIDGE` (IG API Connector) | ⭐⭐⭐ | ⭐⭐⭐ | **4 days** |
| **4.2** | `IG-FAULT-TOLERANCE` (Network Resilience)| ⭐⭐⭐ | ⭐⭐⭐ | **4 days** |
| **5.1** | `WALK-FORWARD-OPTIMIZER` (WFA Engine) | ⭐⭐ | ⭐⭐ | **2 days** |
| **5.2** | `ANOMALY-DETECTOR` (Microstructure Probes)| ⭐⭐ | ⭐⭐ | **2 days** |

## Part 3: "The Compliant Fortress" Chronological Trajectory and Final Wrap-up

The implementation follows a sequential, risk-mitigated progression spanning **21 cumulative working days**, ensuring that no network exchange lines are opened before embedding structural compliance, local persistence, and visual supervision.

### 1. Sequential Execution Gates

#### 🔸 Phase 1: Closing Backtest — Days 1 & 2 (Effort: 2 days)
*   **Objective**: Secure 100% automated test compliance under Backtrader.
*   **Livrable**: Refactored `get_account_snapshot` method embedding the universal risk equation matrix. Running `pytest` outputs a green, deterministic status across the multi-threaded infrastructure suite.

#### 🔸 Phase 2: Vectorized Alpha Factory — Days 3 & 4 (Effort: 2 days)
*   **Objective**: Deliver the core research framework to write simple strategies (low but acceptable alpha) backed by deterministic execution guards.
*   **Livrable**: Fast Polars vector engine integrated into the Domain (`SIGNAL-FACTORY`) and `StatefulBot` abstract classes deployed to eliminate duplicate order submissions.

#### 🔸 Phase 3: The Risk Shield, Transactional Persistence & Streamlit UI — Days 5 to 11 (Effort: 7 days)
*   **Objective**: Construct the pre-trade capital defense armure and provide absolute visual transparency.
*   **Livrable**: The pre-trade barrier rejects unbacked allocation requests; the order machine handles network latencies; the local SQLite database locks position files against hardware failure; and **the Streamlit dashboard displays real-time Equity curves, free available margin levels, and active bot positions**.

#### 🔸 Phase 4: Live Network Ingestion (IG Paper Trading) — Days 12 to 17 (Effort: 8 days)
*   **Objective**: Transition away from the backtester to confront live market data streams and asynchronous demo execution.
*   **Livrable**: `IGBrokerAdapter` listening to live price streams via WebSockets. The fault-tolerance module automatically resolves micro-disconnects and handles REST polling ledger synchronization upon socket awakening.

#### 🔸 Phase 5: Quantitative Acceleration (Advanced R&D) — Days 18 to 21 (Effort: 4 days)
*   **Objective**: Initiate advanced quantitative research focusing on institutional-grade alpha signals.
*   **Livrable**: Walk-Forward Optimizer operational to clear out strategy overfitting parameters; microstructure analytics library active.

### 2. Conclusion and Final Summary
This specification document aligns the AEGIS roadmap with true professional trading priorities. Positioning **local persistence and the Streamlit monitoring dashboard directly before connecting live network APIs** acts as a bulletproof insurance policy. Aegis ceases to be a black-box concurrent application, becoming instead a transparent, resilient quantitative trading environment ready to trade basic bots safely on live exchange feeds.
