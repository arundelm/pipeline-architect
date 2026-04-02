# Pipeline Architect

Pipeline Architect is a local-first app that turns a natural-language data pipeline request into a typed pipeline spec, generated project files, validation output, and downloadable artifacts.

## Model Policy

- Primary model: `qwen2.5-coder:14b`
- Prose model: `llama3.1`

`qwen2.5-coder:14b` is the default for parsing, planning, generation, validation-repair, and other structured or code-heavy tasks.

`llama3.1` is reserved for README content, inline help text, and human-readable summaries.

## Current Status

The repo is in planning/bootstrap mode.

Implementation milestones live in [docs/milestones.md](/Users/mickarundel/Desktop/mig/pipeline-architect/docs/milestones.md).
