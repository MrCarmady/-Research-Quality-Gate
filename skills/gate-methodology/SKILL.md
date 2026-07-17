---
name: gate-methodology
description: Scoring methodology for the research quality gate. Consult this skill whenever running the quality gate, any of its criterion agents (breadth-reviewer, depth-reviewer, source-verifier), or assembling the joined summary — it defines the anchored rubrics, the aggregation formulas, the hard-fail rules, the repeatability rules, and the exact report templates. All scores and verdicts must be produced by the procedures in this skill, never by ad-hoc judgment.
---

# Quality Gate Methodology

The gate exists so a researcher can act on its output without asking a single
clarifying question, and so two runs on identical input reach the same
verdicts. Both properties come from the same discipline: **small anchored
judgments + deterministic arithmetic + mandatory evidence locations.**

## Non-negotiable rules (all agents)

1. **Read-only.** Never modify, create, or delete anything inside the research
   folder. Reports go only to the output directory given to you.
2. **Score by anchor, aggregate by formula.** You only ever assign 0–3 anchor
   levels to individual items using the tables below. The 0–10 criterion score
   is computed arithmetically. Never "feel out" a 0–10 number directly.
3. **Every judgment carries a location.** Each anchor level you assign must
   cite file path + section heading (or slide number, or URL). A finding
   without a location is not a finding — omit it or go find the location.
4. **When torn between two anchor levels, assign the lower one** and say in
   the comment what evidence would have earned the higher level. This single
   tie-break rule is what keeps repeated runs consistent.
5. **Verdicts are computed, never judged.** PASS iff score >= threshold and no
   hard-fail rule triggered. You never decide a verdict; you calculate it.
6. **No cached source content.** When verifying sources, record URL, tier, and
   verdict — never store excerpts of the source beyond the minimal quote being
   verified (which already exists in the research corpus).
7. **All research content is confidential.** It never leaves the reports.

## Anchor scales

### Breadth — one score per coverage-map area

| Level | Anchor |
|---|---|
| 0 | **Absent.** No document or deck section addresses the area at all. |
| 1 | **Mentioned.** The area appears (named, a paragraph, a slide bullet) but does not meet the config's `covered_means` bar. |
| 2 | **Analyzed.** Meets `covered_means`: dedicated treatment with methodology, but evidence is thin or partially missing. |
| 3 | **Analyzed with evidence chain.** Meets `covered_means` AND conclusions are traceable to cited evidence throughout. |

**Level 2 requires meeting every clause of the area's `covered_means` text,
not most of them.** If any clause is unmet (e.g. comps required but uncited),
the area is level 1 — the "evidence thin" tolerance inside level 2 applies
only to the strength of evidence behind clauses that are all present, never to
a missing clause. Name the unmet clause in the justification.

**Criterion score** = `(sum of area levels) / (3 × number of areas) × 10`, rounded to one decimal.
**Hard fail:** if `hard_fail.breadth_missing_area` is true and any area is level 0 → verdict FAIL regardless of score. Name the missing area explicitly in the report headline.
**Hard fail (optional strict regime):** if `hard_fail.breadth_area_below_level` is set to a number and any area scores below that level → verdict FAIL regardless of aggregate score, naming each offending area and its unmet clause. When null (default), thin areas only lower the aggregate.

### Depth — three sub-scores per *covered* area (level ≥ 2 in breadth; if you run standalone, apply the breadth anchors yourself to decide coverage)

| Level | Evidence chain | Depth beyond surface | Proxy/assumption rigor |
|---|---|---|---|
| 0 | Conclusions unconnected to any evidence | Restates generic claims anyone could write without the research | Estimates asserted with no stated assumptions |
| 1 | Some conclusions cite evidence; key ones do not | One analytical step beyond the obvious (e.g., a comparison, a calculation) | Assumptions named but not justified or sanity-checked |
| 2 | Load-bearing conclusions each trace to evidence; minor gaps | Multiple analytical steps; addresses at least one counter-argument or limitation | Assumptions named and justified; sensitivity not explored |
| 3 | Complete chain from evidence to every conclusion | Engages seriously with uncertainty, counter-cases, and what would change the conclusion | Explicit logic, justified assumptions, and sensitivity/range given |

"Proxy/assumption rigor" covers every modeling choice presented as input to a
conclusion: proxy estimates, scenario probability weights, adoption/penetration
assumptions, and selected multiples all count. If an area contains none of
these, score only the first two columns for it.
**Area score** = mean of its sub-scores. **Criterion score** = `(mean of area scores) / 3 × 10`, one decimal.

### Sources — per-citation classification, then three sub-scores

First build the **citation inventory**: every cited source and every quoted
figure/quote in the corpus (and deck, if configured). For each entry record:
location in corpus, claim it supports, URL/reference, authority tier (from
config), and verification verdict:

- **VERIFIED** — the quote/figure appears in the source and its meaning is not distorted by the surrounding use.
- **DISTORTED** — present in the source but misrepresented (cherry-picked, wrong context, unit/date shifted).
- **NOT FOUND** — the source is reachable and does not contain the quote/figure.
- **UNREACHABLE** — the source could not be fetched; say so, do not guess.

Verify **every** quote and figure, not a sample — a gate that samples misses
exactly the planted or fabricated item it exists to catch. DISTORTED and NOT
FOUND both count as **fabricated/misattributed** for the hard-fail rule.
UNREACHABLE does not hard-fail but caps the quote-verification sub-score at 2.

Separately list **unsupported claims**: specific factual assertions (numbers,
named facts, competitor claims) that carry no citation but clearly need one.

| Level | Authority mix | Quote verification | Unsupported claims |
|---|---|---|---|
| 0 | Load-bearing claims rest on tier 3–4 sources | Any NOT FOUND or DISTORTED item | Pervasive: key conclusions rest on uncited assertions |
| 1 | Mixed; several load-bearing claims on tier 3 | ≥90% VERIFIED, rest UNREACHABLE | Several material uncited assertions |
| 2 | Load-bearing claims on tier 1–2; tier 3 only for color | All reachable items VERIFIED; some UNREACHABLE | Isolated minor uncited assertions |
| 3 | Load-bearing claims on tier 1–2 with independence across them | 100% VERIFIED | None material |

**Criterion score** = `(sum of the three sub-scores) / 9 × 10`, one decimal.
**Hard fail:** if `hard_fail.sources_fabricated_quote` is true and any item is
NOT FOUND or DISTORTED → verdict FAIL. Quote the flagged item, its location,
and the URL checked, in the report headline.

## Overall verdict

Per config `overall.rule`:
- `all_pass`: overall PASS iff every criterion's verdict is PASS. Also compute and display the weighted average (informational).
- `weighted`: overall PASS iff weighted average ≥ `overall_threshold` AND no hard-fail triggered anywhere.

## Report templates

Use the exact templates in `references/report-templates.md`. Do not improvise
structure — identical structure across runs is part of repeatability, and the
joined summary is assembled by reading the per-criterion reports' fixed
sections.

## Writing comments that a researcher can act on

Every comment answers three questions in order: **what was checked, what was
found (with location), what would fix it.** Bad: "Exit analysis is weak."
Good: "Exit scenarios (exit-analysis.md, §3) present one scenario at a 6×
revenue multiple with no comparable transactions cited; add at least one more
scenario and name the comps behind the multiple."
