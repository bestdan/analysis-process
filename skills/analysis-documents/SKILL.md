---
name: analysis-documents
description: Document structure for analysis reports — executive summary first, qualitative content in body, data tables from models, sourced references. Use when writing analysis memos, recommendations, or reports.
user-invocable: false
---

# Analysis Documents

**Note**: Project-level preferences (CLAUDE.md, AGENTS.md) override these.

## When This Applies

Writing or structuring narrative documents that accompany quantitative analysis — memos, recommendations, reports, decision docs. Does not apply to pure code or notebook work.

## Markdown Document Structure

- **Executive summary first** with key numbers and outstanding decisions.
- **Qualitative content** (why, how, what) in the markdown body.
- **Data tables** rendered from model data via templates (not hand-written).
- **Financial analysis sections** are brief: summary table with max/realistic, then a link to the model.
- **Sources** as footnoted references in an appendix.

## Numbers in Documents

MUST NOT derive numbers in markdown files. Markdown is for narrative, decisions, qualitative analysis, system descriptions, assumptions, and open questions.

Summary tables in markdown may show max/realistic ranges but MUST reference the model for derivation. Every number should be traceable to a computation.

## Example

Bad (numbers in markdown):
> The system produces 817 kWh/month, saving $142/month.

Good (reference to model):
> See [model.py] for production estimates. Summary: 750-820 kWh/month,
> $128-$145/month savings (sensitivity table in model).
