# Vizanix Quant Engineering Library [![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](LICENSE) [![Link Check](https://github.com/vizanix/awesome-algo-trading-engineering/actions/workflows/links.yml/badge.svg)](https://github.com/vizanix/awesome-algo-trading-engineering/actions/workflows/links.yml) [![Markdown Lint](https://github.com/vizanix/awesome-algo-trading-engineering/actions/workflows/lint.yml/badge.svg)](https://github.com/vizanix/awesome-algo-trading-engineering/actions/workflows/lint.yml)

> Eighty original books and essays on algorithmic trading engineering and on AI's trajectory as a technology and a social force, written by Vizanix, free to read in full, in English and Russian.

This is not a list of links. Every book here is written by the Vizanix team, from a first explanation of an order book to production risk systems, deep learning architectures for market prediction, and sober essays on where AI as a technology is taking society. You read the whole thing here, on GitHub, for free, with no signup and no paywall.

The trading library is organized into three levels, weighted toward beginners, because the widest audience for this material is engineers and researchers just entering algorithmic trading. Intermediate and professional books go deeper into architecture, math, and production tradeoffs. Alongside it sits a separate [AI & Society](#ai--society) collection: essays that step back from trading specifically and reason about AI's risks, benefits, and long-term trajectory, since the two fields are converging and a serious trading engineer today is, increasingly, also an AI engineer.

Read [What This Library Is](#what-this-library-is) and [What This Library Is Not](#what-this-library-is-not) before contributing — see [CONTRIBUTING.md](CONTRIBUTING.md) for how to submit a correction, a translation, or a new diagram.

## Contents

- [How to Use This Library](#how-to-use-this-library)
- [English Library](#english-library)
  - [Beginner](#beginner-english)
  - [Intermediate](#intermediate-english)
  - [Professional](#professional-english)
- [Русская библиотека](#русская-библиотека)
  - [Начинающий уровень](#начинающий-уровень)
  - [Средний уровень](#средний-уровень)
  - [Профессиональный уровень](#профессиональный-уровень)
- [AI, Machine Learning, and Markets](#ai-machine-learning-and-markets)
- [AI & Society](#ai--society)
  - [English essays](#ai--society-english)
  - [Русские эссе](#ai--society-русский)
- [What This Library Is](#what-this-library-is)
- [What This Library Is Not](#what-this-library-is-not)
- [License](#license)
- [About Vizanix](#about-vizanix)

## How to Use This Library

Pick your level, not just your interest. A professional-level book on price impact modeling assumes you've already built something like a backtester; if you haven't, start in Beginner and work up. Each book stands alone, so you don't need to read the whole shelf in order, but within a level the books are sequenced to build on each other.

Every book follows the same shape: a one-line abstract, a table of contents, numbered chapters, a summary, and a license footer. Diagrams are inline SVG, drawn specifically for this library, and shared in [`library/assets/`](library/assets/) so you'll see the same visual language reused across books (the same order book depth chart style, the same architecture diagram style, and so on).

Books are mirrored between English and Russian: the same 30 topics, same levels, same slugs, so if you find a book useful in one language, its counterpart is one click away in the other.

## English Library

### Beginner (English)

| Book | Topic |
|---|---|
| [Algorithmic Trading: A First Map of the Territory](library/en/beginner/algorithmic-trading-first-map.md) | Orientation: what algorithmic trading engineering actually covers |
| [How Markets Actually Work: Orders, Prices, and Participants](library/en/beginner/how-markets-actually-work.md) | Market structure fundamentals |
| [Reading the Order Book: A Beginner's Guide](library/en/beginner/reading-the-order-book.md) | Limit order books, bids, asks, depth |
| [Market Data 101: Ticks, Bars, and Candles Explained](library/en/beginner/market-data-101.md) | Market data primitives |
| [Python for Market Data: Your First Toolkit](library/en/beginner/python-for-market-data.md) | Practical Python for handling market data |
| [Introduction to Backtesting: Testing Ideas Before Risking Money](library/en/beginner/introduction-to-backtesting.md) | Backtesting fundamentals |
| [Understanding Exchange APIs: Connecting to Live Markets](library/en/beginner/understanding-exchange-apis.md) | REST/WebSocket exchange connectivity basics |
| [The Beginner's Guide to Crypto Exchange Connectivity](library/en/beginner/crypto-exchange-connectivity-beginner.md) | Crypto-specific connectivity |
| [Risk Management Basics for New Quant Developers](library/en/beginner/risk-management-basics.md) | Risk fundamentals |
| [From Spreadsheet to System: Building Your First Trading Bot](library/en/beginner/spreadsheet-to-system.md) | Your first end-to-end system |
| [Statistics for Trading: What You Actually Need to Know](library/en/beginner/statistics-for-trading.md) | Applied statistics foundation |
| [Common Mistakes New Algo Traders Make](library/en/beginner/common-mistakes-new-algo-traders.md) | Pitfalls and how to avoid them |
| [Neural Networks Explained for Traders: A Gentle Introduction](library/en/beginner/neural-networks-explained-for-traders.md) `AI/ML` | Neural networks from first principles |
| [Machine Learning for Market Data: First Principles](library/en/beginner/machine-learning-for-market-data-first-principles.md) `AI/ML` | ML fundamentals applied to markets |

### Intermediate (English)

| Book | Topic |
|---|---|
| [Designing an Order Management System from Scratch](library/en/intermediate/designing-oms-from-scratch.md) | OMS architecture |
| [Execution Algorithms in Practice: TWAP, VWAP, and Beyond](library/en/intermediate/execution-algorithms-in-practice.md) | Execution algorithm design |
| [Building an Event-Driven Backtesting Engine](library/en/intermediate/building-event-driven-backtesting-engine.md) | Backtester architecture |
| [Market Microstructure for Engineers](library/en/intermediate/market-microstructure-for-engineers.md) | Microstructure for builders |
| [Time-Series Analysis for Trading Systems](library/en/intermediate/time-series-analysis-for-trading-systems.md) | Applied time-series methods |
| [Observability for Trading Infrastructure](library/en/intermediate/observability-for-trading-infrastructure.md) | Metrics, logs, tracing, alerting |
| [State Reconciliation: Keeping Your System Honest](library/en/intermediate/state-reconciliation.md) | Local vs. exchange state |
| [Portfolio Risk Engineering](library/en/intermediate/portfolio-risk-engineering.md) | Portfolio-level risk systems |
| [Feature Engineering for Financial Machine Learning](library/en/intermediate/feature-engineering-for-financial-ml.md) `AI/ML` | Features, labeling, leakage |
| [Applied Time-Series Forecasting with Machine Learning](library/en/intermediate/applied-time-series-forecasting-ml.md) `AI/ML` | ML forecasting in practice |

### Professional (English)

| Book | Topic |
|---|---|
| [Low-Latency Systems Engineering for Trading](library/en/professional/low-latency-systems-engineering.md) | Latency-critical system design |
| [Smart Order Routing and Liquidity Aggregation at Scale](library/en/professional/smart-order-routing-at-scale.md) | Multi-venue routing |
| [Production Risk Systems: Kill Switches, Limits, and Circuit Breakers](library/en/professional/production-risk-systems.md) | Production-grade risk controls |
| [Building a Multi-Exchange Crypto Trading Infrastructure](library/en/professional/multi-exchange-crypto-infrastructure.md) | Multi-exchange crypto architecture |
| [Advanced Market Microstructure and Price Impact Modeling](library/en/professional/advanced-microstructure-price-impact.md) | Price impact and microstructure at depth |
| [Deep Learning Architectures for Market Prediction: A Rigorous Treatment](library/en/professional/deep-learning-architectures-market-prediction.md) `AI/ML` | Deep learning for markets, in depth |

## Русская библиотека

### Начинающий уровень

| Книга | Тема |
|---|---|
| [Алгоритмическая торговля: первая карта местности](library/ru/beginner/algorithmic-trading-first-map.md) | Ориентация в области |
| [Как устроены рынки: заявки, цены, участники](library/ru/beginner/how-markets-actually-work.md) | Основы структуры рынка |
| [Как читать стакан заявок: руководство для начинающих](library/ru/beginner/reading-the-order-book.md) | Стакан заявок, bid/ask, глубина |
| [Рыночные данные 101: тики, бары и свечи](library/ru/beginner/market-data-101.md) | Примитивы рыночных данных |
| [Python для рыночных данных: первый набор инструментов](library/ru/beginner/python-for-market-data.md) | Python для работы с данными |
| [Введение в бэктестинг: как проверять идеи, не рискуя деньгами](library/ru/beginner/introduction-to-backtesting.md) | Основы бэктестинга |
| [Биржевые API: подключение к реальным рынкам](library/ru/beginner/understanding-exchange-apis.md) | REST/WebSocket подключение |
| [Подключение к криптобиржам для начинающих](library/ru/beginner/crypto-exchange-connectivity-beginner.md) | Крипто-коннекторы |
| [Основы риск-менеджмента для начинающего квант-разработчика](library/ru/beginner/risk-management-basics.md) | Основы риска |
| [От таблицы Excel до системы: ваш первый торговый бот](library/ru/beginner/spreadsheet-to-system.md) | Первая система целиком |
| [Статистика для трейдинга: что действительно нужно знать](library/ru/beginner/statistics-for-trading.md) | Прикладная статистика |
| [Частые ошибки начинающих алго-трейдеров](library/ru/beginner/common-mistakes-new-algo-traders.md) | Типичные ошибки |
| [Нейросети простыми словами: введение для трейдеров](library/ru/beginner/neural-networks-explained-for-traders.md) `ИИ` | Нейросети с нуля |
| [Машинное обучение для рыночных данных: азы](library/ru/beginner/machine-learning-for-market-data-first-principles.md) `ИИ` | Основы ML для рынков |

### Средний уровень

| Книга | Тема |
|---|---|
| [Проектирование Order Management System с нуля](library/ru/intermediate/designing-oms-from-scratch.md) | Архитектура OMS |
| [Алгоритмы исполнения на практике: TWAP, VWAP и не только](library/ru/intermediate/execution-algorithms-in-practice.md) | Проектирование алгоритмов исполнения |
| [Создание event-driven движка бэктестинга](library/ru/intermediate/building-event-driven-backtesting-engine.md) | Архитектура бэктестера |
| [Микроструктура рынка для инженеров](library/ru/intermediate/market-microstructure-for-engineers.md) | Микроструктура для разработчиков |
| [Анализ временных рядов для торговых систем](library/ru/intermediate/time-series-analysis-for-trading-systems.md) | Прикладные методы временных рядов |
| [Observability торговой инфраструктуры](library/ru/intermediate/observability-for-trading-infrastructure.md) | Метрики, логи, трейсинг, алертинг |
| [Сверка состояния: как система остаётся честной](library/ru/intermediate/state-reconciliation.md) | Локальное состояние против биржевого |
| [Инжиниринг портфельного риска](library/ru/intermediate/portfolio-risk-engineering.md) | Риск на уровне портфеля |
| [Feature engineering для финансового машинного обучения](library/ru/intermediate/feature-engineering-for-financial-ml.md) `ИИ` | Признаки, разметка, утечки данных |
| [Прикладной прогноз временных рядов с помощью машинного обучения](library/ru/intermediate/applied-time-series-forecasting-ml.md) `ИИ` | ML-прогнозирование на практике |

### Профессиональный уровень

| Книга | Тема |
|---|---|
| [Low-latency инжиниринг торговых систем](library/ru/professional/low-latency-systems-engineering.md) | Проектирование latency-critical систем |
| [Smart Order Routing и агрегация ликвидности на масштабе](library/ru/professional/smart-order-routing-at-scale.md) | Мультибиржевая маршрутизация |
| [Производственные риск-системы: kill switch, лимиты, circuit breaker](library/ru/professional/production-risk-systems.md) | Продакшн риск-контроли |
| [Построение мультибиржевой криптотрейдинговой инфраструктуры](library/ru/professional/multi-exchange-crypto-infrastructure.md) | Мультибиржевая крипто-архитектура |
| [Продвинутая микроструктура рынка и моделирование price impact](library/ru/professional/advanced-microstructure-price-impact.md) | Микроструктура и price impact вглубь |
| [Архитектуры глубокого обучения для прогноза рынка: строгий разбор](library/ru/professional/deep-learning-architectures-market-prediction.md) `ИИ` | Глубокое обучение для рынков вглубь |

## AI, Machine Learning, and Markets

Six books across the two languages are marked `AI/ML` above, one pair at each level. They're kept inside the normal level structure rather than split into a separate track, because applying machine learning to markets is an extension of the same engineering discipline as the rest of the library, not a separate field: the same leakage discipline that matters in a backtest matters twice as much in a training pipeline, and the same production rigor that matters in an order gateway matters in a model-serving path. As neural network and general AI tooling keeps merging into trading infrastructure, expect this set to grow faster than the rest of the library.

## AI & Society

Trading infrastructure is one place AI shows up. It isn't the only one, and it won't stay the biggest one for long. This collection sets trading aside and asks harder, slower questions: what advanced AI actually threatens, what it could plausibly deliver, whether talk of a singularity means anything precise, and what institutions a world with increasingly capable AI systems might need. These essays are analytical, not promotional and not alarmist. They weigh real disagreement among serious people in the field rather than picking a side and arguing only for it, and they're explicit about where the honest answer is "nobody knows yet."

### AI & Society (English)

| Essay | Central question |
|---|---|
| [The Danger of Advanced AI: Mapping the Real Risks](library/en/ai-society/danger-of-advanced-ai.md) | Which AI risks are concrete engineering problems, and which are speculative? |
| [Prosperity with AI: A Realistic Vision of Abundance](library/en/ai-society/prosperity-with-ai.md) | What does plausible, non-utopian abundance from AI look like? |
| [The Singularity: What It Might Actually Mean](library/en/ai-society/the-singularity-explained.md) | Stripped of hype, what claim is the singularity idea actually making? |
| [Alignment: Can We Steer a Mind We Don't Fully Understand](library/en/ai-society/ai-alignment-problem.md) | Is steering increasingly capable AI systems a solvable engineering problem? |
| [Economic Disruption and Reinvention in an AI World](library/en/ai-society/ai-economic-disruption.md) | How does an economy absorb rapid, uneven automation of cognitive work? |
| [AI and Power: Concentration, Governance, and Control](library/en/ai-society/ai-and-power.md) | Who ends up holding power as AI capability concentrates? |
| [The Post-Labor Society: Work, Meaning, and AI](library/en/ai-society/post-labor-society.md) | What replaces work as a source of income and meaning? |
| [Superintelligence: Timelines, Uncertainty, and Humility](library/en/ai-society/superintelligence-timelines.md) | How seriously should specific timelines to superintelligence be taken? |
| [AI Rights and Moral Status: A Speculative Inquiry](library/en/ai-society/ai-rights-and-moral-status.md) | Could a machine ever warrant moral consideration, and how would we know? |
| [Coexistence: Designing Institutions for an AI Century](library/en/ai-society/coexistence-ai-institutions.md) | What institutions does a long coexistence with AI actually require? |

### AI & Society (Русский)

| Эссе | Центральный вопрос |
|---|---|
| [Опасность продвинутого ИИ: карта реальных рисков](library/ru/ai-society/danger-of-advanced-ai.md) | Какие риски ИИ — инженерная реальность, а какие — спекуляция? |
| [Процветание с ИИ: реалистичный взгляд на изобилие](library/ru/ai-society/prosperity-with-ai.md) | Как выглядит правдоподобное, неутопичное изобилие благодаря ИИ? |
| [Сингулярность: что это может означать на самом деле](library/ru/ai-society/the-singularity-explained.md) | Какое именно утверждение делает идея сингулярности, если убрать шумиху? |
| [Согласование целей ИИ: можно ли направлять разум, который мы не до конца понимаем](library/ru/ai-society/ai-alignment-problem.md) | Управление всё более способными системами ИИ — решаемая инженерная задача? |
| [Экономические потрясения и перезапуск в мире ИИ](library/ru/ai-society/ai-economic-disruption.md) | Как экономика переживает быструю, неравномерную автоматизацию когнитивного труда? |
| [ИИ и власть: концентрация, управление, контроль](library/ru/ai-society/ai-and-power.md) | В чьих руках оказывается власть по мере концентрации возможностей ИИ? |
| [Общество после труда: работа, смысл и ИИ](library/ru/ai-society/post-labor-society.md) | Что заменяет труд как источник дохода и смысла? |
| [Сверхинтеллект: сроки, неопределённость и скромность прогнозов](library/ru/ai-society/superintelligence-timelines.md) | Насколько серьёзно стоит воспринимать конкретные сроки появления сверхинтеллекта? |
| [Права ИИ и моральный статус: спекулятивное исследование](library/ru/ai-society/ai-rights-and-moral-status.md) | Может ли машина заслуживать морального отношения, и как мы это узнаем? |
| [Сосуществование: проектирование институтов для века ИИ](library/ru/ai-society/coexistence-ai-institutions.md) | Какие институты на самом деле нужны для долгого сосуществования с ИИ? |

## What This Library Is

- Original writing, entirely produced by Vizanix, on the engineering and research side of algorithmic trading, and on AI's broader trajectory as a technology and a social force.
- Organized by level (beginner, intermediate, professional) for the trading library, mirrored across English and Russian; the AI & Society collection is a separate, level-agnostic set of essays, also mirrored across both languages.
- Free to read in full, here, with no signup, no paywall, and no ads.
- Licensed under [CC BY 4.0](LICENSE): free to share and adapt, with attribution.
- Focused on infrastructure, market microstructure, execution, backtesting, risk, production engineering, the machine learning techniques that increasingly sit alongside all of it, and — in the AI & Society collection — the wider risks, benefits, and open questions of advanced AI.

## What This Library Is Not

- Not a curated list of external books, courses, blogs, or open source projects. This repository doesn't link out to third-party material.
- Not a source of trading strategies, signals, or performance claims. Nothing here promises returns of any kind.
- Not affiliated with, and does not cite, any specific external author, publisher, or paper by name — general technical concepts are explained in Vizanix's own words.

See [docs/curation-policy.md](docs/curation-policy.md) for the full editorial policy, and [CONTRIBUTING.md](CONTRIBUTING.md) for how to submit a correction, a translation, or a new diagram.

## License

All books and diagrams in this repository are licensed under [CC BY 4.0](LICENSE) — free to read, share, and adapt, with attribution to Vizanix.

## About Vizanix

This library is written and maintained by [Vizanix](https://vizanix.com), a software engineering team focused on algorithmic trading infrastructure, exchange integrations, execution systems, risk management, and quantitative tooling.

Building custom trading infrastructure? You can learn more about our work at [vizanix.com](https://vizanix.com) or on [GitHub](https://github.com/vizanix).
