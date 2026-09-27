# Curation Policy

This document describes how resources are evaluated for inclusion in, and removal from, Awesome Algorithmic Trading Engineering. It expands on the summary in the [README](../README.md#what-belongs-here).

## Scope

This list covers the engineering, infrastructure, and research tooling side of algorithmic trading:

- Market microstructure and market data
- Exchange connectivity and execution
- Backtesting and simulation
- Risk management
- Production engineering, observability, and testing
- Quantitative research tooling, including ML applied to markets
- Crypto-specific infrastructure

It explicitly does **not** cover trading strategies, signals, or performance claims of any kind.

## Inclusion criteria

A resource should meet most of the following to be added:

1. **Technically substantial.** It explains, implements, or documents something concrete — a protocol, an algorithm, a library, a methodology. It is not a marketing page.
2. **Practically useful.** Someone building, testing, researching, or operating trading infrastructure would plausibly use or reference it.
3. **Maintained or historically important.** Actively maintained software/documentation is preferred; unmaintained resources may still qualify if they remain a standard reference (e.g., a foundational paper or book).
4. **Transparent.** Commercial resources disclose pricing or access model where relevant; nothing hides its actual purpose.
5. **Relevant.** Directly related to algorithmic trading engineering or a closely adjacent discipline (distributed systems, low-latency engineering, applied statistics) with a clear tie back to trading infrastructure.
6. **Not primarily promotional.** The resource's main purpose is not to sell a product, course, or signal service.
7. **No unrealistic claims.** No "guaranteed returns," "100% win rate," or similarly unfalsifiable performance language.
8. **No spam patterns.** No obvious affiliate chains, referral codes, or SEO-farm content.
9. **Not disguised signal-selling.** Educational framing does not exempt a resource whose actual product is trading signals or copy-trading.

## Exclusion criteria

The following are excluded outright, regardless of framing:

- Strategy or signal-selling content, including "free" signal groups used as funnels.
- Copy-trading platforms or their promotional content.
- Pump-and-dump or coordinated trading communities.
- Referral-link collections.
- Content generated primarily to manipulate search rankings, with no original technical substance.
- Unverified "holy grail" strategies presented without methodology, code, or peer review.
- Tools or repositories whose primary purpose is credential theft, account takeover, exchange abuse, or other malicious activity.

## Evaluation process

1. A contributor submits a resource via pull request or the **Suggest a resource** issue template, including the disclosures described in [CONTRIBUTING.md](../CONTRIBUTING.md).
2. A maintainer checks the submission against the inclusion and exclusion criteria above.
3. If the resource qualifies, it is merged into the appropriate section, matching the existing description style (neutral, specific, no superlatives).
4. If it does not qualify, the PR/issue is closed with a brief explanation referencing the specific criterion it fails.
5. Borderline cases (genuinely useful but partially promotional, for example) are discussed openly in the PR/issue rather than silently accepted or rejected.

## Removal criteria

A resource may be removed when:

- The link is dead and no canonical replacement URL exists.
- The underlying project is abandoned **and** has been meaningfully superseded by a better-maintained alternative already in the list.
- The resource's nature has changed since inclusion (e.g., a previously neutral blog pivoted to signal-selling or affiliate marketing).
- It's later discovered that the resource violates the exclusion criteria (e.g., undisclosed affiliate relationship, misrepresented authorship).
- A rights holder requests removal for a valid reason.

Removal is done via pull request, same as addition, so the change is visible and reviewable in the project's history. Automated tooling (see [the link-check workflow](../.github/workflows/links.yml)) flags candidates for review — it does not remove resources automatically.

## Placeholder markers

Where a resource is known to exist but its canonical URL isn't confirmed, the list uses an explicit marker:

```markdown
<!-- VERIFY RESOURCE -->
```

These markers are tracked as open items — see the [First 10 issues](../README.md) tracked at launch — and should be resolved (verified and linked, or removed) rather than left indefinitely.

## Vizanix's role

Vizanix maintains this repository as a public resource and applies this policy consistently, including to any resource submitted by Vizanix team members, which is held to the same criteria as any other submission.
