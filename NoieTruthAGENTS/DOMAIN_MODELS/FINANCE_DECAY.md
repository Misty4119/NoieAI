# FINANCE_DECAY.md

## Financial-claim freshness review

Financial claims can depend on changing market conditions, product terms, accounting periods, regulations, and an individual's circumstances. This document defines a review checklist; it provides no investment advice, market forecast, decay constant, or trading rule.

## Freshness record

For each time-sensitive financial claim, record the source and publication/effective date, the date checked, relevant market or jurisdiction, instrument/product and time horizon, assumptions, and what event would invalidate the claim. Verify current prices and terms from the appropriate primary source before use. Historical returns and backtests do not guarantee future results.

Do not assign a universal exponential decay rate or half-life to financial knowledge. Review intervals should depend on the claim's source, update cycle, decision horizon, and the cost of stale information. Mark information stale or unverified when its current status is unknown. Regulatory interpretation must be checked against current authoritative sources for the applicable jurisdiction.

No financial data feed, forecasting model, regulatory monitor, or personalized suitability assessment is included in this Markdown repository. An information freshness status does not imply that an investment decision is suitable or permitted.
## Claim classes and invalidation events

| Claim class | Freshness questions and common invalidation events |
| --- | --- |
| Market quote or yield | Exchange, instrument, timestamp, trading status, delayed-feed status, and corporate action |
| Fund, deposit, or insurance product terms | Issuer prospectus or contract version, fee changes, eligibility, withdrawal conditions, and jurisdiction |
| Company results or balance-sheet fact | Reporting period, restatement, filing status, accounting basis, and later material disclosure |
| Historical return or valuation assumption | Data window, currency, benchmark, survivorship and look-ahead bias, inflation treatment, and model assumptions |
| Tax or financial regulation | Jurisdiction, effective date, transitional rule, official amendment, and authoritative interpretation |
| Personal suitability statement | Current objectives, time horizon, liquidity needs, risk constraints, tax status, and verified user-provided information |

These are review prompts; they do not define a universal expiry period. A current price should be checked close to the decision time. A historical filing remains a historical fact after it becomes old, while its use as evidence about present solvency may become stale. Separate “the report stated X on date t” from “X is true now.”

## Review disposition

Mark a claim CURRENTLY_CHECKED only when the relevant authoritative source and effective date were checked for this decision. Otherwise use NOT_CHECKED, STALE, SOURCE_UNAVAILABLE, or SCOPE_MISMATCH. Record the event or source that could update the claim and any uncertainty about delayed or revised data.

Do not translate freshness into investment confidence or an expected return. A current quote does not establish fair value, suitability, or future performance. Personal recommendations require applicable policy, relevant current context, and appropriate authorization; this freshness module supplies none of those.
