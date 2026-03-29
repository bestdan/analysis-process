# analysis-process

A Claude Code plugin that enforces conventions for auditable quantitative analysis — separation of narrative and computation, input provenance, reproducible pipelines.

## Install

```sh
claude plugin add bestdan/analysis-process
```

Or add it manually to your project's `.claude/settings.json`:

```json
{
  "plugins": ["bestdan/analysis-process"]
}
```

## Skills

### analysis-pipeline (user-invocable)

Reproducible analysis pipelines: separation of narrative and computation, input provenance, document structure, template-driven reports. Applies when building analyses where numbers feed into decisions or reports.

Invoke explicitly with `/analysis-pipeline` or let it activate automatically.

### analysis-conventions

Coding conventions for analysis work: when to use plain scripts vs marimo notebooks, `uv` for execution, scratchpad patterns, notebook structure.

Activates automatically when writing notebooks or analysis scripts.

## Core Principles

- Every number in a final document traces to an input or a computation — never to a one-time response
- Markdown is for narrative; all derived numbers live in code
- Inputs document their source; assumptions are flagged
- Re-running the pipeline regenerates everything from current inputs
