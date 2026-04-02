# Pipeline Architect Milestones

This roadmap starts with the smallest proof of value: one local script that can turn a prompt into a structured spec, generate one file, and validate it.

## Working Decisions

- Default runtime: Ollama
- Primary model for code and structured tasks: `qwen2.5-coder:14b`
- Prose model for README/help/summaries: `llama3.1`
- First proof target: natural language -> structured spec -> generated artifact -> validation
- Delay FastAPI, LangGraph, frontend, retrieval, and repair orchestration until the core loop works locally

## Milestone Sequence

### Milestone 0: Repo Bootstrap

Goal: lock in the local-first defaults and make the repo ready for the first script.

Scope:
- add the initial repo docs
- add `.env.example`
- add `.gitignore`
- document the model split and local Ollama assumptions

Recommended commit(s):
- `docs: add project overview and MVP milestones`
- `chore: add env example and gitignore`

Done when:
- the repo has a clean starting point
- local config expectations are obvious

### Milestone 1: Prompt -> Valid JSON Spec

Goal: build a single Python function that sends a request to Ollama and returns parsed JSON shaped like a `PipelineSpec`.

Input:
- `"Ingest daily CSVs from S3, clean null emails, dedupe by user_id, load to Postgres"`

Output JSON fields:
- `project_name`
- `source`
- `destination`
- `transforms`
- `schedule`

Constraints:
- one runnable script
- no frontend
- no FastAPI
- no LangGraph

Recommended commit(s):
- `feat(spec): add Ollama-backed prompt-to-json script`
- `test(spec): verify parsed output shape`

Done when:
- one script runs locally
- it prints valid parsed JSON
- the JSON contains the required fields

### Milestone 2: JSON Spec -> One Generated File

Goal: take the parsed JSON and render one Jinja template into one real file on disk.

Target output:
- `dags/<project_name>.py`

Scope:
- convert the parsed JSON into template context
- render one Airflow DAG template
- derive the output filename from the spec
- write the generated file to disk

Recommended commit(s):
- `feat(generate): render one Airflow DAG from the parsed spec`
- `test(generate): verify output path and template rendering`

Done when:
- the same prompt can generate a real Airflow DAG file
- template variables are filled correctly
- the file name comes from the spec

### Milestone 3: Validate Generated Output

Goal: add one validation pass after file generation.

MVP validation rules:
- Python AST parse for generated `.py`
- YAML parse for generated `.yml` or `.yaml`
- fail if unresolved template markers remain, such as `{{ ... }}`

Recommended commit(s):
- `feat(validate): add generated file validation`
- `test(validate): catch syntax and template marker failures`

Done when:
- bad generations are caught automatically
- the script prints `valid` or a list of validation errors

### Milestone 4: Refactor the Script into Real Project Structure

Goal: move the proven loop into reusable modules without changing the behavior.

Scope:
- introduce typed Pydantic models
- split the script into small services
- add a clearer package layout under `core/`
- keep the same prompt-to-artifact flow working

Recommended commit(s):
- `refactor(core): extract models and services from the MVP script`
- `test(core): preserve prompt-to-artifact behavior through refactor`

### Milestone 5: Add Repair and Better Spec Quality

Goal: improve the loop after the basic path is already reliable.

Scope:
- add one repair attempt after validation fails
- strengthen prompt instructions and structured parsing
- add fixture-based tests for multiple prompt variants

Recommended commit(s):
- `feat(repair): add one-pass repair attempt for invalid outputs`
- `test(spec): add fixture coverage for multiple pipeline prompts`

### Milestone 6: Add a Thin API Layer

Goal: expose the existing local workflow through FastAPI without rewriting the core logic.

Scope:
- add a generate endpoint
- add a validate endpoint
- return generated files and validation output as structured responses

Recommended commit(s):
- `feat(api): add generate and validate endpoints`

### Milestone 7: Add Workflow Orchestration

Goal: introduce LangGraph only after the single-path workflow is already working.

Scope:
- model workflow state explicitly
- convert the local flow into nodes
- preserve inspectability for parse, generate, validate, and repair steps

Recommended commit(s):
- `feat(graph): add workflow state and core nodes`

### Milestone 8: Add Retrieval and Example Support

Goal: improve planning quality with local examples, while staying local-first.

Scope:
- add curated example specs and outputs
- add Chroma-backed retrieval behind a service boundary
- keep a static fallback if retrieval is unavailable

Recommended commit(s):
- `feat(retrieval): add local example store and Chroma search`

### Milestone 9: Add Frontend and OSS Polish

Goal: make the repo usable as a local app and presentable as an open-source project.

Scope:
- add the Next.js UI
- surface spec, artifacts, and validation results
- improve README, setup docs, and demo flow

Recommended commit(s):
- `feat(web): add local generation UI`
- `docs: polish setup and demo instructions`

## Suggested Commit Rhythm

Use small checkpoints that match visible progress:

1. docs and config
2. prompt-to-json
3. json-to-file
4. validation
5. refactors that preserve the working loop
6. API, orchestration, retrieval, and UI only after the loop is proven

## Recommended Build Order

Build in this order:

1. Milestone 0
2. Milestone 1
3. Milestone 2
4. Milestone 3
5. Milestone 4
6. Milestone 5
7. Milestone 6
8. Milestone 7
9. Milestone 8
10. Milestone 9

Do not start FastAPI, LangGraph, or the frontend before Milestone 3 is working end to end.
