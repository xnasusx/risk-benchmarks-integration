# Risk benchmarks integration

An engineering write-up of one change made across eleven repositories: replacing invented numbers
in a set of cyber risk tools with sourced ones, and finding out how little that data actually
covers.

![The FAIR decomposition tree with a real UK financial-services data-breach scenario loaded. Loss Event Frequency and Loss Magnitude are highlighted in amber carrying 0.43 / 0.65 / 0.69 events per year and £10K / £5.7M / £11.2M per event. The other eleven nodes — Threat Event Frequency, Susceptibility, Contact Frequency, Probability of Action, Threat Capability, Resistance Strength, Primary Loss, Secondary Loss and the rest — are empty. A panel below reads "2 of 13 nodes filled" and explains that a published loss study reports how often an event happens and how much it costs, and nothing else](docs/images/two-of-thirteen.png)

That picture is the finding. A published loss study fills exactly two nodes of the FAIR
decomposition, because frequency and magnitude are the only factors anyone measures across a
population. The other eleven are the analysis, and no dataset hands them to you.

## Why it exists

Interactive risk tools ship with invented numbers. The ranges have to come from somewhere, so the
author makes them up, labels them illustrative, and moves on. That is defensible for a teaching
tool and indefensible the moment anyone uses the output to decide something.

The fix is not better invented numbers. It is numbers that arrive with their provenance attached,
so a range backed by six citations and a range typed from memory do not look equally
authoritative.

What follows is what that took, what it cost, and where it stopped working.

## What was built

A data layer — [risk-benchmarks](https://github.com/RootCawsLLC/risk-benchmarks) — holding twelve
source-backed shards, 72 parameters, every one traced to a named public source with its
publication date, confidence level and stated limitation. It is generated from
[RiskShard](https://github.com/raviaxo/RiskShard), an evidence-governed cyber risk project, and
carries two provenance tiers so a shard with no practitioner review does not read like one that
has it.

Then eight tools were wired to it, and **each integration is different on purpose**:

| Tool | Relationship to the data |
| --- | --- |
| [risk-quantifier](https://github.com/RootCawsLLC/risk-quantifier) | Loads a shard as a starting range, per risk |
| [loss-exceedance-curve](https://github.com/RootCawsLLC/loss-exceedance-curve) | Same, and loading one switches currency across every axis and threshold |
| [cyber-materiality-workbench](https://github.com/RootCawsLLC/cyber-materiality-workbench) | A cited cross-check that **cannot move the verdict** |
| [incident-severity-calculator](https://github.com/RootCawsLLC/incident-severity-calculator) | Calibrates one OWASP factor against annual profit; **cannot move the score** |
| [fair-model-study](https://github.com/RootCawsLLC/fair-model-study) | Loads a shard onto the taxonomy to show how little of it fills |
| [monte-carlo-demo](https://github.com/RootCawsLLC/monte-carlo-demo) | Runs a real shard at four iteration counts to show which statistics settle |
| [ai-risk-register](https://github.com/RootCawsLLC/ai-risk-register) | **No picker.** A panel stating why, counted against the live corpus |
| [proofplane](https://github.com/RootCawsLLC/proofplane) | A cross-check in its loss-exposure report, vendored rather than fetched |

[proofscan](https://github.com/RootCawsLLC/proofscan) got documentation and no code at all.

The same file supported a prefill, a runtime fetch with currency switching, a read-only
cross-check barred from touching the answer, and a stated absence — without changing. That is the
strongest evidence the schema is right.

## The decisions worth explaining

**Not every tool should get the data.** `ai-risk-register` models *first-party* AI system risk:
prompt injection, data poisoning, model inversion, drift, agentic action without approval. The
only AI-related shard in the corpus is deepfake-enabled fraud — an attacker using AI *against*
you. Different threat class. Anchoring prompt injection to it would produce a citation without a
measurement, which is worse than an openly invented number. So that tool has no picker, and
instead states the gap and counts it against the corpus on every load: if a first-party AI shard
ever qualifies, the claim fails visibly rather than quietly.

**A proven exploit is not a loss estimate.** `proofscan` proves specific application flaws by
executing them. A finding describes one weakness in one codebase; a loss distribution describes
what incidents cost across a population. Multiplying them produces a number with a citation and no
measurement behind it. It got two paragraphs of README and no feature.

**Vendored, not fetched, where reproducibility matters.** `proofplane` hash-chains its evidence and
regenerates it nightly in CI. A build that reached the network for benchmark figures would make a
reproducible artefact depend on a third-party site staying up. Its copy is stamped with the
upstream commit and refreshed by hand.

**Absent is reported, never rendered as clean.** Where a shard has no practitioner statement of
what it is not good for, every consumer says the statement is *missing* rather than showing
nothing. Silence reads as "no caveats". In a materiality memo especially, a filing record must not
imply a caveat was considered and found unnecessary.

**Currency is displayed, never converted.** Four currencies, no FX table. The quantifier refuses a
mixed-currency portfolio total; the materiality workbench offers USD shards only; nothing silently
converts.

## What broke

Nine defects, and the method that caught each. Three came from composing a README screenshot rather
than from testing, which is the reusable lesson: rendering the thing at the size a reader sees it
is a verification step.

| Defect | Found by |
| --- | --- |
| A CUE schema draft used bare `min <= mode`, which embeds a boolean and does not compile | running `cue vet` |
| A money parser stripped only `$`, so any non-USD figure typed back became `NaN` and reverted | switching a shard's currency |
| Threshold pre-fills hardcoded at $5M/$2M/$20M — meaningless against a six-figure shard | as above |
| A `Loss per event ($)` label hardcoded while the field showed `£10K` | screenshot |
| A container capped at 500px, clipping the citation list mid-sentence | screenshot |
| `boxShadow: C.shadow` where that palette has no `shadow` key — the panel rendered flat | screenshot |
| The corpus builder silently dropped 23 sourced parameters, including the only AI shard, because they lacked a manifest | being asked to wire the AI tool |
| `--window-size=$W,$H` unquoted in PowerShell — split on the comma, ignored by Chrome, capture silently fell back to a 982×368 viewport | re-running the script |
| A formatter rounding above ten units printed a cited `$11,500,000` as `$12M` in a legal memo | reading the rendered output |

The last two are the interesting ones. A silent fallback to a smaller viewport produces an image
that looks like a deliberate crop, which is why it can sit in a repository unnoticed. And a
rounding rule that is right for a filer's own estimate is wrong for reproducing a published figure
in a document meant to be defensible — the same function, two different jobs.

## Claims that had to be corrected

**"The tail is still moving when the mean has stopped."** Written into the convergence panel, then
contradicted by its own output: at 50,000 iterations P99 came in tighter than P90. With twelve
repeats the spreads are themselves noisy and the column ordering is not stable. The copy now
claims the mechanism — a P99 is read off the worst one percent, so 10,000 iterations leave it
about a hundred events to stand on — and the panel says outright that pressing Run twice can
reorder the columns.

**"Nothing you type leaves the browser. There is no server."** True of the first clause and
imprecise after adding a fetch. That README now names all three outbound requests, states that
none carries anything entered, and says what to do if a policy forbids them.

**"Opening the file directly from disk works too."** A new cross-origin fetch looked like it would
break this. Driving real Chrome at a `file:` origin showed it does not — GitHub Pages sends
`Access-Control-Allow-Origin: *`, which matches a null origin. The claim was left alone rather
than hedged, because checking is cheaper than qualifying.

## Honest limits

**The corpus is small and the shards are starters.** Twelve shards, three threat types, mostly
financial services, mostly mid-market. Most are labelled governed starters rather than
benchmark-grade, meaning the evidence is real but no human review has signed them off.

**Nothing in it is high confidence.** Not one parameter of 72. Most are medium; the AI shard is
six-of-six low.

**Some frequencies are bridged from another country.** Where no local per-firm rate is published, a
shard borrows one that is — the US data-breach frequency comes from a UK survey. Every bridged
parameter says so in its own limitation, but a reader who skips the limitations will not notice.

**Within one shard, the three points can measure different things.** A survey average, a
claims-study mean, and a single documented incident used as a stress anchor produce a range wider
than any one source and endorsed by none of them.

**There is almost nothing for AI risk, and that did not change.** The public AI incident
catalogues record what happened without recording what it cost. One shard covers deepfake-enabled
fraud, and it is the weakest thing in the corpus. This work made that gap visible and measurable;
it did not close it.

**This is a write-up, not a method.** It describes what was done once, on one portfolio, by one
person. The decisions are defended, not validated.

## Reproducing it

The corpus and every tool are separate repositories, each with its own instructions. The data
layer regenerates from an upstream checkout:

```bash
git clone https://github.com/RootCawsLLC/risk-benchmarks && cd risk-benchmarks && pip install -r requirements.txt && git clone --depth 1 https://github.com/raviaxo/RiskShard && python build_benchmarks.py RiskShard risk-benchmarks.json
```

The build reports coverage and warns on any parameter it cannot trace to a source. The current run
resolves 72 of 72.

## Attribution

The underlying shards come from [RiskShard](https://github.com/raviaxo/RiskShard) by
[raviaxo](https://github.com/raviaxo), AGPL-3.0 — an evidence-governed cyber risk quantification
project where every parameter traces to a reviewed public source. Figures, limitations and
"not good for" statements are RiskShard's, carried through unchanged.

The shards cite public sources including the UK Cyber Security Breaches Survey, FBI IC3, the
Australian Bureau of Statistics, ACCC Scamwatch, Statistics Canada, CNIL, Japan's National Police
Agency, the Singapore Police Force, NetDiligence, Sophos, IBM and Verizon. None is reproduced
here; see the corpus for per-parameter citations.

FAIR is a model published by the [FAIR Institute](https://www.fairinstitute.org/). The FAIR
Model™ is their trademark. This project does not reproduce their standards.

## License

Copyright (c) 2026 RootCaws LLC.

[GNU AGPL v3 or later](LICENSE). If you modify this and run it as a network service, the AGPL
requires you to offer your users the modified source under the same terms.
