---
name: analysis-conventions
description: Coding conventions for analysis work — marimo vs plain scripts, scratchpad patterns, notebook structure, input/formula/source organization. Use when writing notebooks or analysis scripts.
user-invocable: false
---

# Analysis Conventions

**Note**: Project-level preferences (CLAUDE.md, AGENTS.md) override these.

## When to Use Marimo vs. Plain Scripts

- **Marimo notebook**: exploratory analysis, interactive parameter tweaking, work that benefits from visible intermediate outputs and tables. Good default for analysis work.
- **Plain `.py` script**: deterministic pipeline steps, one-shot computations, CI/automation contexts where a notebook adds unnecessary complexity.

When in doubt, start with marimo. Refactor to a plain script if the notebook is just running top-to-bottom with no interactivity.

- `uvx marimo edit <path>` — interactive editing
- `uvx marimo export html <path> -o output.html` — executes cells and renders output (use for generating reports as side effects)
- `uvx marimo export md <path>` — exports source code as markdown (does NOT execute cells)

## Scratchpad Scripts

When testing setup calls or ephemerally computing numbers during a conversation, MUST write a `temp/_.py` file and re-use it. Don't inline multi-line Python in bash commands that need to be re-approved each time.

Once the numbers are incorporated into a notebook or model, the scratchpad can be deleted — its purpose is to make a single approval all that's required, keep intermediate work visible, not to be a permanent artifact. If there's no model to fold it into, the scratchpad *is* the artifact and should be kept.

MUST NOT embed raw calculated numbers in markdown without a visible formula or a reference to the model that produced them.

## Notebook/Script Structure

- **Inputs first** — key parameters, inputs, specs etc with source links, rates, costs as simple variable assignments
- **Formulas as functions** — named, with docstrings showing the formula
- **Show your work** — render tables with the formula and result side by side (e.g., `(370/1000) x 92 x 24 = 817`)
- **Sources at the bottom** — document where inputs came from
