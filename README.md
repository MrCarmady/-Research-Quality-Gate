# Research Quality Gate — Cowork plugin

An automated review step that scores a research corpus and draft deck before
they go to the principal. Three criterion agents (breadth, quality & depth,
source trustworthiness) each produce an evidence-based report with a numeric
score; an orchestrator joins them into a summary with threshold comparisons
and explicit pass / not-pass verdicts. Strictly read-only on the research
materials.

## Install

1. Put this folder somewhere stable (or push it to a git repository — private
   repos work for internal distribution).
2. In Claude Desktop / Cowork: **+ → Plugins → Add plugin**, point at the
   folder or repo.
3. Confirm `/quality-gate` appears in the slash-command list.

## Use

```
/quality-gate /path/to/research-folder [optional /path/to/deck.pptx]
```

Output lands in `<research-folder>-quality-gate/<timestamp>/`:

- `report-breadth.md`, `report-depth.md`, `report-sources.md`
- `report-summary.md` — overall verdict, per-criterion table, blocking issues,
  impact-ordered fix list

## Configure (no rebuild ever needed)

All project specifics live in one YAML file. The plugin ships a default at
`config/gate-config.yaml`; a `gate-config.yaml` placed inside the research
folder overrides it. It controls:

- **coverage_map** — the required research areas and what "covered" means
- **thresholds** — per-criterion pass bars (0–10)
- **hard_fail** — missing coverage area / fabricated quote auto-fail rules
- **overall** — all-must-pass (default) or weighted-average verdict rule
- **deck** — which criteria evaluate the deck
- **authority_tiers** — source-quality tier definitions
- **output.directory** — where reports go

Change a threshold, re-run, verdicts update. To reuse on the next project,
copy the config into that project's folder and edit the coverage map.

## How repeatability is achieved

Agents never pick a 0–10 score. They assign small anchored 0–3 levels from
fixed rubric tables (in `skills/gate-methodology/`), with a lower-when-torn
tie-break; arithmetic converts levels to criterion scores; verdicts are pure
score-vs-threshold comparisons plus hard-fail rules. Report structure is
template-locked, and the summary is assembled by parsing machine-readable
blocks from the criterion reports, not by re-judging.

## Acceptance test procedure

Run these against a copy of a real (or synthetic) corpus before relying on
the gate:

1. **Happy path** — full corpus + deck → four reports exist, each with score,
   threshold, comments with file locations, and a verdict.
2. **Breadth trap** — remove one coverage area's document(s) → breadth FAILs
   and the Verdict line names the missing area.
3. **Fabrication trap** — plant one quote attributed to a real URL that does
   not contain it → sources FAILs, the report quotes the item, its location,
   and the URL checked.
4. **Threshold flip** — drop `gate-config.yaml` into the folder with a
   threshold moved across a known score → verdict flips on re-run; report's
   `config_source` shows the folder config.
5. **Repeatability** — run twice on identical input → identical verdicts on
   all criteria and overall (scores materially consistent).
6. **Actionability** — hand the reports to the researcher; they should locate
   and fix every named issue without asking a clarifying question.

## Layout

```
research-quality-gate/
├── .claude-plugin/plugin.json      manifest
├── commands/quality-gate.md        /quality-gate orchestrator
├── agents/
│   ├── breadth-reviewer.md
│   ├── depth-reviewer.md
│   └── source-verifier.md
├── skills/gate-methodology/
│   ├── SKILL.md                    rubrics, formulas, hard rules
│   └── references/report-templates.md
├── config/gate-config.yaml         default config (overridable per folder)
└── README.md
```

## Notes

- The gate never modifies, adds to, or reorganizes the research folder;
  reports are written to a sibling directory.
- Sources are referenced by link and verified by fetching; source content is
  never cached into reports.
- All research content is confidential to Delin Capital.
