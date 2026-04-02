# Pipeline Architect Milestones

This roadmap turns the product spec into commit-sized increments so the repo can grow in a clean, reviewable way.

## Working Decisions

- Default runtime: Ollama
- Primary model for all code and structured tasks: `qwen2.5-coder:14b`
- Prose model for README/help/summaries: `llama3.1`
- Backend stack: Python 3.12, FastAPI, Pydantic v2, LangGraph, Jinja2
- Frontend stack: Next.js, TypeScript, Tailwind CSS
- MVP targets:
  - `s3_csv -> postgres`
  - `s3_csv -> clickhouse`
  - `stripe_api -> snowflake + dbt`

## Milestone Sequence

### Milestone 0: Repo Bootstrap

Goal: create the base repo layout and lock in project decisions.

Scope:
- add the initial repo docs
- create top-level folders for `apps`, `core`, `tests`, and `docs`
- add base config files such as `.gitignore` and environment examples
- document the model-routing policy

Recommended commit(s):
- `docs: add project overview and milestone roadmap`
- `chore: scaffold repository structure and base config`

Done when:
- the repo has a stable layout
- a new contributor can see the project direction immediately

### Milestone 1: Core Domain Models

Goal: define the typed backbone of the system before any orchestration work.

Scope:
- implement `PipelineSpec`, `FileArtifact`, and `ValidationReport`
- add enums or registries for sources, transforms, destinations, and orchestrators
- add fixture examples for the three MVP pipeline shapes
- add unit tests for model validation

Recommended commit(s):
- `feat(core): add pipeline domain models`
- `test(core): cover pipeline spec validation rules`

Done when:
- the core data model is stable enough to drive the rest of the app
- invalid combinations fail fast and clearly

### Milestone 2: Provider and LLM Routing Layer

Goal: make model usage explicit and swappable without committing to hosted providers.

Scope:
- add a provider interface in `core/providers`
- implement Ollama as the default provider
- add a small routing layer that sends:
  - code/structured tasks to `qwen2.5-coder:14b`
  - README/help/summary tasks to `llama3.1`
- add config-driven model names and sensible defaults
- add tests for task-to-model routing

Recommended commit(s):
- `feat(llm): add provider abstraction and Ollama client`
- `feat(llm): route prose tasks to llama3.1 and code tasks to qwen2.5-coder`

Done when:
- model selection is centralized
- no workflow node hardcodes model names directly

### Milestone 3: Planning Workflow MVP

Goal: turn a free-form request into a validated `PipelineSpec`.

Scope:
- implement request parsing and normalization
- implement planning nodes for:
  - `parse_request`
  - `enrich_context`
  - `plan_pipeline`
- start with local example lookup using static examples before vector retrieval
- expose inspectable intermediate state

Recommended commit(s):
- `feat(graph): add request parsing and planning nodes`
- `feat(api): add generate-spec endpoint`

Done when:
- a user prompt can produce a typed spec for the three MVP targets
- workflow state can be inspected during development

### Milestone 4: Template Rendering and Artifact Output

Goal: generate useful files from a valid spec.

Scope:
- create template registries and template selection logic
- add Jinja templates for:
  - Airflow or Dagster skeletons
  - dbt files
  - common project files
- implement artifact storage on the local filesystem
- return generated files through the API

Recommended commit(s):
- `feat(templates): add template registry and first MVP templates`
- `feat(artifacts): render files and persist outputs locally`

Done when:
- the system can produce a coherent project bundle for at least one end-to-end MVP path

### Milestone 5: Validation and Single Repair Loop

Goal: keep output deterministic and recover once when generation is invalid.

Scope:
- implement structural validation for generated files
- add syntax checks and lightweight linting where practical
- implement one repair pass using the primary code model
- record validator output and repair attempts in workflow state

Recommended commit(s):
- `feat(validation): validate generated artifacts`
- `feat(repair): add one-pass repair loop for invalid outputs`

Done when:
- invalid generations are surfaced clearly
- one automatic repair attempt can improve or fail cleanly

### Milestone 6: Frontend MVP

Goal: let a user run the workflow locally without touching the backend directly.

Scope:
- build a simple Next.js interface for:
  - entering a request
  - viewing the generated spec
  - browsing generated files
  - reviewing validation output
  - downloading a ZIP artifact
- add lightweight help text generated or maintained for the prose model lane

Recommended commit(s):
- `feat(web): add generation form and result views`
- `feat(web): add artifact browser and validation panels`

Done when:
- one local browser session can complete the main generation flow

### Milestone 7: Retrieval and Example Library

Goal: improve planning quality with local examples, while staying local-first.

Scope:
- add curated example specs and outputs
- add embedding and retrieval support with Chroma
- wire retrieval into the planning workflow behind a clean service boundary
- keep static fallback behavior if embeddings are unavailable

Recommended commit(s):
- `feat(examples): add curated local example library`
- `feat(retrieval): add Chroma-backed example search`

Done when:
- planning can use similar local examples without making remote calls

### Milestone 8: Polish, Demo Path, and OSS Readiness

Goal: make the repo strong enough for open-source use and portfolio review.

Scope:
- tighten README and local setup docs
- add sample screenshots or demo assets
- expand test coverage for the critical workflow
- add a stable demo script or seed prompt set
- clean up env handling and developer ergonomics

Recommended commit(s):
- `docs: improve local setup, examples, and demo flow`
- `test: expand end-to-end coverage for main workflow`

Done when:
- a new developer can clone, run Ollama, start the app, and produce a demo output quickly

## Suggested Commit Rhythm

If you want especially frequent Git checkpoints, use this cadence:

1. commit after folder/config scaffolding
2. commit after each major domain or service layer
3. commit after each user-visible workflow slice
4. commit after each test batch that locks in behavior

That gives you small, reviewable commits without fragmenting the work into noise.

## Recommended Build Order

If you want the fastest path to a working demo, build in this order:

1. Milestone 0
2. Milestone 1
3. Milestone 2
4. Milestone 3
5. Milestone 4
6. Milestone 5
7. Milestone 6
8. Milestone 7
9. Milestone 8

Do not start retrieval, multi-provider support, or polish work before the spec-to-artifacts path works locally.
