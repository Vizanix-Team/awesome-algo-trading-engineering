# Algorithmic Trading: A First Map of the Territory

*By Vizanix — Beginner Level*

> This book gives you a working mental map of what algorithmic trading is, who does it, and how the pieces fit together before you write a single line of trading code.

![diagram](../../assets/system-architecture.svg)

## Table of Contents

1. What "Algorithmic Trading" Actually Means
2. The Cast of Characters
3. Why Automate Trading At All
4. The Anatomy of a Trading System
5. Strategy Types You Will Encounter
6. Time Horizons and What They Change
7. Where Beginners Get Confused
8. Building Your Own First Map

## 1. What "Algorithmic Trading" Actually Means

Algorithmic trading means using a computer program to decide when to buy or sell a financial instrument, and then to send that decision to a market automatically. Nothing more mystical than that. You write rules, the program checks those rules against incoming data, and when the rules trigger, the program submits an order without a human clicking a button in the moment.

A financial instrument is anything you can buy or sell in a market: a stock, a bond, a currency pair, a futures contract, a cryptocurrency. Trading means exchanging one of these for cash or for another instrument. When you add "algorithmic," you're just saying the decision-making step got handed to code.

You might picture algorithmic trading as an exotic activity reserved for hedge funds with supercomputers. Some of it is exactly that. But the same principle scales down to a hobbyist checking price data every hour and firing a script that buys a small amount of an asset when a condition is met. The core idea, "let code decide," stretches from a weekend project to systems processing millions of orders per second.

Three things distinguish algorithmic trading from simply "trading with a computer nearby." First, the decision logic is explicit and repeatable: given the same inputs, the program produces the same output every time. Second, the program can act without waiting for you to notice something, meaning it can react in milliseconds instead of the seconds or minutes a human needs. Third, because the logic is code, you can test it against historical data before ever risking real money, a process called backtesting that a later book in this library covers in depth.

None of this requires you to predict the future with certainty. Good algorithmic trading is closer to running a small, disciplined business: you define an edge (a repeatable reason you expect to make money on average), you manage risk so a single bad outcome doesn't wipe you out, and you let the numbers, not your emotions, pull the trigger. The code is just the tool that keeps you honest and fast.

## 2. The Cast of Characters

Every market has participants, and understanding their motives explains why prices move the way they do.

Market makers quote both a price at which they'll buy (the bid) and a price at which they'll sell (the ask), profiting from the small gap between the two, called the spread. They provide liquidity, meaning they make it possible for you to trade immediately instead of waiting for someone else to show up with the opposite order.

Institutional investors, like pension funds or asset managers, move large amounts of money and often need to execute big orders without moving the price too much against themselves. Their trading style is shaped by size: a fund buying a billion dollars of stock cannot simply drop it into the market at once without pushing the price up.

Retail traders are individuals, often trading through an app or broker, in smaller sizes. You're likely reading this book as a retail participant, a quant developer, or someone building tools for either group.

Arbitrageurs look for price discrepancies between related instruments or venues and trade to close that gap, earning a small, low-risk profit in the process. If gold trades at a slightly different price on two exchanges, an arbitrageur buys where it's cheap and sells where it's expensive.

High-frequency trading firms operate at extremely short time horizons, sometimes holding positions for fractions of a second, competing on speed and infrastructure quality.

Each of these groups leaves a signature in the data: order sizes, timing patterns, how aggressively they cross the spread. Part of learning algorithmic trading is learning to read those signatures.

## 3. Why Automate Trading At All

Speed is the most obvious reason. A price-sensitive opportunity might exist for only a few hundred milliseconds. No human can reliably spot and react to that.

Consistency matters just as much. Humans get tired, get scared after a loss, get greedy after a win. A well-built algorithm executes the same rule at 3am on a Tuesday as it does after a string of losses. That discipline is often worth more than raw speed for a beginner.

Scale is another driver. If you want to track a hundred instruments simultaneously and react to changes in any of them, you need software, because no person can watch a hundred screens at once.

Finally, automation lets you separate the thinking phase from the doing phase. You can spend a weekend designing and testing a strategy, then let it run unattended during the week, checking in periodically rather than staring at charts all day.

None of this promises profit. Automating a bad idea just loses money faster and more consistently than a human would. The value of automation is in execution quality, not in inventing an edge out of nowhere.

## 4. The Anatomy of a Trading System

Picture the system as a pipeline. Data flows in one end, decisions come out the other, and orders go out to the market.

First comes the market data feed: a stream of prices, trades, and order book updates coming from an exchange or a data provider. Your system needs to receive, parse, and store this reliably.

Next is the strategy or signal layer, where your logic lives. This is where you compare a moving average to the current price, check whether a spread has widened beyond a threshold, or evaluate a machine learning model's output. The strategy layer produces a decision: buy, sell, or do nothing.

Then comes risk management, a layer that sits between your strategy and the market and refuses to let a single decision blow up your account. It checks position sizes, enforces stop losses, and can veto an order that violates a rule, regardless of how confident the strategy is.

After risk checks pass, the order management layer translates the decision into an actual order message and sends it to the exchange through an API, a defined way for software to talk to the exchange's systems. It also tracks the order's status: is it filled, partially filled, rejected, or still waiting.

Finally, a monitoring and logging layer records everything that happened, so you can review performance, debug problems, and prove to yourself that the system behaved as designed.

Even a tiny hobby project touches all five layers, just in a simpler form. Recognizing the layers early helps you organize code sensibly instead of writing one giant tangled script.

## 5. Strategy Types You Will Encounter

Trend-following strategies bet that an asset moving in one direction will keep moving that way for a while. If a stock has risen steadily for two weeks, a trend follower buys, expecting the momentum to continue.

Mean-reversion strategies bet the opposite: that prices swinging far from their recent average will snap back. If an asset suddenly drops far below its typical price without an obvious reason, a mean-reversion trader buys, expecting a bounce.

Arbitrage strategies exploit price differences between related instruments, as mentioned earlier. These tend to be lower risk per trade but require speed and low transaction costs to be worthwhile.

Market-making strategies place both buy and sell orders around the current price, earning the spread repeatedly while managing the risk of holding inventory.

Event-driven strategies react to specific triggers, like scheduled economic announcements or unusual trading volume, rather than to a continuous signal.

You'll likely start with trend-following or mean-reversion strategies, since they're conceptually simple and require the least infrastructure. Arbitrage and market-making demand faster execution and tighter cost control, which makes them harder for a beginner to run profitably.

## 6. Time Horizons and What They Change

How long you hold a position changes almost everything about your system's design. A strategy that holds for months cares about company fundamentals and macroeconomic trends, checks data once a day, and can tolerate a slow, simple codebase.

A strategy that holds for minutes to hours needs faster data updates, tighter risk controls, and a system that can react without you watching it constantly.

A strategy that holds for seconds or less, the domain of high-frequency trading, needs specialized infrastructure: colocated servers physically close to the exchange, highly optimized code, and a deep understanding of exchange mechanics. This tier is expensive and competitive, and it's not where a beginner should start.

As a new algo developer, pick a time horizon that matches your available time, capital, and infrastructure. Trading on hourly or daily bars with a laptop and a free data feed is a completely legitimate place to begin, and the lessons transfer upward if you later want to move faster.

## 7. Where Beginners Get Confused

The first confusion is mistaking a backtest that looks good for a strategy that will make money live. A backtest can look great because of a coding bug, because you unconsciously tuned it to fit historical noise, or because it ignores real trading costs. Treat every backtest with suspicion until you understand exactly why it produced its result.

The second confusion is underestimating costs. Every trade has a spread to cross, and possibly a commission or fee. A strategy that looks profitable before costs can turn unprofitable once you account for them realistically.

The third confusion is conflating "automated" with "hands-off forever." Automated systems still need monitoring, updates, and occasional intervention when market conditions change in ways your rules didn't anticipate.

The fourth confusion is jumping straight to complex machine learning models before understanding basic market mechanics. A simple, well-understood strategy that you can explain in one sentence usually beats a complex model you can't fully interpret, especially early on.

## 8. Building Your Own First Map

Before you write code, sketch your own version of the pipeline in section 4, naming the specific instruments, data source, and time horizon you plan to use. Write one sentence describing your intended edge: why do you think this strategy makes money, in terms anyone could understand.

Then look honestly at your constraints: how much time can you spend monitoring the system, how much capital can you risk, what data can you actually access for free or cheaply. Your first project should fit comfortably inside those constraints rather than stretching toward the most sophisticated approach you've read about.

This book gave you the map. The rest of this library walks you through each region of that map in detail, starting with how markets actually work underneath the price you see on a chart.

## Summary

- Algorithmic trading means letting code make and execute trading decisions using explicit, testable rules.
- Markets contain different participant types, each leaving a distinct footprint in the data.
- Automation buys you speed, consistency, and scale, not a guaranteed edge.
- A trading system is a pipeline: data in, strategy logic, risk checks, order execution, and monitoring.
- Match your strategy type and time horizon to your actual time, capital, and infrastructure before you build.

---

*This book is part of the Vizanix Quant Engineering Library. Licensed under CC BY 4.0 — free to read, share, and adapt with attribution to Vizanix.*
