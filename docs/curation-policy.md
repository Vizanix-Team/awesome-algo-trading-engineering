# Editorial Policy

This document describes how the Vizanix Quant Engineering Library is written, structured, and maintained. It replaces the earlier curation policy that governed an external links list; this repository no longer links out to third-party books, papers, courses, or projects — everything in `library/` is original Vizanix work.

## Scope

The library covers the engineering and research side of algorithmic trading:

- Market microstructure and market data
- Exchange connectivity and execution
- Backtesting and simulation
- Risk management
- Production engineering, observability, and testing
- Quantitative research tooling
- Machine learning and neural networks as applied to markets
- Crypto-specific infrastructure

Alongside the trading library, the `library/en/ai-society/` and `library/ru/ai-society/` trees hold a separate, level-agnostic collection of essays reasoning about AI's broader trajectory: existential and near-term risk, economic and social disruption, alignment, governance, and open philosophical questions. These are analytical and exploratory, not activist — they weigh competing views rather than argue a single conclusion, and they don't make policy recommendations on behalf of any real government, company, or political actor.

Neither collection covers trading strategies, signals, or performance claims of any kind, at any level.

## Structure

Every book belongs to exactly one language tree and one level:

- `library/en/` and `library/ru/` — language.
- `beginner/`, `intermediate/`, `professional/` — level, inside each language tree.

**Beginner** books assume no prior trading or finance background and define every term on first use. **Intermediate** books assume working trading vocabulary and programming ability, and focus on design tradeoffs. **Professional** books assume a practicing production engineer and go deep on edge cases, failure modes, and rigorous treatment, including formulas or pseudocode where they clarify a mechanism.

The library intentionally weights toward beginner material, since the widest audience is engineers and researchers just entering the field.

## Editorial standards

Every book in the library must:

1. Be written entirely by Vizanix, with no content copied or closely paraphrased from an external source.
2. Avoid attributing ideas to real named authors, papers, or companies — general technical facts (what a limit order book is, what TWAP means) are described in Vizanix's own words and examples, not sourced to a specific outside work.
3. Use clearly hypothetical, illustrative examples rather than presenting fabricated data as real-world case studies.
4. Avoid unrealistic profit claims, "guaranteed returns" language, or anything resembling a trading signal.
5. Follow the standard book format: title, byline, one-line abstract, table of contents, numbered chapters, summary, and the CC BY 4.0 license footer.
6. Include diagrams from `library/assets/` only where they genuinely clarify the content, not as decoration.

## Corrections and translations

Corrections (factual errors, outdated code, unclear explanations) and translations between English and Russian are the main way the community contributes. See [CONTRIBUTING.md](../CONTRIBUTING.md) for the process. A correction is evaluated on whether it makes an existing book more accurate or clearer, not on introducing new external material.

## New books

New book proposals go through an issue, but the writing itself is done by Vizanix to keep voice, structure, and licensing consistent across sixty-plus books. A proposal that identifies a genuine gap (a level or subject not yet covered) is the most useful kind.

## Removal

A book is retired or rewritten when it becomes materially inaccurate (e.g., describes a protocol or practice that has changed in a way that misleads readers) or is superseded by a clearer replacement covering the same ground. Retirement happens via pull request, same as any other change, so it stays visible in the project history.
