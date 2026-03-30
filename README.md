# analysis-process

A Claude Code plugin that makes AI-assisted analysis auditable and reproducible. It teaches Claude to separate narrative from computation, track where every number comes from, and produce pipelines you can re-run when inputs change.

## The Problem

When you ask Claude to analyze something — compare vendors, project costs, evaluate options — the default behavior is to do the math in its head and write the answer directly into prose:

> Based on the pricing, Vendor A costs $2,510/year, Vendor B costs $2,627/year,
> and Vendor C costs $2,197/year. **I recommend Vendor C**, saving $430/year.

This looks helpful, but it's actually fragile:

- **Where did those numbers come from?** You can't tell if Claude used the right inputs, made a math error, or hallucinated a price. There's no code to inspect.
- **What happens when inputs change?** A vendor updates pricing, your usage estimate changes, a new option appears. You have to re-prompt from scratch and hope Claude remembers all the context.
- **Can you trust the document?** If this feeds into a decision memo or a report, every number is a one-time calculation that nobody can verify or reproduce.

This gets worse as analysis complexity grows. With 5+ inputs, multiple scenarios, or sensitivity ranges, the chance of a silent error in a prose-only response approaches certainty.

## What This Plugin Does

With this plugin installed, Claude structures analyses as reproducible pipelines instead of one-shot responses. For the same vendor comparison:

```
vendor-comparison/
  model.py              # all inputs (with sources) and computations
  model_output.json     # structured results, committed and diffable
  memo.template.md      # narrative with {{placeholders}} for data
  fill_templates.py     # deterministic: JSON data -> filled document
  memo.filled.md        # final document, every number traceable to model.py
```

Now you can:
- **Audit**: open `model.py`, see every input with its source, follow the math
- **Update**: change an input, run `uv run model.py && uv run fill_templates.py`, get an updated memo
- **Diff**: `git diff model_output.json` shows exactly what changed and why
- **Trust**: no number exists in the document that isn't computed from a named, sourced input

The plugin provides two skills that activate automatically during analysis work, or you can invoke the pipeline skill explicitly with `/analysis-pipeline`.

## Install

```sh
/plugin marketplace add bestdan/analysis-process
/plugin install analysis-process@analysis-process
```

## Skills

| Skill | Trigger | What it does |
|---|---|---|
| **analysis-pipeline** | `/analysis-pipeline` or auto when building report-producing analyses | Structures work as model -> template -> fill pipelines. Enforces input provenance, separation of narrative and computation, and reproducible output. |
| **analysis-conventions** | Auto when writing notebooks or analysis scripts | Coding conventions: when to use plain scripts vs marimo, `uv run` for execution, scratchpad patterns, notebook structure. |

## Example

The `skills/analysis-pipeline/example/` directory contains a complete working pipeline (vendor cost comparison) you can run:

```sh
cd skills/analysis-pipeline/example
uv run model.py              # compute all values -> model_output.json
uv run fill_templates.py     # fill template placeholders -> memo.filled.md
```

## Staying Up to Date

Third-party marketplaces have auto-update disabled by default. Enable auto-update in the `/plugin` UI (Marketplaces tab), or update manually:

```sh
/plugin marketplace update bestdan/analysis-process
/reload-plugins
```
