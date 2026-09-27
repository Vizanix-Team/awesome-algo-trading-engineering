# Machine Learning for Market Data: First Principles

*By Vizanix — Beginner Level*

> This book lays out the practical, disciplined workflow for applying machine learning to market data honestly, from framing the problem correctly to evaluating a model without fooling yourself.

![diagram](../../assets/neural-network.svg)

## Table of Contents

1. What Machine Learning Actually Adds Here
2. Framing the Prediction Problem Correctly
3. Features: Turning Market Data Into Model Inputs
4. Labels: Defining What You're Actually Predicting
5. Splitting Data Without Cheating Yourself
6. Choosing a Model: Simpler Is a Feature, Not a Limitation
7. Evaluating a Model Honestly
8. From a Trained Model to a Trading Decision

## 1. What Machine Learning Actually Adds Here

Machine learning, broadly, is a set of techniques where a program learns patterns directly from data rather than following rules a human wrote out explicitly. The neural network book in this library covered one specific, powerful family of these techniques in detail; this book steps back and covers the broader workflow that applies whether your specific model is a neural network, a simpler statistical technique, or something else entirely.

The appeal of machine learning for market data is straightforward: rather than manually hypothesizing "the twenty-period moving average crossing the fifty-period moving average predicts a rise," as in the spreadsheet-to-system book, you let an algorithm search across many possible combinations of inputs and find whatever pattern best predicts your target outcome in historical data. This can uncover relationships a human wouldn't have thought to test manually.

The catch, which this entire book is really about, is that "found a pattern in historical data" and "found a genuine, repeatable, exploitable pattern" are very different claims, and the gap between them is wider and easier to fall into in trading than in almost any other application of machine learning, for reasons the statistics-for-trading and neural network books of this library already began to explain. This book walks through the concrete, practical steps that keep that gap from swallowing your project.

## 2. Framing the Prediction Problem Correctly

Before touching any data, define precisely what you're trying to predict, at what point in time you'd know the inputs, and at what point in time you'd know whether the prediction was right. This sounds obvious, but vague framing here is the single most common source of the look-ahead bias the backtesting book of this library warned about, now hiding inside a machine learning pipeline instead of a hand-written strategy rule.

A well-framed problem looks like this: "using only data available up through the close of trading today, predict whether tomorrow's closing price will be higher than today's." Notice this specifies exactly what information the model can use, exactly what it's predicting, and exactly when that prediction would be checkable against reality. A poorly framed problem, like "predict whether the price will rise," without pinning down the exact information cutoff and the exact prediction horizon, leaves room for inputs that subtly include information from after the point you'd actually be making the prediction in real life.

Write this framing down explicitly before writing any code, exactly as the spreadsheet-to-system book recommended writing strategy rules down in plain language first. A machine learning pipeline built on a vaguely framed problem tends to quietly develop look-ahead bias somewhere in its feature construction, and catching that after the fact is far harder than preventing it by being precise from the very beginning.

## 3. Features: Turning Market Data Into Model Inputs

Features are the numerical inputs you hand to your model, and constructing them well matters at least as much as which specific model you eventually choose. Good features summarize market data in ways that plausibly carry real information about your target outcome: recent returns over multiple periods, measures of recent volatility from the statistics-for-trading book, order book imbalance from the reading-the-order-book book, or volume patterns from the market data book.

Every single feature must respect the information cutoff you defined in the previous section. If you're predicting tomorrow's close using only information available through today's close, then a feature computed using any data from tomorrow, even data that seems unrelated to price, like a volume figure, is invalid and will produce exactly the look-ahead bias the backtesting book warned about, just embedded inside a feature calculation rather than an obvious strategy rule.

Resist the urge to construct an enormous number of features hoping some of them prove useful. Each additional feature gives your model more room to find and exploit coincidental patterns in your specific historical sample, exactly the overfitting risk the neural network book described at length. A smaller set of features you can individually justify with a plausible economic or behavioral reasoning, tied to concepts like liquidity, order flow, or volatility covered earlier in this library, tends to generalize far better than a huge, unfiltered pile of everything you could compute.

## 4. Labels: Defining What You're Actually Predicting

The label is the known, correct answer for each historical example, the thing your model is trained to predict, corresponding to the "correct answer" in the neural network book's description of training. Defining labels for market data carries its own specific pitfalls worth understanding clearly.

A label like "did the price rise over the next period" seems simple, but consider what happens near the edges of your dataset, or during a period with a data gap, discussed in the market data book of this library. A label computed incorrectly at these edges, perhaps accidentally using a stale or missing price, introduces quiet corruption into your training data that can be very hard to spot after the fact, since the model will simply learn from whatever labels you handed it, wrong or not, without any way of flagging the mistake itself.

Consider also whether a binary label, "up" or "down," discards useful information a more detailed label would keep, like the actual magnitude of the move, and whether trading costs mean a small predicted move isn't worth acting on even if the direction prediction were correct. Building your labels to reflect what you'd actually trade on, factoring in costs from the backtesting book of this library, rather than an idealized, cost-free notion of "up" or "down," produces a model whose apparent accuracy translates more honestly into real trading value.

## 5. Splitting Data Without Cheating Yourself

The backtesting book's discussion of in-sample and out-of-sample data applies here with extra force, because machine learning models are specifically designed to find and fit patterns in whatever data you show them, which is exactly the mechanism that makes overfitting so easy if you're not careful. Split your historical data into a training set, used to fit the model's parameters, and a genuinely separate test set, touched only once, at the very end, to check how the model performs on data it never saw during training.

For market data specifically, this split must respect chronological order: train on an earlier period, test on a strictly later period, never on data shuffled randomly across time. Randomly shuffling before splitting, a common default in general machine learning tutorials that don't consider time series, would let information from the future leak into your training process indirectly, since nearby time points in market data are often correlated with each other, producing an optimistic, misleading test result.

A further refinement, called a validation set, gives you a third, separate slice of data for making decisions about the model itself, like choosing how many layers a neural network should have, without touching your final test set until every such decision is already locked in. This matters because if you repeatedly check your test set performance while adjusting the model, you've effectively turned the test set into part of your training process, through the same data snooping mechanism the statistics-for-trading book of this library warned about, quietly undermining the very independence that made it useful as a check in the first place.

## 6. Choosing a Model: Simpler Is a Feature, Not a Limitation

With features and labels defined and data properly split, you face a choice of what type of model to actually train. Simpler models, ones with fewer adjustable parameters, are generally easier to understand, easier to debug, and less prone to the severe overfitting risk the neural network book described, precisely because they have less room to fit coincidental noise in your training data.

A sensible default for a beginner working with market data is to start with the simplest model that can plausibly capture the relationship you're investigating, and only move to something more complex, like the neural networks covered in the previous book, if the simpler model demonstrably underperforms and you have good reason to believe a more flexible model would genuinely help rather than just overfit more elaborately.

Whatever model you choose, prioritize being able to inspect and understand why it makes the predictions it makes. A model whose behavior you can partially explain lets you sanity-check its logic against what you actually know about markets from earlier books in this library, catching cases where it has seemingly learned something implausible, like weighting a feature in a direction that makes no economic sense, which often signals a subtle bug or a spurious, overfit pattern rather than a genuine insight.

## 7. Evaluating a Model Honestly

Evaluating a machine learning model for trading needs more than the general accuracy measures common in other machine learning applications. A model can achieve seemingly impressive accuracy on a simple "up or down" prediction while still being useless or even harmful once you account for trading costs, because it might be highly accurate specifically on the small, easy moves and unreliable on the larger moves that would actually be worth trading after costs.

Translate your model's predictions into simulated trading decisions and run them through the same rigorous backtesting process from the dedicated backtesting book of this library, including realistic slippage and commission assumptions from the very start. This step, converting raw prediction accuracy into an actual simulated profit and loss curve, often reveals that a statistically interesting model doesn't actually produce a viable trading strategy once real-world frictions enter the picture.

Apply the sample size caution from the statistics-for-trading book directly here as well: a model tested against a small number of independent out-of-sample predictions, even if each individual prediction looks impressively accurate, doesn't give you strong evidence of a genuinely reliable edge. Insist on a reasonably large, genuinely out-of-sample test period, spanning varied market conditions, before drawing any real conclusion about a model's practical value.

## 8. From a Trained Model to a Trading Decision

A model that survives honest evaluation, with realistic costs and a properly sized, genuinely out-of-sample test, earns the same cautious next steps as any other strategy covered in this library: paper trading or testnet deployment from the exchange API and crypto connectivity books, explicit risk management rules from the dedicated risk management book, and a gradual, deliberate path to any real capital, exactly as the spreadsheet-to-system book laid out for a simpler rule-based strategy.

Keep monitoring the model's live performance against its backtested and validated expectations on an ongoing basis, since the regime dependence covered in the statistics-for-trading book applies to a machine learning model just as much as to any simpler rule: a pattern the model learned from historical data can fade or shift as market conditions change, and only active monitoring catches this in time to matter.

Treat a machine learning model as one more tool in the toolkit this library has built up, subject to exactly the same discipline, honest testing, realistic costs, careful risk management, as every other strategy type covered here, never as a shortcut that exempts you from that discipline simply because it sounds more sophisticated or arrived at its conclusions through a more elaborate process.

## Summary

- Machine learning finds patterns from data automatically, but a pattern found in history is not automatically a genuine, repeatable edge.
- Frame the prediction problem precisely, specifying the information cutoff and prediction horizon, before writing any feature code.
- Every feature and label must respect that information cutoff exactly, or look-ahead bias creeps into the pipeline invisibly.
- Split data chronologically into training, validation, and a test set touched only once, never shuffled randomly across time.
- Evaluate through realistic, cost-inclusive backtesting and a sufficiently large out-of-sample period, not raw prediction accuracy alone.

---

*This book is part of the Vizanix Quant Engineering Library. Licensed under CC BY 4.0 — free to read, share, and adapt with attribution to Vizanix.*
