# Report templates

Use these templates exactly. Placeholders in {braces}. Omit nothing; if a
section has no content, write "None." The machine-readable block at the top of
each report is what the orchestrator parses to build the joined summary — keep
its field names and format byte-exact.

---

## Per-criterion report — file name: `report-{criterion_id}.md`

```markdown
# Quality Gate — {Criterion Name}

```yaml
# GATE-RESULT (machine-readable; do not edit)
criterion: {criterion_id}          # breadth | depth | sources
score: {score}                     # 0–10, one decimal
threshold: {threshold}
hard_fail_triggered: {true|false}
hard_fail_reason: "{one line, or empty}"
verdict: {PASS|FAIL}
run_timestamp: "{ISO-8601}"
config_source: "{path of the config file actually used}"
```

## Verdict
**{PASS | FAIL}** — score {score} vs threshold {threshold}.{" Hard-fail rule triggered: {reason}." if applicable}

## What was checked
{2–5 sentences: inputs inventoried (file count, deck), the rubric applied, and
the procedure followed. For sources: how many citations/quotes were in the
inventory and how many were fetched.}

## Scoring detail
{The criterion-specific anchor table, one row per scored item, with columns:
Item | Level(s) assigned | Location | One-line justification.
Follow with the aggregation arithmetic written out, e.g.
"(3 + 2 + 2 + 0) / 12 × 10 = 5.8".}

## Findings
{Numbered list. Each finding: what was checked → what was found → exact
location (file + section / slide / URL) → severity (BLOCKER if it triggers or
contributes to a FAIL, MAJOR if it costs ≥1 anchor level, MINOR otherwise).}

## What would raise the score
{Numbered list, ordered by impact. Each entry names the concrete change and
the anchor level it would unlock, e.g. "Add comparable transactions behind the
6× multiple (exit-analysis.md §3): rigor 1 → 2."}
```

---

## Joined summary — file name: `report-summary.md`

```markdown
# Quality Gate — Joined Summary

```yaml
# GATE-SUMMARY (machine-readable; do not edit)
overall_verdict: {PASS|FAIL}
overall_rule: {all_pass|weighted}
weighted_average: {0–10, one decimal}
overall_threshold: {value, if rule is weighted}
criteria:
  breadth:  { score: {x}, threshold: {t}, verdict: {PASS|FAIL}, hard_fail: {true|false} }
  depth:    { score: {x}, threshold: {t}, verdict: {PASS|FAIL}, hard_fail: {true|false} }
  sources:  { score: {x}, threshold: {t}, verdict: {PASS|FAIL}, hard_fail: {true|false} }
run_timestamp: "{ISO-8601}"
input_folder: "{path}"
config_source: "{path}"
```

## Overall verdict
**{PASS | FAIL}** under the `{rule}` rule.

| Criterion | Score | Threshold | Verdict | Hard fail |
|---|---|---|---|---|
| Breadth | {x} | {t} | {PASS/FAIL} | {—/reason} |
| Quality & depth | {x} | {t} | {PASS/FAIL} | {—/reason} |
| Sources & quotes | {x} | {t} | {PASS/FAIL} | {—/reason} |
| **Weighted average** | **{x}** | {overall_threshold or "informational"} | | |

## Blocking issues
{Every BLOCKER finding from the three reports, verbatim, with a pointer to its
report. If none: "None."}

## Top actions to reach PASS
{Merged, impact-ordered list drawn from the three "What would raise the score"
sections. Cap at 10. If overall verdict is PASS: the top 3 improvements anyway.}

## Inputs evaluated
{File inventory of the research folder (names only, no content), the deck
identified, and which criteria evaluated the deck per config.}
```
