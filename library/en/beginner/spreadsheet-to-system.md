# From Spreadsheet to System: Building Your First Trading Bot

*By Vizanix — Beginner Level*

> This book walks you through the concrete journey from a rough idea in a spreadsheet to a small, working automated trading system, one deliberate step at a time.

![diagram](../../assets/system-architecture.svg)

## Table of Contents

1. Starting Where Most People Actually Start
2. Formalizing the Idea Into Rules
3. From Spreadsheet Formulas to Code
4. Adding a Real Backtest
5. Wrapping the Strategy in a Live Loop
6. Connecting to a Real Data Feed and a Testnet
7. The First Small, Real Deployment
8. What Changes as the System Grows

## 1. Starting Where Most People Actually Start

Almost nobody starts their algorithmic trading journey by writing a polished piece of software. Most people start with a spreadsheet: a column of dates, a column of prices, maybe a formula computing a moving average, and a manual eyeball check of whether buying when the price crosses above that average seems to work out over the visible history.

This is a perfectly reasonable starting point, and this book treats it as one rather than something to be embarrassed about. A spreadsheet lets you explore an idea quickly, see the actual numbers, and build genuine intuition about how a rule behaves, all without the overhead of writing a full program. The problems appear later. A spreadsheet doesn't scale to more than a modest amount of data, it makes subtle errors easy to introduce and hard to spot, like accidentally referencing tomorrow's price today, and it has no way to actually execute a trade in the real world.

This book takes exactly this kind of spreadsheet idea and walks it through every step to becoming a small working system, using the vocabulary and concepts already introduced throughout this library: market data, backtesting, exchange APIs, and risk management. Nothing here is exotic. It's the practical stitching together of what you've already learned.

## 2. Formalizing the Idea Into Rules

Before opening any code editor, write your strategy's rules in plain, unambiguous language, precise enough that another person, or a computer, could follow them without needing to ask you a clarifying question. Vague phrasing like "buy when the trend looks strong" isn't a rule yet, it's a feeling. A real rule looks more like "buy when the twenty-period moving average of the closing price is above the fifty-period moving average, and I currently hold no position in this instrument."

Write down the exit rule with equal precision. When do you sell? Is there a stop loss, and at what level or distance from entry? Is there a target profit level where you take gains? Is there a maximum holding time after which you exit regardless of price? Leaving the exit vague is one of the most common ways a strategy that seemed clear in your head turns into something inconsistent once you try to code it.

Finally, write down the position sizing rule from the risk management book of this library: exactly how much capital or how many units you commit per trade, and how that might change based on your available capital or the instrument's recent volatility. A complete strategy definition has all three: an entry rule, an exit rule, and a sizing rule, each stated precisely enough to remove any need for judgment calls in the moment.

## 3. From Spreadsheet Formulas to Code

Translating your written rules into code, using the Python skills from earlier in this library, forces a kind of honesty your spreadsheet couldn't. Code has no tolerance for ambiguity. Every condition needs an exact, unambiguous comparison, and every edge case, like what happens on the very first day when you don't yet have enough history to compute a fifty-period moving average, needs an explicit answer.

Start by loading your historical data and computing whatever indicators your rules reference, like the moving averages in the earlier example, using the time series techniques from the Python book of this library. Then write a function that looks at the data available up to a given point in time and returns a clear decision: buy, sell, or hold. Keep this function pure in the sense that, given the same inputs, it always produces the same output, since that predictability is exactly what let you test and trust it in the first place.

Test this decision function directly against a handful of specific, hand-checked moments from your data before running any larger backtest. Pick a date where you're confident, from manually looking at the numbers, what the correct decision should be, and confirm your code agrees. This small step catches a surprising number of bugs, like an off-by-one error in your indicator calculation, before they contaminate a larger, harder-to-debug backtest run.

## 4. Adding a Real Backtest

With a working decision function in hand, build the simulation loop described in the backtesting book of this library: step through your historical data strictly in chronological order, call your decision function at each step using only data available up to that point, and simulate the resulting trades, including realistic slippage and commission assumptions from the start, not added later as an afterthought.

Compare this proper backtest's results against your original spreadsheet's rough numbers. Differences are common and worth investigating rather than dismissing, since your spreadsheet likely had subtle look-ahead bias, discussed in the backtesting book, baked into a formula that referenced a value you wouldn't have actually known yet in real time. Finding and understanding this kind of discrepancy is one of the most valuable exercises in this entire progression, because it makes the abstract warnings from the backtesting book concrete and personal.

Once your backtest results look reasonable, split your data into in-sample and out-of-sample periods as described in that same chapter, and resist adjusting your rules further after looking at the out-of-sample result. If the strategy doesn't hold up out-of-sample, that's valuable information delivered cheaply, before any real capital was ever at risk, not a reason to keep tweaking until the number you want appears.

## 5. Wrapping the Strategy in a Live Loop

A backtest processes historical data that already fully exists. A live system needs a different structure: a loop that runs continuously, checks for new data, and calls your same decision function whenever there's something new to evaluate, exactly as introduced in the anatomy of a trading system in the first book of this library.

The critical discipline here is reusing the exact same decision function you built and backtested, rather than rewriting similar-but-not-identical logic for live use. If your live system's logic diverges even slightly from what you backtested, you've silently invalidated everything your backtest told you, since you're no longer actually running the strategy you tested.

Structure the live loop to separate clearly the parts covered in the exchange API book of this library: fetching current data, calling your decision function, and, if it returns a buy or sell decision, placing the order through your exchange wrapper. Add the risk management checks from that dedicated book explicitly into this loop too, like a maximum position size check, as a safeguard that runs regardless of what the strategy function decided.

## 6. Connecting to a Real Data Feed and a Testnet

Before your live loop touches real money, connect it to a real, live data feed, using the WebSocket concepts from the exchange API book, and run it against a testnet, the practice environment with fake funds introduced in the crypto connectivity book, if your chosen exchange offers one.

This step surfaces an entirely new category of problems that a backtest, working with clean historical data, simply cannot reveal. What happens if the data feed briefly disconnects? What happens if an order gets rejected for a reason your backtest never considered, like a precision or minimum size rule? What happens if two decision triggers fire in quick succession before the first order has finished processing?

Run this testnet version for long enough to see it handle a reasonable number of real decision points, not just a few minutes, and deliberately introduce some failure conditions yourself if you can, like disconnecting your own internet connection briefly, to confirm your reconnection and reconciliation logic from the exchange API book actually works as intended rather than just in theory.

## 7. The First Small, Real Deployment

When you finally move to real capital, follow the same disciplined progression recommended in the crypto connectivity book: start with an amount you're genuinely comfortable losing entirely, treating this stage explicitly as tuition for learning the operational realities of live deployment rather than as a serious capital allocation.

Keep close, active watch over the system during this initial period rather than treating it as fully hands-off from day one, even though it's automated. Compare its actual live behavior against what your backtest and testnet runs predicted, and treat any meaningful, unexplained divergence as a signal to pause and investigate rather than to hope it resolves itself.

Keep a simple log or journal of what you observe: every trade, every unexpected event, every moment you felt tempted to manually intervene and why. This record becomes genuinely valuable input for improving the system, and it also directly serves the psychological preparation discussed in the risk management book of this library, helping you calibrate in advance for the drawdowns and rough stretches that will eventually happen.

## 8. What Changes as the System Grows

As you gain confidence and consider scaling up capital, expect several things to change. Your position sizing calculations need revisiting as absolute dollar amounts grow, since a rule that felt conservative with a small account might behave differently once real dollar amounts increase, particularly regarding your actual impact on market liquidity, covered in the order book book of this library.

You'll likely want more sophisticated monitoring than a simple log file: dashboards, alerts that reach you promptly if something looks wrong, and more systematic ways to compare live performance against backtest expectations over time. You may also want to run multiple strategies or instruments simultaneously, which reintroduces the diversification and correlation considerations from the risk management book, now with real, not just backtested, capital behind the decision.

None of this growth requires abandoning what you built here. The same core structure, a decision function, a backtest, a live loop, a wrapper around your exchange, and explicit risk checks, scales conceptually even as each individual piece becomes more sophisticated. The spreadsheet you started with was never the wrong place to begin. It was simply the first, roughest sketch of the system you've now built properly.

## Summary

- A spreadsheet idea is a legitimate starting point; the goal is formalizing it into precise, unambiguous rules before coding.
- A complete strategy definition needs an explicit entry rule, exit rule, and position sizing rule.
- Reuse the exact same decision function across backtesting and live trading; never let the two silently diverge.
- Testnet deployment surfaces operational failures a clean backtest can never reveal; test it deliberately.
- Scale capital gradually, keep close watch during early live deployment, and journal what you observe along the way.

---

*This book is part of the Vizanix Quant Engineering Library. Licensed under CC BY 4.0 — free to read, share, and adapt with attribution to Vizanix.*
