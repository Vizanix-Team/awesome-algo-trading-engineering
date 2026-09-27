# Common Mistakes New Algo Traders Make

*By Vizanix — Beginner Level*

> This book collects the recurring, predictable mistakes that trip up new algorithmic traders, so you can recognize and avoid them before they cost you real money.

## Table of Contents

1. Trusting a Backtest Too Quickly
2. Ignoring Costs Until It's Too Late
3. Confusing a Good Idea With a Tested Idea
4. Sizing Positions by Feel Instead of by Rule
5. Treating Automation as "Set and Forget"
6. Chasing Complexity Instead of Understanding
7. Skipping the Testnet Because It Feels Like a Waste of Time
8. Letting Emotion Back Into an Automated System

## 1. Trusting a Backtest Too Quickly

The single most common mistake this library sees among new algo traders is treating a good-looking backtest as proof of a good strategy, rather than as one piece of evidence that still needs scrutiny. The backtesting book earlier in this library covered look-ahead bias and overfitting in detail, but knowing about these traps in theory and actually catching them in your own specific backtest are different skills, and the gap between them is where this mistake lives.

A backtest that shows a smooth, steadily rising equity curve with an enormous return over a short historical period should trigger suspicion before excitement. Real trading is messy; a suspiciously clean result is far more likely to reflect a coding bug, a subtle instance of look-ahead bias, or an unrealistic cost assumption than a genuinely powerful, previously undiscovered edge.

Build the habit of actively trying to break your own backtest before trusting it. Deliberately check whether any input could have leaked future information. Rerun the same strategy on a completely different time period or instrument and see whether the result holds up reasonably, rather than accepting the very first promising result you produce and moving straight to live deployment.

## 2. Ignoring Costs Until It's Too Late

New traders frequently build and test a strategy without trading costs, get excited about the resulting numbers, and only add realistic slippage and commission assumptions as an afterthought, if at all. As the backtesting book explained, this ordering itself biases you: you form an emotional attachment to the exciting, cost-free numbers first, which makes you more inclined to explain away or minimize the less exciting, cost-adjusted numbers that follow.

This mistake hits frequent-trading strategies especially hard, since costs are paid on every single trade and compound quickly. A strategy trading a hundred times a day with a small edge per trade can look wonderful before costs and become a reliable loser after them, simply because the accumulated cost of crossing the spread a hundred times a day outweighs the accumulated small edges.

Build cost assumptions into your backtest from the very first version you run, not as a later refinement, and make those assumptions at least as pessimistic as your best honest estimate of real trading conditions, since underestimating costs is a far more common and more damaging error among beginners than overestimating them.

## 3. Confusing a Good Idea With a Tested Idea

A strategy idea can sound completely sensible in conversation, backed by a plausible-sounding story about why it should work, and still fail to hold up once actually tested rigorously against data. New traders often skip or rush the testing step precisely for the ideas that feel most obviously correct, reasoning that a sufficiently sensible idea doesn't need the same scrutiny as a more speculative one.

This reasoning is backwards. A plausible story makes an idea more appealing to trade, but it says nothing about whether the specific numerical relationship the idea depends on actually held historically, or whether the effect, even if real, is large enough to survive real trading costs. Markets are full of ideas that sound reasonable and simply don't produce a measurable, exploitable edge once you check.

Treat every strategy idea, no matter how intuitively appealing, with exactly the same testing discipline from the backtesting book: proper out-of-sample validation, realistic costs, and honest scrutiny for look-ahead bias. The appeal of an idea and the evidence for an idea are two entirely separate questions, and conflating them is one of the fastest paths to a costly, avoidable mistake.

## 4. Sizing Positions by Feel Instead of by Rule

Even traders who carefully test their entry and exit logic often size their positions inconsistently, adding more to trades that "feel" more confident and less to trades that feel uncertain, rather than following the disciplined, calculated approach from the risk management book of this library. This feels reasonable in the moment, since confidence seems like relevant information, but it quietly reintroduces exactly the kind of unmeasured human judgment that a systematic strategy was supposed to remove.

The practical danger is that "feeling more confident" correlates poorly with actual outcome probability and often correlates instead with recent results: traders tend to feel more confident right after a winning streak, precisely the moment position sizing discipline from the risk management book warns against increasing size. Sizing by feel during a hot streak is a direct, well-worn path to an oversized position at exactly the wrong time.

Define your position sizing rule explicitly and mechanically, as covered in the risk management book, and apply it identically regardless of how a given trade happens to feel. If your strategy's testing suggests confidence levels genuinely predict better outcomes, encode that insight explicitly as a measurable input to your sizing formula, rather than trusting your own in-the-moment gut sense of it.

## 5. Treating Automation as "Set and Forget"

New traders sometimes assume that once a strategy is automated and running, their job is essentially finished, and they can walk away entirely. Automation removes the need for constant manual decision-making, but it doesn't remove the need for ongoing attention, and treating it as fully hands-off is a mistake that turns small problems into large ones simply through delayed detection.

Markets change, as the backtesting book noted: a pattern that was genuinely real can fade as conditions shift, and a strategy that performed well for months can gradually or suddenly stop working without any dramatic, obvious signal announcing the change. Without periodic review of live performance against expectations, a strategy can quietly bleed capital for a considerable stretch before anyone notices.

Operational problems compound this risk. A connectivity issue, an exchange API change, or a subtle bug can cause a live system to behave incorrectly, and without monitoring, this can continue undetected for far longer than it should. Build regular review into your process, whether that's a daily check of key metrics or automated alerts that flag unusual behavior, so that "automated" means "the decision-making is systematic," not "nobody needs to pay attention anymore."

## 6. Chasing Complexity Instead of Understanding

A common beginner instinct is to assume that a more sophisticated approach, more indicators, more parameters, a more complex model, must produce a better strategy than a simple one. In practice, complexity without corresponding understanding tends to produce worse, not better, outcomes, largely through the overfitting mechanism covered in the backtesting book: every additional parameter or rule gives your backtest more opportunities to fit historical noise rather than a genuine, repeatable pattern.

Complexity also makes a strategy harder to debug and harder to reason about when something goes wrong live. A simple strategy you fully understand lets you diagnose unexpected behavior quickly, tracing exactly which rule triggered a given trade. A complex strategy with many interacting parts can behave in ways that surprise even the person who built it, making live problems much harder to catch and fix promptly.

This doesn't mean sophisticated techniques, including the machine learning approaches covered later in this library, have no place. It means earning the right to add complexity by first building genuine understanding and a working, well-tested simple version, then adding sophistication deliberately and incrementally, testing carefully at each step, rather than starting from maximum complexity because it seems more impressive.

## 7. Skipping the Testnet Because It Feels Like a Waste of Time

Testnet environments and paper trading, covered in the exchange API and backtesting books of this library, exist specifically to catch operational problems before real capital is exposed to them. New traders, eager to see real results after building something that backtests well, sometimes skip or rush through this stage, reasoning that the backtest already proved the strategy works.

This mistake misunderstands what testnet and paper trading are actually for. They don't re-test whether your strategy's logic is sound; the backtest already addressed that question, imperfectly but usefully. They test whether your system's connection to the real world, data feeds, order placement, error handling, reconnection logic, actually works as intended, which a backtest running against a clean historical file cannot verify at all.

The specific failures caught at this stage, a malformed order rejected due to a precision rule, a WebSocket disconnection your code doesn't detect and recover from, a race condition between two nearly simultaneous signals, are exactly the kind of operational risks the risk management book warned about, and they're dramatically cheaper to discover with fake testnet funds than with real capital on the line.

## 8. Letting Emotion Back Into an Automated System

The entire appeal of algorithmic trading, as the first book of this library described, is replacing emotional, inconsistent human decision-making with disciplined, tested rules. A subtle but common mistake undoes this benefit: manually overriding the system's decisions in the moment, based on a feeling, a headline, or simple impatience during a drawdown.

This isn't necessarily wrong in every single instance; sometimes a genuine, unanticipated event justifies human intervention. The mistake is doing it without a predefined rule for when intervention is and isn't appropriate, which the risk management book specifically recommended deciding in advance, while calm, rather than in the moment. Ad hoc overrides made during emotional moments tend to happen at exactly the worst times: pulling out of a strategy right before a drawdown recovers, or overriding a sell signal because you're hoping for a bounce that a disciplined backtest never assumed you'd wait for.

If you find yourself frequently tempted to override your system, that's valuable information, but the correct response is to examine and possibly revise your tested rules deliberately, through the same rigorous process used to build them in the first place, not to quietly bypass them in real time based on a feeling the whole system was designed specifically to remove from your decision-making.

## Summary

- A good-looking backtest is a starting point for scrutiny, not proof of a working strategy; actively try to break your own results.
- Build realistic trading costs into every backtest from the first version, never as an afterthought once you already like the numbers.
- A plausible-sounding idea still needs the same rigorous, out-of-sample testing as any other; intuition and evidence are separate questions.
- Size positions by a predefined, mechanical rule, never by how confident a given trade happens to feel.
- Automation still needs ongoing monitoring, testnet validation, and disciplined resistance to emotional, ad hoc overrides.

---

*This book is part of the Vizanix Quant Engineering Library. Licensed under CC BY 4.0 — free to read, share, and adapt with attribution to Vizanix.*
