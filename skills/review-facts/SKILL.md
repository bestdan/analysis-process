---
name: review-facts
description: Independent fact-check of a completed analysis pipeline — verifies links resolve, cited values match their sources, model output is reproducible, narrative numbers trace to the model, units and formulas are correct, and the recommendation matches what the data actually shows. Use when an analysis (model.py + model_output.json + memo.filled.md, or equivalent) is complete and needs an audit before it ships.
user-invocable: true
---

# Review Facts

A reviewer-side skill. The author of the analysis is one agent; this skill is run by a **separate agent** that has not seen the analysis's construction. The reviewer reads the artifacts cold and verifies them against their sources and against each other.

## When This Applies

Run after an analysis pipeline (per `analysis-pipeline`) is complete:

- a model file (e.g. `model.py`) with sourced inputs and derived values,
- a structured output (e.g. `model_output.json`),
- a narrative document (e.g. `memo.filled.md`) and its template.

It also applies to lighter analyses — single-script computations, ad-hoc memos with cited numbers — where the same checks are still meaningful.

Does NOT apply to drafts in progress, exploratory notebooks, or conversational answers.

## Core Principle

**Independence.** The reviewer must not inherit the author's reasoning. Read the artifacts as a stranger would, fetch sources fresh, and recompute derived values without consulting the author's narrative for hints. If you (the agent invoked by the user) wrote or helped write the analysis being reviewed, **delegate the review to a fresh subagent** (Agent tool, `general-purpose`) so the review is done with no prior context. Pass the subagent only the artifact paths and this checklist, not your reasoning about the analysis.

## What To Review

Locate the analysis directory. Identify:

- the model file(s) (e.g. `model.py`)
- the model output (e.g. `model_output.json`)
- the template(s) (e.g. `memo.template.md`)
- the filled document(s) (e.g. `memo.filled.md`)
- any `inputs/` directory with raw source data

If any of these are missing or unclear, ask before proceeding.

## The Checklist

Run every applicable check. For each, record `pass`, `fail`, or `n/a` with a one-line reason. Do not stop at the first failure — collect all findings.

### 1. Links resolve

For every URL in source comments, model code, JSON `source` fields, and the filled document:

- Fetch the URL (WebFetch, curl, or equivalent). Treat 4xx/5xx as failures. Treat redirects to login pages, parked domains, or unrelated content as failures.
- For scheme-less citations (e.g. `nimbus.io/pricing` rather than `https://nimbus.io/pricing`), try `https://` then `http://` before flagging as broken.
- For non-URL citations (local PDFs, internal docs, "vendor email 2025-11-12", spec sheets), confirm the file exists at the cited path or note that the source is offline-only and cannot be auto-verified — do not flag as a dead link.
- Note the date you checked.

### 2. Cited values match their sources

For every input whose comment cites a URL or document:

- Fetch the source.
- Confirm the cited value (price, rate, spec, date) actually appears at the source — or, if it's computed from the source, confirm the computation.
- Flag mismatches with both the cited value and the value found at the source.

### 3. Output reproducibility

- Re-run the pipeline (`uv run model.py`, `uv run fill_templates.py`, etc.).
- Diff regenerated `model_output.json` against the committed copy (normalize with `jq -S .` if available, so key ordering or whitespace differences don't show as drift).
- Diff regenerated `memo.filled.md` against the committed copy in full — `fill_templates.py` leaves `{{narrative:*}}` placeholders untouched, so the filled memo is deterministic.
- If a separately-generated narrative-filled artifact exists (e.g. `memo.final.md` produced by piping `memo.filled.md` through Claude CLI), treat that one as nondeterministic: diff only the data-bearing portions, not the prose.
- Any drift in the deterministic artifacts means the committed copies are stale relative to the code.

### 4. Numbers in the narrative trace to the model

- For every numeric value, currency amount, percentage, date, or named quantity in the filled document, confirm it appears in `model_output.json` (or is a verbatim copy of an input documented there).
- Orphan numbers in markdown — values not present in the model output — are a primary failure mode this plugin exists to prevent. Flag them all.

### 5. Units and formulas

- Pick a sample of derived values (at minimum: the headline number, the recommendation's key figure, and one randomly selected derived value). Recompute them by hand from the inputs.
- Check unit conversions (kWh vs kW, $/month vs $/year, hours vs days, basis points vs percent). Wrong units are a common silent failure.

### 6. Sources are fresh and labeled

- For each input, check the "checked YYYY-MM-DD" stamp (or equivalent). Flag stamps older than 6 months, or any source whose own page metadata (a "last updated", "published", or version date on the source itself) is newer than the stamp.
- Every input without a source MUST be explicitly labeled as an assumption. Unlabeled, unsourced inputs are a fail.

### 7. Recommendation matches the data

- Read the recommendation/conclusion in the narrative. Independently determine, from `model_output.json` alone, what the recommendation should be (which option is cheapest, which scenario is best, which threshold is crossed).
- Flag any case where the narrative's recommendation does not follow from the model's numbers.

### 8. Compatibility / model-vs-reality

- For analyses where components must work together (capacity vs throughput, generation vs storage, headcount vs hours, two tax treatments, etc.), confirm the model encodes the constraint as an assertion or check — not only as prose in the memo.
- A model that silently produces numbers for an infeasible system is a fail even when the arithmetic is correct.

## Output

Write a `fact_review.md` in the analysis directory with this structure:

```markdown
# Fact Review — <analysis name>
Reviewer: independent agent, <YYYY-MM-DD>

## Summary
- Checks passed: N / M
- Critical issues: <count>
- Recommendation: ship / fix-then-ship / do-not-ship

## Findings
### [FAIL] Check 2 — Cited values match sources
- `model.py:42` cites Vendor A price as $99/mo from <url>; source shows $109/mo as of <date>.

### [PASS] Check 1 — Links resolve
- 14/14 URLs returned 200.

...
```

Order findings critical-first. **Critical** = the analysis would mislead a decision: cited values that don't match their sources, wrong recommendations, formula or unit errors, infeasible-system bugs, orphan numbers in the narrative. **Medium** = correctness is intact but trust is reduced: dead links, stale "checked" stamps, unlabeled assumptions, output drift. **Low** = cosmetic or easily fixable inconsistencies. Be specific: file paths, line numbers, the cited value, the value found, the URL fetched, the date checked. A finding without evidence the user can re-verify is not useful.

## What This Skill Does NOT Do

- Does not edit the analysis. The reviewer reports; the author fixes.
- Does not re-do the analysis. If the methodology is wrong, flag it; do not silently produce a "corrected" version.
- Does not check prose style, tone, or grammar. Out of scope.
