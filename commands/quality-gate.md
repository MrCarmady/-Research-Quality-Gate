---
description: Run the research quality gate against a research folder (and optional deck). Produces one report per criterion (breadth, depth, sources) plus a joined summary with scores, thresholds, and pass/not-pass verdicts.
argument-hint: <research-folder-path> [deck-path]
---

Run the research quality gate. Arguments: $ARGUMENTS — the first path is the
research folder; an optional second path is the draft deck (if omitted, look
for an obvious deck inside the folder: .pptx, or a file named like
deck/presentation; if several candidates exist, ask which one rather than
guessing).

Read the `gate-methodology` skill before anything else; it governs scoring,
verdicts, and report structure for this entire run.

## 1. Resolve inputs and config

- Verify the research folder exists and is non-empty. If not, stop and say so
  — never invent an evaluation.
- Config resolution: if `gate-config.yaml` exists in the research folder, use
  it; otherwise use the plugin default at `config/gate-config.yaml`. Record
  which one was used — every report states its config source, so a threshold
  change is always traceable to a file.
- Compute the output directory from the config's `output.directory` pattern
  (expand `{research_folder}` and `{timestamp}` as ISO-8601, e.g.
  `2026-07-16T14-30-00`) and create it. Reports are written ONLY there. The
  research folder is strictly read-only for this entire run: no edits, no new
  files inside it, no reorganizing — if something looks misplaced, that is a
  finding, not a fix.

## 2. Dispatch the criterion agents

Launch as subagents, passing each: research folder path, deck path, the
resolved config (path and relevant contents), and the output directory.

- Launch `breadth-reviewer` and `source-verifier` in parallel (the source
  verifier is the slow one — it fetches every cited URL).
- Launch `depth-reviewer` when the breadth report exists, passing its path so
  depth scores exactly the areas breadth found covered.

Wait for all three reports: `report-breadth.md`, `report-depth.md`,
`report-sources.md`. If an agent fails or a report is missing its
machine-readable GATE-RESULT block, re-run that agent once; if it fails again,
stop and report the failure honestly — a partial gate must never present
itself as a completed one.

## 3. Assemble the joined summary

- Parse the GATE-RESULT block from each report. Use only those parsed values —
  do not re-derive or "adjust" any score.
- Compute the weighted average and the overall verdict exactly per the
  config's `overall` section (`all_pass`: overall PASS iff all three PASS;
  `weighted`: compare weighted average to `overall_threshold`, and any
  hard-fail anywhere forces FAIL).
- Write `report-summary.md` to the output directory using the joined-summary
  template: the criterion table, blocking issues collected verbatim from the
  three reports, the merged impact-ordered action list, and the input
  inventory.

## 4. Present the result

Reply in chat with: the overall verdict, the criterion table (score /
threshold / verdict), the single most important blocking issue if any, and the
paths to the four reports. Keep it short — the reports carry the detail.

Confidentiality: all of this is Delin Capital private research. Never send its
content anywhere other than the output directory and this chat.
