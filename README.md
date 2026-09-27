# Awesome Algorithmic Trading Engineering [![Awesome](https://awesome.re/badge.svg)](https://awesome.re) [![License: CC0-1.0](https://img.shields.io/badge/License-CC0--1.0-lightgrey.svg)](LICENSE) [![Link Check](https://github.com/vizanix/awesome-algo-trading-engineering/actions/workflows/links.yml/badge.svg)](https://github.com/vizanix/awesome-algo-trading-engineering/actions/workflows/links.yml) [![Markdown Lint](https://github.com/vizanix/awesome-algo-trading-engineering/actions/workflows/lint.yml/badge.svg)](https://github.com/vizanix/awesome-algo-trading-engineering/actions/workflows/lint.yml)

> A curated collection of high-quality resources for building, testing, operating, and understanding algorithmic trading systems.

Algorithmic trading is not only alpha models. Durable trading systems depend on quality market data pipelines, correct order book handling, realistic execution assumptions, disciplined risk controls, rigorous testing, production-grade observability, and fault tolerance — plus a working understanding of market microstructure. This list is about that engineering surface: the infrastructure, tooling, and practices that sit underneath any strategy, in both traditional and crypto markets.

It is not a list of trading strategies, signals, "guaranteed profit" systems, or influencer content. See [What Belongs Here](#what-belongs-here) for the full inclusion criteria.

Contributions are welcome — read [CONTRIBUTING.md](CONTRIBUTING.md) before opening a pull request.

## Contents

- [Market Microstructure](#market-microstructure)
- [Quantitative Finance Foundations](#quantitative-finance-foundations)
- [Market Data](#market-data)
  - [Exchange APIs](#exchange-apis)
  - [WebSocket Infrastructure](#websocket-infrastructure)
  - [Order Book Reconstruction](#order-book-reconstruction)
  - [Historical Market Data](#historical-market-data)
  - [Data Storage](#data-storage)
- [Execution](#execution)
  - [Order Management Systems](#order-management-systems)
  - [Execution Algorithms](#execution-algorithms)
  - [Smart Order Routing](#smart-order-routing)
  - [Slippage & Transaction Costs](#slippage--transaction-costs)
  - [Exchange Connectivity](#exchange-connectivity)
- [Backtesting](#backtesting)
  - [Backtesting Engines](#backtesting-engines)
  - [Event-Driven vs Vectorized Backtesting](#event-driven-vs-vectorized-backtesting)
  - [Common Backtesting Errors](#common-backtesting-errors)
  - [Walk-Forward Testing](#walk-forward-testing)
  - [Monte Carlo Analysis](#monte-carlo-analysis)
- [Simulation](#simulation)
  - [Exchange Simulators](#exchange-simulators)
  - [Order Book Simulation](#order-book-simulation)
  - [Agent-Based Market Simulation](#agent-based-market-simulation)
  - [Paper Trading](#paper-trading)
- [Risk Management](#risk-management)
  - [Position & Portfolio Risk](#position--portfolio-risk)
  - [Pre-Trade Risk](#pre-trade-risk)
  - [Kill Switches & Drawdown Controls](#kill-switches--drawdown-controls)
  - [Exposure Monitoring](#exposure-monitoring)
- [Production Engineering](#production-engineering)
  - [Architecture & Event-Driven Systems](#architecture--event-driven-systems)
  - [Message Queues](#message-queues)
  - [Low-Latency Engineering](#low-latency-engineering)
  - [Concurrency](#concurrency)
  - [Fault Tolerance](#fault-tolerance)
  - [State Reconciliation](#state-reconciliation)
- [Observability](#observability)
  - [Metrics, Logging & Tracing](#metrics-logging--tracing)
  - [Alerting](#alerting)
  - [Trading-Specific Monitoring](#trading-specific-monitoring)
- [Testing](#testing)
  - [Unit & Integration Testing](#unit--integration-testing)
  - [Property-Based Testing](#property-based-testing)
  - [Chaos Testing](#chaos-testing)
  - [Exchange API Testing & Replay Testing](#exchange-api-testing--replay-testing)
- [Quant Research Tooling](#quant-research-tooling)
  - [Python](#python)
  - [Rust](#rust)
  - [C++](#c)
  - [DataFrames & Analytical Engines](#dataframes--analytical-engines)
  - [Notebooks & Experiment Tracking](#notebooks--experiment-tracking)
- [Machine Learning for Markets](#machine-learning-for-markets)
- [Crypto-Specific Engineering](#crypto-specific-engineering)
  - [Exchange APIs (Crypto)](#exchange-apis-crypto)
  - [Funding Rates, Perpetuals & Liquidations](#funding-rates-perpetuals--liquidations)
  - [Crypto Market Data & On-Chain Data](#crypto-market-data--on-chain-data)
- [Books](#books)
- [Papers](#papers)
- [Courses & Lectures](#courses--lectures)
- [Blogs](#blogs)
- [Open Source Projects](#open-source-projects)
- [Datasets](#datasets)
- [Security](#security)
- [Further Reading](#further-reading)
- [What Belongs Here](#what-belongs-here)
- [What Does NOT Belong Here](#what-does-not-belong-here)
- [About Vizanix](#about-vizanix)

---

## Market Microstructure

Market microstructure explains how orders actually become trades: how limit order books are organized, how priority rules allocate fills, and how liquidity providers and takers interact. Understanding it is a prerequisite for realistic execution modeling and backtesting.

- [Market Microstructure Theory](https://www.wiley.com/en-us/Market+Microstructure+Theory-p-9781557863684) — Maureen O'Hara's foundational text on how markets aggregate information through the trading process; the standard academic reference for the field.
- [Trading and Exchanges: Market Microstructure for Practitioners](https://global.oup.com/academic/product/trading-and-exchanges-9780195144703) — Larry Harris. Practitioner-oriented coverage of order types, price-time priority, maker/taker economics, and liquidity provision.
- [Empirical Market Microstructure](https://www.cambridge.org/core/books/empirical-market-microstructure/) — Joel Hasbrouck. Statistical treatment of price formation, information asymmetry, and adverse selection.
- [High-Frequency Trading: A Practical Guide](https://www.wiley.com/en-us/High+Frequency+Trading-p-9781118343500) — Irene Aldridge. Covers microprice, order flow imbalance, and inventory risk from a systems-oriented perspective.
- [LOBSTER](https://lobsterdata.com/) — Academic limit order book data reconstruction tool and dataset used widely in microstructure research.
- [Optimal High-Frequency Trading with Limit and Market Orders (Guéant, Lehalle, Fernandez-Tapia)](https://arxiv.org/abs/1106.5040) — Formalizes inventory risk and market making as a stochastic control problem.
- [The Microprice: A High-Frequency Estimator of Future Prices](https://arxiv.org/abs/1512.00426) — Sasha Stoikov. Introduces the microprice as a better predictor of near-term price than the mid-price.
- <!-- VERIFY RESOURCE --> Cont, Kukanov & Stoikov, *The Price Impact of Order Book Events* — widely cited work connecting order flow imbalance to short-term price impact; verify current arXiv link before adding.

## Quantitative Finance Foundations

Kept intentionally short: only material with direct relevance to engineering and research work, not general finance theory.

- [Options, Futures, and Other Derivatives](https://www.pearson.com/en-us/subject-catalog/p/options-futures-and-other-derivatives/) — John Hull. Standard reference for derivatives pricing and hedging mechanics relevant to risk systems.
- [Advances in Financial Machine Learning](https://www.wiley.com/en-us/Advances+in+Financial+Machine+Learning-p-9781119482086) — Marcos López de Prado. Directly engineering-relevant: labeling, cross-validation for time series, and leakage prevention.
- [QuantLib](https://www.quantlib.org/) — Open-source C++ library for quantitative finance: pricing engines, day-count conventions, term structures, and calendars, with bindings for Python, Java, and other languages.
- [Paul Wilmott Introduces Quantitative Finance](https://www.wiley.com/en-us/Paul+Wilmott+Introduces+Quantitative+Finance) — Accessible bridge from stochastic calculus to practical derivatives and risk modeling.

## Market Data

### Exchange APIs

- [FIX Trading Community — FIX Protocol Specifications](https://www.fixtrading.org/) — The standard messaging protocol used across most institutional equities, futures, and FX venues.
- [Nasdaq TotalView-ITCH](https://www.nasdaqtrader.com/Trader.aspx?id=DPSpecs) — Binary market data feed specification used as a reference implementation target for order-book-level feed handlers.
- [CME Group Market Data Platform (MDP 3.0)](https://www.cmegroup.com/market-data/market-data-platform.html) — Documentation for CME's market data protocol, widely referenced for futures feed handler design.
- [Interactive Brokers TWS API](https://interactivebrokers.github.io/tws-api/) — Widely used broker API covering equities, options, futures, and FX, with mature client libraries.
- [Alpaca Trading API](https://docs.alpaca.markets/) — REST/WebSocket API for US equities and crypto with a strong developer experience, commonly used for prototyping trading systems.
- [Polygon.io API Docs](https://polygon.io/docs) — REST and WebSocket market data API covering equities, options, futures, and forex.

### WebSocket Infrastructure

- [websockets (Python)](https://websockets.readthedocs.io/) — Widely used async WebSocket client/server library, a common building block for exchange feed handlers.
- [Tokio Tungstenite](https://github.com/snapview/tokio-tungstenite) `Rust` — Async WebSocket implementation built on Tokio; a common base for low-latency Rust market data clients.
- [uWebSockets](https://github.com/uNetworking/uWebSockets) `C++` — High-performance WebSocket and HTTP library used in latency-sensitive services.
- [Autobahn Testsuite](https://github.com/crossbario/autobahn-testsuite) — Standard conformance and performance test suite for WebSocket implementations; useful for validating custom feed handlers.

### Order Book Reconstruction

- [LOBSTER](https://lobsterdata.com/) — Reconstructs full order books from Nasdaq ITCH data for research use; also documents the mechanics of book reconstruction itself.
- [Order Book Simulator concepts (Stoikov, Cont, et al.)](https://arxiv.org/abs/1012.0349) — Cont, Stoikov, Talreja: *A Stochastic Model for Order Book Dynamics*, a widely cited basis for order book reconstruction and simulation logic.
- <!-- VERIFY RESOURCE --> Open-source ITCH/OUCH parsers on GitHub — several exist per exchange; verify maintenance status before listing a specific one.

### Historical Market Data

- [Tick Data LLC](https://www.tickdata.com/) — Commercial provider of historical tick-level equities, futures, and FX data with documented cleaning methodology.
- [Databento](https://databento.com/) — Historical and live market data with a usage-based pricing model and normalized schemas across venues.
- [Kaggle Datasets: Finance](https://www.kaggle.com/datasets?tags=13209-Finance) — Mixed-quality but occasionally useful source of free historical market datasets; vet each dataset individually.
- [Dukascopy Historical Data Feed](https://www.dukascopy.com/swiss/english/marketwatch/historical/) — Free tick-level FX and CFD historical data, commonly used for retail-accessible backtesting.

### Data Storage

- [Apache Parquet](https://parquet.apache.org/) — Columnar storage format that is the de facto standard for storing tick and bar data for research workloads.
- [ClickHouse](https://clickhouse.com/) — Column-oriented OLAP database widely adopted by trading firms for storing and querying large tick-data volumes with low latency.
- [TimescaleDB](https://www.timescale.com/) — PostgreSQL extension purpose-built for time-series data, useful when relational features (joins, constraints) are needed alongside time-series performance.
- [DuckDB](https://duckdb.org/) — Embedded analytical database well suited to ad-hoc research queries over Parquet files without standing up a server.
- [kdb+/q](https://kx.com/) — Column-store database and query language long used in institutional trading for its performance on time-series workloads; steep learning curve but still relevant in production HFT/quant environments.
- [InfluxDB](https://www.influxdata.com/) — Purpose-built time-series database commonly used for metrics and market data alike.

## Execution

### Order Management Systems

- [QuickFIX](https://github.com/quickfix/quickfix) `C++` — Open-source FIX engine implementation used as the basis for many OMS and connectivity layers.
- [QuickFIX/J](https://github.com/quickfixj/quickfixj) `Java` — JVM port of QuickFIX, common in institutional order management systems.
- [Hummingbot](https://github.com/hummingbot/hummingbot) `Python` — Open-source framework for running automated trading strategies with built-in connectivity to many crypto exchanges; useful as a reference OMS/connector architecture.

### Execution Algorithms

Execution algorithms manage how an order is worked into the market over time. None of the resources below claim to be profitable strategies — they describe scheduling and cost-minimization techniques for order placement.

- [Optimal Execution of Portfolio Transactions (Almgren & Chriss)](https://www.math.nyu.edu/faculty/chriss/optliq_f.pdf) — The canonical paper formalizing the trade-off between market impact and timing risk; basis for most modern execution algorithms (TWAP/VWAP/IS variants).
- [Algorithmic Trading and DMA](https://www.amazon.com/Algorithmic-Trading-DMA-introduction-strategies/dp/0956399207) — Barry Johnson. Practical, implementation-level treatment of TWAP, VWAP, POV, and implementation shortfall algorithms.
- [Optimal Trading Strategy and Supply/Demand Dynamics](https://www.sciencedirect.com/science/article/abs/pii/S0304405X07001096) — Obizhaeva & Wang. Extends execution cost modeling with a transient market impact framework.

### Smart Order Routing

- [Smart Order Routing (SOR) — overview](https://www.sec.gov/marketstructure/research.html) — SEC market structure research pages that include studies relevant to order routing and fragmented liquidity across venues.
- <!-- VERIFY RESOURCE --> A well-maintained open-source SOR reference implementation — none currently verified as actively maintained; add one if found.

### Slippage & Transaction Costs

- [Transaction Cost Analysis (TCA) primer — Investopedia](https://www.investopedia.com/terms/t/transaction-costs.asp) — Basic vocabulary reference for slippage, market impact, and implicit/explicit cost components.
- [The Cost of Latency in High-Frequency Trading (Moallemi & Sağlam)](https://www0.gsb.columbia.edu/faculty/cmoallemi/papers/latency.pdf) — Quantifies how latency translates into realized transaction costs.

### Exchange Connectivity

- [FIX Trading Community](https://www.fixtrading.org/) — Protocol specifications and conformance resources for connecting to FIX-based venues.
- [ITCH/OUCH Protocol Specifications (Nasdaq)](https://www.nasdaqtrader.com/Trader.aspx?id=DPSpecs) — Binary protocols underlying direct exchange connectivity for US equities.
- [CME iLink / MDP 3.0 documentation](https://www.cmegroup.com/confluence/display/EPICSANDBOX/Client+Systems+Wiki+Home) — Reference for connecting to CME's order entry and market data gateways.

## Backtesting

### Backtesting Engines

- [Backtrader](https://github.com/mementum/backtrader) `Python` — Mature, widely used event-driven backtesting framework with broker simulation and live-trading extensions.
- [Zipline Reloaded](https://github.com/stefan-jansen/zipline-reloaded) `Python` — Community-maintained fork of Quantopian's Zipline engine; still one of the most complete open-source pipeline-based backtesters.
- [vectorbt](https://github.com/polakowo/vectorbt) `Python` — High-performance vectorized backtesting library built on NumPy/Numba/Pandas, useful for fast large-scale parameter sweeps.
- [NautilusTrader](https://github.com/nautechsystems/nautilus_trader) `Python/Rust` — Event-driven backtesting and live trading platform with a Rust core, nanosecond-resolution timestamps, and a design aimed at parity between backtest and live execution.
- [Lean (QuantConnect)](https://github.com/QuantConnect/Lean) `C#/Python` — Open-source algorithmic trading engine underlying the QuantConnect platform; supports multi-asset backtesting and live deployment.
- [QSTrader](https://github.com/mhallsmoore/qstrader) `Python` — Educationally oriented event-driven backtesting framework from the QuantStart project.

### Event-Driven vs Vectorized Backtesting

Event-driven and vectorized backtesting make different trade-offs between realism and speed. Vectorized backtests are fast but can silently assume information is available before it should be; event-driven backtests process data sequentially and are better suited to modeling realistic order lifecycles, at the cost of speed.

- [Python for Finance: Mastering Data-Driven Finance](https://www.oreilly.com/library/view/python-for-finance/9781492024323/) — Yves Hilpisch. Contrasts vectorized and event-driven backtesting approaches with working code.
- [QuantStart: Event-Driven Backtesting](https://www.quantstart.com/articles/Event-Driven-Backtesting-with-Python-Part-I/) — Series explaining the architecture of an event-driven backtester from first principles.

### Common Backtesting Errors

A backtest can produce impressive results while being fundamentally invalid. Before evaluating performance, verify that the simulation does not use information unavailable at decision time, and that it models transaction costs, fills, and latency realistically.

- **Look-ahead bias** — using information (prices, fundamentals, corporate actions) that would not have been known at the simulated decision time.
- **Survivorship bias** — testing only on instruments that still exist today, ignoring delisted or bankrupt names.
- **Data leakage** — features or labels derived using future information, often introduced silently during feature engineering.
- **Unrealistic fills** — assuming full fills at the last traded price or mid-price regardless of order size and available liquidity.
- **Ignoring fees** — omitting exchange, clearing, or funding fees that materially affect net returns, especially for high-turnover strategies.
- **Ignoring funding costs** — relevant for perpetual futures and margin positions; funding payments can dominate returns for some strategies.
- **Incorrect latency assumptions** — assuming zero or unrealistic latency between signal generation and order arrival.

References:

- [Advances in Financial Machine Learning](https://www.wiley.com/en-us/Advances+in+Financial+Machine+Learning-p-9781119482086) — López de Prado. Extensive treatment of backtest overfitting and leakage.
- [The Probability of Backtest Overfitting](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=2326253) — Bailey, Borwein, López de Prado, Zhu. Formal framework for detecting overfit backtests via the deflated Sharpe ratio.
- [Pitfalls in Backtesting (QuantStart)](https://www.quantstart.com/articles/Successful-Backtesting-of-Algorithmic-Trading-Strategies-Part-I/) — Practical checklist of common implementation mistakes.

### Walk-Forward Testing

- [Walk Forward Analysis — Robert Pardo, *The Evaluation and Optimization of Trading Strategies*](https://www.wiley.com/en-us/The+Evaluation+and+Optimization+of+Trading+Strategies) — Original detailed treatment of walk-forward optimization methodology.
- [scikit-learn TimeSeriesSplit](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.TimeSeriesSplit.html) — Standard tool for implementing walk-forward and expanding-window cross-validation without shuffling temporal order.

### Monte Carlo Analysis

- [Monte Carlo Methods in Financial Engineering](https://link.springer.com/book/10.1007/978-0-387-21617-1) — Paul Glasserman. Reference text on simulation methods applicable to strategy robustness testing and risk estimation.
- [Trade Sequence Randomization for Robustness Testing](https://www.quantstart.com/articles/Monte-Carlo-Simulations-In-Finance-Part-I-Introduction/) — Introductory treatment of using Monte Carlo resampling of trade sequences to estimate strategy robustness ranges.

## Simulation

### Exchange Simulators

- [NautilusTrader](https://github.com/nautechsystems/nautilus_trader) `Python/Rust` — Includes a high-fidelity exchange simulation module supporting realistic order matching and latency modeling.
- [ABIDES](https://github.com/jpmorganchase/abides-jpmc-public) `Python` — Agent-based interactive discrete event simulator originally released by JPMorgan for market simulation research.

### Order Book Simulation

- [A Stochastic Model for Order Book Dynamics](https://arxiv.org/abs/1012.0349) — Cont, Stoikov, Talreja. Widely used stochastic framework for simulating limit order book evolution.
- [Market Simulation Under Adverse Selection](https://arxiv.org/abs/1704.00994) — Example of research-grade order book simulation incorporating informed trading effects.

### Agent-Based Market Simulation

- [ABIDES](https://github.com/jpmorganchase/abides-jpmc-public) `Python` — Multi-agent market simulator supporting heterogeneous trading agent types for studying market dynamics and testing execution strategies.
- [Mesa](https://github.com/projectmesa/mesa) `Python` — General-purpose agent-based modeling framework sometimes adapted for market simulation research.

### Paper Trading

- [Alpaca Paper Trading](https://docs.alpaca.markets/docs/paper-trading) — Free, API-compatible paper trading environment for US equities and crypto.
- [Binance Testnet](https://testnet.binance.vision/) — Official sandbox environment for testing trading logic against Binance's spot and futures APIs without real funds.
- [Interactive Brokers Paper Trading](https://www.interactivebrokers.com/en/trading/free-trial.php) — Paper trading accounts available through the same TWS API used for live trading, useful for integration testing.

## Risk Management

### Position & Portfolio Risk

- [Value at Risk, 3rd Edition](https://www.mheducation.com/) — Philippe Jorion. Standard reference on VaR and portfolio risk measurement methodology.
- [Riskfolio-Lib](https://github.com/dcajasn/Riskfolio-Lib) `Python` — Open-source portfolio optimization and risk metrics library covering VaR, CVaR, and drawdown-based risk measures.
- [PyPortfolioOpt](https://github.com/robertmartin8/PyPortfolioOpt) `Python` — Portfolio optimization library implementing mean-variance and other allocation methods with practical constraints.

### Pre-Trade Risk

- [FIA Market Access Risk Management Recommendations](https://www.fia.org/) — Industry guidance on pre-trade risk controls for firms with direct market access, including fat-finger and order size checks.
- [SEC Market Access Rule (15c3-5) Overview](https://www.sec.gov/rules/final/2010/34-63241.pdf) — US regulatory requirement mandating pre-trade risk controls for broker-dealers providing market access; a useful reference for what a production pre-trade risk layer needs to check.

### Kill Switches & Drawdown Controls

- <!-- VERIFY RESOURCE --> Exchange-specific kill switch documentation (e.g., CME Globex kill switch) — verify current URLs per venue before adding, as these pages move frequently.
- [Circuit Breakers and Market Volatility Controls (SEC)](https://www.sec.gov/investor/alerts/circuitbreakersbulletin.htm) — Regulatory background on market-wide circuit breakers, useful context for designing firm-level drawdown and kill-switch logic.

### Exposure Monitoring

- [Prometheus](https://prometheus.io/) — Commonly used to track real-time exposure, position limits, and risk metric breaches as time-series data with alerting rules.
- [Grafana](https://grafana.com/) — Standard visualization layer for exposure dashboards built on top of Prometheus, InfluxDB, or TimescaleDB.

## Production Engineering

### Architecture & Event-Driven Systems

- [Designing Data-Intensive Applications](https://dataintensive.net/) — Martin Kleppmann. Not trading-specific, but foundational for designing the durable, consistent systems that trading infrastructure depends on.
- [Enterprise Integration Patterns](https://www.enterpriseintegrationpatterns.com/) — Hohpe & Woolf. Reference patterns for message-driven architectures commonly used in OMS/EMS design.

### Message Queues

- [Apache Kafka](https://kafka.apache.org/) — De facto standard for durable, ordered event streaming; widely used as the backbone for market data distribution and order event logs.
- [Redpanda](https://redpanda.com/) — Kafka-API-compatible streaming platform written in C++, designed for lower latency and operational simplicity than Kafka.
- [NATS](https://nats.io/) — Lightweight, high-performance messaging system used for low-latency internal service communication in trading systems.

### Low-Latency Engineering

Latency engineering is relevant well beyond HFT: even strategies operating on second- or minute-level horizons benefit from understanding where latency is introduced and how to measure it.

- [Mechanical Sympathy (Martin Thompson's blog)](https://mechanical-sympathy.blogspot.com/) — Foundational writing on understanding hardware behavior (caches, memory layout, branch prediction) to write fast software.
- [LMAX Disruptor](https://github.com/LMAX-Exchange/disruptor) `Java` — Open-source concurrent ring-buffer data structure originally built for a low-latency financial exchange; widely studied as a reference for lock-free inter-thread messaging.
- [Aeron](https://github.com/real-logic/aeron) `Java/C++` — Efficient reliable UDP unicast, multicast, and IPC message transport, used in latency-sensitive trading systems.

### Concurrency

- [The Art of Multiprocessor Programming](https://www.elsevier.com/books/the-art-of-multiprocessor-programming/herlihy/978-0-12-415950-1) — Herlihy & Shavit. Rigorous treatment of concurrent data structures and synchronization relevant to building safe, fast trading system internals.
- [Tokio](https://tokio.rs/) `Rust` — Asynchronous runtime widely used for building concurrent network services, including market data handlers and order gateways.

### Fault Tolerance

- [Release It!](https://pragprog.com/titles/mnee2/release-it-second-edition/) — Michael Nygard. Patterns for building systems that survive partial failures: circuit breakers, bulkheads, and timeouts, all directly applicable to exchange connectivity.
- [Site Reliability Engineering (Google)](https://sre.google/books/) — Free online book covering error budgets, incident response, and reliability practices adaptable to trading infrastructure.

### State Reconciliation

Reconciliation is the process of continuously verifying that a trading system's internal view of its own state (open orders, positions, balances) matches the authoritative state held by the exchange or broker. Without it, silent drift between local and exchange state can lead to duplicate orders, incorrect risk calculations, or unrecognized fills — failures that are often invisible until they cause real losses. A production trading system should treat reconciliation as a first-class, continuously running process, not an occasional manual check.

- [FIX Drop Copy / Trade Capture Report specifications (FIX Trading Community)](https://www.fixtrading.org/) — Standard mechanisms exchanges and brokers expose specifically to support independent reconciliation of executions.
- <!-- VERIFY RESOURCE --> A dedicated open-source reconciliation engine for trading systems — none currently verified as actively maintained and trading-specific; most firms build this in-house.

## Observability

### Metrics, Logging & Tracing

- [Prometheus](https://prometheus.io/) — Time-series metrics collection and alerting system, a common default for trading system observability stacks.
- [Grafana](https://grafana.com/) — Dashboarding layer typically paired with Prometheus, InfluxDB, or Loki.
- [OpenTelemetry](https://opentelemetry.io/) — Vendor-neutral standard for distributed tracing, metrics, and logs; useful for tracing an order's path across OMS, risk, and execution services.
- [Grafana Loki](https://grafana.com/oss/loki/) — Log aggregation system designed to integrate with Grafana and Prometheus-style labels.

### Alerting

- [Prometheus Alertmanager](https://prometheus.io/docs/alerting/latest/alertmanager/) — Handles deduplication, grouping, and routing of alerts generated by Prometheus rules.
- [PagerDuty Incident Response Documentation](https://response.pagerduty.com/) — Freely available incident response practices adaptable to on-call rotations for trading operations.

### Trading-Specific Monitoring

Generic infrastructure monitoring is necessary but not sufficient for trading systems. The metrics below are specific to detecting problems in the trading logic and exchange connectivity itself, not just host or process health.

- Order rejection rate — sudden increases often indicate risk limit misconfiguration or exchange-side issues.
- Fill ratio — the proportion of resting orders that receive fills; a leading indicator of execution algorithm health.
- Realized slippage — measured against arrival price or a chosen benchmark, tracked over time to detect execution degradation.
- Stale market data detection — flagging when a feed stops updating without disconnecting.
- WebSocket disconnect/reconnect rate — frequent reconnects can silently produce gaps in market data or order state.
- Execution latency — time from signal/decision to order acknowledgment, and from order to fill.
- Inventory and position drift — divergence between expected and actual net position.
- Realized and unrealized PnL — tracked continuously, not just at end of day, to catch anomalies early.

## Testing

### Unit & Integration Testing

- [pytest](https://docs.pytest.org/) `Python` — De facto standard testing framework for Python trading systems and research code.
- [Google Test](https://github.com/google/googletest) `C++` — Standard unit testing framework for C++ trading infrastructure.

### Property-Based Testing

- [Hypothesis](https://hypothesis.readthedocs.io/) `Python` — Property-based testing library well suited to trading systems, e.g., verifying order book invariants hold under randomized sequences of operations.
- [QuickCheck (original Haskell)](https://hackage.haskell.org/package/QuickCheck) — The original property-based testing tool; useful background for understanding the technique before applying it via Hypothesis or similar ports.
- [proptest](https://github.com/proptest-rs/proptest) `Rust` — Property-based testing library used in Rust trading system components such as NautilusTrader.

### Chaos Testing

- [Chaos Monkey / Chaos Engineering (Netflix)](https://netflix.github.io/chaosmonkey/) — Origin of the chaos engineering discipline; principles apply directly to testing exchange disconnects and partial-failure scenarios.
- [Principles of Chaos Engineering](https://principlesofchaos.org/) — Concise reference for designing chaos experiments, adaptable to simulating exchange outages or feed corruption.

### Exchange API Testing & Replay Testing

- [Binance Testnet](https://testnet.binance.vision/) — Sandbox for integration-testing order entry and market data handling against real exchange behavior.
- [VCR / vcrpy](https://vcrpy.readthedocs.io/) `Python` — Records and replays HTTP interactions, useful for deterministic integration tests against exchange REST APIs.
- [WireMock](https://wiremock.org/) — Mocks and replays HTTP-based APIs, applicable to simulating exchange endpoints in integration tests.

## Quant Research Tooling

### Python

- [pandas](https://pandas.pydata.org/) — The standard data manipulation library for quantitative research in Python.
- [NumPy](https://numpy.org/) — Core numerical computing library underlying nearly all Python quant tooling.
- [SciPy](https://scipy.org/) — Scientific computing library providing optimization, statistics, and signal processing routines used throughout quant research.
- [statsmodels](https://www.statsmodels.org/) — Statistical modeling library covering time-series models (ARIMA, GARCH, VAR) relevant to quant research.

### Rust

- [ndarray](https://github.com/rust-ndarray/ndarray) — N-dimensional array library, a Rust analogue to NumPy used in performance-sensitive research and production code.
- [Polars](https://github.com/pola-rs/polars) `Rust/Python` — High-performance DataFrame library with a Rust core and Python bindings; increasingly used as a faster alternative to pandas for large datasets.

### C++

- [QuantLib](https://www.quantlib.org/) `C++` — Comprehensive quantitative finance library for pricing, calendars, and term structures.
- [Eigen](https://eigen.tuxfamily.org/) `C++` — Header-only linear algebra library commonly used in performance-critical quant and risk code.

### DataFrames & Analytical Engines

- [pandas](https://pandas.pydata.org/) — Widely adopted, flexible, but not always the fastest option for very large datasets.
- [Polars](https://github.com/pola-rs/polars) — Multi-threaded, memory-efficient DataFrame library increasingly preferred for large tick-data workloads.
- [DuckDB](https://duckdb.org/) — In-process analytical database that can query Parquet/CSV files directly with SQL, useful for research without a data warehouse.

### Notebooks & Experiment Tracking

- [JupyterLab](https://jupyter.org/) — Standard interactive research environment for quant workflows.
- [MLflow](https://mlflow.org/) — Open-source experiment tracking, useful for keeping backtest and model runs reproducible and comparable over time.
- [Weights & Biases](https://wandb.ai/site) — Experiment tracking platform commonly used for tracking model training runs, including free tiers for research use.

## Machine Learning for Markets

Machine learning applied to markets is unusually prone to data leakage and overfitting, because financial time series are non-stationary, autocorrelated, and low signal-to-noise. Treat any ML-for-trading resource that does not explicitly address these issues with skepticism.

- **Feature engineering** — [Advances in Financial Machine Learning](https://www.wiley.com/en-us/Advances+in+Financial+Machine+Learning-p-9781119482086) (López de Prado) covers fractional differentiation, triple-barrier labeling, and meta-labeling.
- **Time-series validation** — [scikit-learn TimeSeriesSplit](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.TimeSeriesSplit.html) and purged/embargoed cross-validation techniques described in López de Prado's book, which prevent information bleeding across train/test splits.
- **Leakage prevention** — [The Probability of Backtest Overfitting](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=2326253) provides a quantitative framework (deflated Sharpe ratio) for detecting overfit results.
- **Regime detection** — [hmmlearn](https://github.com/hmmlearn/hmmlearn) `Python` — Hidden Markov Model library sometimes applied to market regime classification.
- **Forecasting** — [Forecasting: Principles and Practice](https://otexts.com/fpp3/) — Free online textbook covering general time-series forecasting methodology applicable beyond markets.
- **Experiment tracking** — see [Notebooks & Experiment Tracking](#notebooks--experiment-tracking) above; reproducibility discipline matters more in ML-for-trading than almost any other ML domain, given how easy it is to overfit to a backtest.

## Crypto-Specific Engineering

### Exchange APIs (Crypto)

- [Binance API Documentation](https://binance-docs.github.io/apidocs/) — REST, WebSocket, and FIX documentation for Binance spot, margin, and futures markets.
- [Bybit API Documentation](https://bybit-exchange.github.io/docs/v5/intro) — Unified v5 API documentation covering spot, derivatives, and options.
- [OKX API Documentation](https://www.okx.com/docs-v5/en/) — REST and WebSocket documentation for OKX spot, margin, and derivatives products.
- [Coinbase Advanced Trade API](https://docs.cloud.coinbase.com/advanced-trade-api/docs/welcome) — Official documentation for Coinbase's institutional/advanced trading API.
- [Kraken REST & WebSocket API](https://docs.kraken.com/) — Official API documentation for Kraken spot and futures markets.
- [CCXT](https://github.com/ccxt/ccxt) `Python/JS/PHP` — Unified library providing a consistent interface across 100+ crypto exchange APIs; widely used for multi-exchange connectivity and research.

### Funding Rates, Perpetuals & Liquidations

- [Binance Funding Rate Documentation](https://www.binance.com/en/support/faq/introduction-to-binance-futures-funding-rates-360033525031) — Explains funding rate mechanics for perpetual futures, a cost component often ignored in naive backtests.
- [BitMEX Perpetual Contracts Guide](https://www.bitmex.com/app/perpetualContractsGuide) — One of the original detailed explanations of perpetual futures mechanics and funding rate design, from the venue that introduced the product.
- [Liquidation Mechanics (Bybit Learn)](https://learn.bybit.com/) — Explains margin, maintenance margin, and liquidation engine mechanics relevant to modeling forced deleveraging risk.

### Crypto Market Data & On-Chain Data

- [CCXT](https://github.com/ccxt/ccxt) — Also useful as a normalized market data source across many crypto exchanges.
- [Kaiko](https://www.kaiko.com/) — Institutional-grade historical and real-time crypto market data provider.
- [Dune Analytics](https://dune.com/) — SQL-based analytics platform over indexed on-chain data across multiple blockchains, useful for on-chain research.
- [The Graph](https://thegraph.com/) — Decentralized protocol for indexing and querying blockchain data via GraphQL subgraphs.

## Books

Every book below is free to read online, in full, with no paywall or signup. No paid titles here — for priced references on the same topics, see [Further Reading](#further-reading) and the citations throughout this README.

### English

- [Think Stats](https://greenteapress.com/wp/think-stats-2e/) — Allen B. Downey. Free book on applying statistics with Python; solid grounding for anyone processing market data.
- [Think Bayes](https://greenteapress.com/wp/think-bayes/) — Allen B. Downey. Free book on Bayesian methods in Python, relevant to regime detection and probabilistic forecasting.
- [Forecasting: Principles and Practice](https://otexts.com/fpp3/) — Rob J Hyndman & George Athanasopoulos. Free online textbook on time-series forecasting, published openly by the authors.
- [Python Data Science Handbook](https://jakevdp.github.io/PythonDataScienceHandbook/) — Jake VanderPlas. Free online book covering NumPy, pandas, and scikit-learn, core tools for quant research pipelines.
- [Automate the Boring Stuff with Python](https://automatetheboringstuff.com/) — Al Sweigart. Free online book; useful onboarding material for engineers new to the Python tooling used across this list.
- [The Rust Programming Language](https://doc.rust-lang.org/book/) — Steve Klabnik & Carol Nichols. The official free book, relevant given Rust's growing use in low-latency trading infrastructure.
- [The Rust Performance Book](https://nnethercote.github.io/perf-book/) — Nicholas Nethercote. Free online guide to profiling and optimizing Rust, applicable to latency-sensitive trading components.
- [Convex Optimization](https://web.stanford.edu/~boyd/cvxbook/) — Stephen Boyd & Lieven Vandenberghe. Free PDF published by the authors; foundational for portfolio optimization and risk modeling.
- [Mining of Massive Datasets](http://www.mmds.org/) — Jure Leskovec, Anand Rajaraman, Jeffrey Ullman. Free PDF from Stanford; relevant to large-scale market data processing.
- [Site Reliability Engineering](https://sre.google/books/) — Google. Free online book on production reliability practices, directly applicable to trading infrastructure uptime and incident response.

### Russian / Русскоязычные

Открытых бесплатных книг, посвящённых именно инженерии алгоритмической торговли, на русском языке немного — большинство изданий в этой узкой области выходят только платно. Раздел будет расширяться по мере проверки конкретных материалов; предложения принимаются через [issue](.github/ISSUE_TEMPLATE/suggest-resource.md) — только с рабочей ссылкой на бесплатный полный текст.

<!-- VERIFY RESOURCE --> Открытые бесплатные книги на русском языке по алгоритмической торговле, микроструктуре рынка или бэктестингу — подтверждённых вариантов пока нет. Если вы знаете такое издание с полностью бесплатным легальным доступом, откройте issue с названием, автором и прямой ссылкой.

### Vizanix

Vizanix has not yet published a book. This section is reserved for future open, freely readable material from the team — added here only once it exists and is verifiably free to read in full.

## Papers

- [Optimal Execution of Portfolio Transactions](https://www.math.nyu.edu/faculty/chriss/optliq_f.pdf) — Almgren & Chriss (2000). Foundational market-impact/timing-risk trade-off model underlying most execution algorithms.
- [A Stochastic Model for Order Book Dynamics](https://arxiv.org/abs/1012.0349) — Cont, Stoikov & Talreja (2010). Widely used stochastic model for order book simulation and queueing dynamics.
- [The Microprice: A High-Frequency Estimator of Future Prices](https://arxiv.org/abs/1512.00426) — Stoikov (2018). Introduces a liquidity-adjusted price estimator that outperforms the mid-price at short horizons.
- [The Probability of Backtest Overfitting](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=2326253) — Bailey, Borwein, López de Prado & Zhu (2014). Quantitative framework for detecting overfit trading strategies via the deflated Sharpe ratio.
- [Optimal Trading Strategy and Supply/Demand Dynamics](https://www.sciencedirect.com/science/article/abs/pii/S0304405X07001096) — Obizhaeva & Wang (2013). Transient market impact model extending Almgren-Chriss.

## Courses & Lectures

- [MIT 18.S096: Topics in Mathematics with Applications in Finance](https://ocw.mit.edu/courses/18-s096-topics-in-mathematics-with-applications-in-finance-fall-2013/) — Free MIT OpenCourseWare lecture series covering market microstructure, risk, and quantitative methods.
- [Stanford CS246 / CME241: Foundations of Reinforcement Learning with Applications in Finance](https://web.stanford.edu/class/cme241/) — Free lecture notes applying reinforcement learning formally to trading and execution problems.
- [QuantConnect Lean Algorithm Framework Documentation](https://www.quantconnect.com/docs/v2) — Structured, hands-on documentation that doubles as a course in building multi-asset algorithmic trading systems.
- [Coursera: Financial Engineering and Risk Management (Columbia University)](https://www.coursera.org/specializations/financialengineering) — University-led specialization covering derivatives pricing and portfolio risk relevant to systems engineers moving into quant roles.

## Blogs

- [Mechanical Sympathy](https://mechanical-sympathy.blogspot.com/) — Martin Thompson's writing on low-latency systems engineering, highly relevant to trading infrastructure despite not being trading-specific.
- [QuantStart](https://www.quantstart.com/) — Long-running technical blog covering backtesting architecture, statistical methods, and quant career guidance.
- [Chronicle Software Engineering Blog](https://chronicle.software/blog/) — Technical writing on low-latency Java systems, often trading-infrastructure focused.
- [High Scalability](http://highscalability.com/) — Not trading-specific, but frequently covers architecture patterns (event sourcing, message queues) directly applicable to trading systems.

## Open Source Projects

**Trading Engines**

- [NautilusTrader](https://github.com/nautechsystems/nautilus_trader) `Python/Rust` — Event-driven backtesting and live trading platform designed for parity between simulated and live execution.
- [Lean](https://github.com/QuantConnect/Lean) `C#/Python` — Multi-asset algorithmic trading engine powering the QuantConnect platform.
- [Hummingbot](https://github.com/hummingbot/hummingbot) `Python` — Framework for automated trading with broad crypto exchange connectivity.

**Backtesting**

- [Backtrader](https://github.com/mementum/backtrader) `Python` — Mature event-driven backtesting framework with broker simulation.
- [Zipline Reloaded](https://github.com/stefan-jansen/zipline-reloaded) `Python` — Community-maintained continuation of Quantopian's backtesting engine.
- [vectorbt](https://github.com/polakowo/vectorbt) `Python` — Vectorized backtesting library optimized for fast, large-scale parameter exploration.

**Market Data**

- [CCXT](https://github.com/ccxt/ccxt) `Python/JS/PHP` — Unified API across 100+ crypto exchanges for market data and trading.
- [ABIDES](https://github.com/jpmorganchase/abides-jpmc-public) `Python` — Agent-based market simulator, useful for generating synthetic order-book-level data.

**Execution**

- [QuickFIX](https://github.com/quickfix/quickfix) `C++` — Open-source FIX protocol engine, a common base for building OMS/EMS connectivity.
- [QuickFIX/J](https://github.com/quickfixj/quickfixj) `Java` — JVM implementation of the FIX engine.

**Risk**

- [Riskfolio-Lib](https://github.com/dcajasn/Riskfolio-Lib) `Python` — Portfolio optimization and risk analytics covering VaR, CVaR, and drawdown metrics.
- [PyPortfolioOpt](https://github.com/robertmartin8/PyPortfolioOpt) `Python` — Practical portfolio construction and risk-constrained optimization library.

**Research**

- [TA-Lib](https://github.com/TA-Lib/ta-lib-python) `Python/C` — Widely used technical analysis indicator library; useful as a fast, well-tested reference implementation of common indicators rather than as a strategy source.
- [Polars](https://github.com/pola-rs/polars) `Rust/Python` — High-performance DataFrame library for large-scale research workloads.

**Crypto Infrastructure**

- [Hummingbot](https://github.com/hummingbot/hummingbot) `Python` — Reference architecture for multi-exchange crypto connectivity and execution.
- [CCXT](https://github.com/ccxt/ccxt) `Python/JS/PHP` — De facto standard unified crypto exchange API library.

*GitHub star counts are not used as a quality signal in this list. Inclusion is based on technical substance, maintenance status, and relevance.*

## Datasets

- [LOBSTER](https://lobsterdata.com/) — Paid, reconstructed Nasdaq limit order book data at various depth levels; widely used in academic microstructure research.
- [Dukascopy Historical Data](https://www.dukascopy.com/swiss/english/marketwatch/historical/) — Free tick-level FX and CFD historical data; retail-accessible but with documented limitations on tick reconstruction accuracy.
- [Databento](https://databento.com/) — Paid, usage-based historical and live market data across US equities, futures, and options with normalized schemas.
- [Kaiko](https://www.kaiko.com/) — Paid institutional-grade crypto market data, including order book and trade data across major exchanges.
- [Binance Public Data](https://github.com/binance/binance-public-data) — Free historical trade, kline, and order book snapshot data published directly by Binance.
- [Kaggle Datasets: Finance](https://www.kaggle.com/datasets?tags=13209-Finance) — Free but mixed-quality; verify licensing and data quality before use in research.

## Security

Security failures in trading infrastructure tend to be catastrophic and fast-moving: a leaked API key with withdrawal permissions can be drained in minutes. This section covers defensive engineering practices only.

- **API key security** — Use exchange-provided scoped API keys; never reuse a single key across services with different trust levels.
- **Withdrawal permissions** — Disable withdrawal permissions on any API key used by automated trading systems unless withdrawal automation is an explicit, carefully isolated requirement.
- **Secret management** — Use a dedicated secrets manager (e.g., [HashiCorp Vault](https://www.vaultproject.io/), cloud KMS/secrets services) rather than environment files or source control for API keys and credentials.
- **IP whitelisting** — Restrict API key usage to known, static IP ranges where the exchange supports it.
- **Credential rotation** — Rotate API keys on a defined schedule and immediately after any suspected compromise or employee/contractor offboarding.
- **Least privilege** — Grant each service and each API key only the permissions it needs (read-only market data keys separate from order-entry keys, for example).
- **Supply-chain security** — Pin dependency versions, audit third-party trading libraries before production use, and monitor for compromised packages, given the direct financial impact of a compromised dependency in this domain.

Further reading:

- [OWASP Top Ten](https://owasp.org/www-project-top-ten/) — General web application security risks, many of which apply directly to trading system APIs and dashboards.
- [NIST SP 800-57: Recommendation for Key Management](https://csrc.nist.gov/publications/detail/sp/800-57-part-1/rev-5/final) — Reference standard for cryptographic key management practices applicable to credential handling.

## Further Reading

- [Awesome Quant](https://github.com/wilsonfreitas/awesome-quant) — Broader curated list of quantitative finance libraries and resources across many languages.
- [Awesome Systems Trading](https://github.com/paperswithbacktest/awesome-systematic-trading) — Adjacent list covering systematic trading resources with some overlap in scope.

## What Belongs Here

A resource should meet most of the following criteria to be included:

- Technically substantial — explains or implements something concrete, not a marketing overview.
- Practically useful to someone building, testing, or operating trading infrastructure.
- Actively maintained, or historically important enough to remain a relevant reference.
- Transparent about what it does, its limitations, and (for commercial resources) its pricing model.
- Directly relevant to algorithmic trading engineering, market microstructure, or the surrounding quantitative/software tooling.
- Not primarily promotional in nature.
- Free of unrealistic profit claims or "guaranteed returns" language.
- Free of obvious affiliate spam or referral-link chains.
- Not a signal-selling or copy-trading business disguised as educational content.

## What Does NOT Belong Here

The following are explicitly excluded:

- "100% profitable strategy" or "holy grail" claims of any kind.
- Pump-and-dump communities or coordination groups.
- Trading signal services, including "free trial" signal groups.
- Copy-trading promotion or affiliate-driven copy-trading platforms.
- Referral-link collections or resources whose primary value to the submitter is referral revenue.
- Low-quality, AI-generated, or content-farm articles with no original technical substance.
- SEO-driven submissions with no clear engineering or research value.
- Unverified strategies presented without methodology or code.
- Repositories or tools whose primary purpose is credential theft, account takeover, or other malicious trading activity.

See [docs/curation-policy.md](docs/curation-policy.md) for the full curation policy, including the review and removal process.

---

## About Vizanix

This project is maintained by [Vizanix](https://vizanix.com), a software engineering team focused on algorithmic trading infrastructure, exchange integrations, execution systems, risk management, and quantitative tooling.

Building custom trading infrastructure? You can learn more about our work at [vizanix.com](https://vizanix.com) or on [GitHub](https://github.com/vizanix).

## License

[CC0 1.0 Universal](LICENSE) — see the LICENSE file for details.
