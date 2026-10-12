# AGENTS.md — Market Trend Predictor

> Engineering instructions for AI coding agents working in this repository.
> Applies to the entire repository unless a more specific `AGENTS.md` exists in a subdirectory.

## 1. Mission and context

Market Trend Predictor is a **personal research tool**, not a SaaS product. Its purpose is to identify Brazilian Mercado Livre categories and products that may gain popularity in the coming weeks, then help the owner investigate whether buying inventory for resale makes financial sense.

Read `README.md` before substantial work. The README defines the current product goals, planned sources, stack and roadmap. Treat planned capabilities as **not yet implemented** unless confirmed by code and tests. If implementation diverges from the README, flag the discrepancy and propose a documentation update.

Success means trustworthy, reproducible, actionable research—not attractive dashboards, unsupported forecasts, or unnecessary architecture.

## 2. Working principles

1. **Keep it simple.** Favor the smallest maintainable solution that meets the request. Do not introduce microservices, message brokers, Kubernetes dependencies, Redis, TimescaleDB, LLMs, or new frameworks without a concrete need.
2. **Work incrementally.** Make small, focused changes. Avoid unrelated refactors, cosmetic rewrites, or speculative scaffolding.
3. **Prefer evidence.** Verify repository conventions and external API contracts before implementation. Do not invent endpoints, schemas, data fields, quota allowances or model performance.
4. **Preserve user control.** Discuss consequential changes (data deletion, costly paid APIs, infrastructure provisioning, major dependencies, architecture changes) before making them.
5. **Make assumptions explicit.** If an ambiguity is low risk, choose a sensible default and note it. Ask when the choice materially affects data integrity, money, security or irreversible behavior.
6. **No fabricated completion.** Never claim a feature, test, integration, forecast or benchmark works unless verified. Clearly distinguish implemented, mocked and planned work.

## 3. Planned architecture and boundaries

The following is the intended stack, **not a mandate to create every component immediately**:

- **Go:** source connectors, collection workers, API, application/business logic.
- **Python:** exploratory analysis, feature engineering, baseline forecasting, ML training, temporal backtesting.
- **PostgreSQL:** categories, source observations, time series, prediction records and evaluation results.
- **Next.js + TypeScript:** personal dashboard for exploration and comparisons.
- **Docker Compose:** local development and reproducible services.
- **Kubernetes:** optional later deployment in a homelab; not a prerequisite for local operation.

Use clear boundaries and data contracts between Go and Python. Prefer explicit schemas and versioned migrations. Avoid duplicated business logic across languages. Do not create a directory or service merely because it appears in a proposed architecture diagram.

Organize implementation code under `src/`, introducing subdirectories only as needed. Feature planning artifacts live under `specs/` according to the mandatory workflow below. Do not introduce competing top-level application layouts without an agreed architectural decision.

## 4. Source integration and data integrity

Expected candidate sources: Mercado Livre official APIs, Google Trends or a permitted alternative, Google Ads Keyword Planner, Brazilian commercial calendar, Open-Meteo and possibly social signals later.

For each connector:

- Use official or expressly permitted access methods; verify current documentation, authentication, permissions, rate limits, license and retention terms.
- Never bypass access controls, CAPTCHAs, rate limits or platform restrictions. Do not assume scraping is permitted.
- Keep credentials in environment variables or a secrets mechanism; never commit keys, tokens, cookies or personal account data.
- Use bounded timeouts, sensible retries with exponential backoff and jitter, rate limiting, and actionable errors. Respect `429` and `Retry-After` where applicable.
- Make collection idempotent where practicable, to avoid duplicate snapshots. Log source, execution time, outcomes and failures without leaking credentials.
- Preserve provenance: source, external identifier, query/category, geographic scope, unit, extraction timestamp and the date/time the observation refers to.
- Represent missing data explicitly. Never silently substitute zeros or invented values.
- Normalize timestamps consistently (UTC for storage; explicit timezone for business dates). Preserve source timestamps and document aggregation windows.
- Treat rankings, search interest, engagement and estimated searches as **proxies**, not actual sales volumes or revenue.
- Separate original source records from derived metrics when allowed; otherwise retain only what licensing permits.

## 5. Forecasting and evaluation: non-negotiable rules

The initial target is **relative growth of category search interest** at weekly frequency over approximately four weeks (30-day planning horizon). Mercado Livre rankings and trends are complementary signals, not ground-truth sales quantities.

- Define every label and feature precisely: measurement, category mapping, country/region, aggregation interval and prediction horizon.
- Never use information published or observed after the prediction cutoff in features or preprocessing. Account for source publication delays and retrospective data revisions.
- Split train/validation/test **chronologically**; use walk-forward backtesting. Avoid random splits that leak temporal information.
- Establish and report simple baselines (e.g. persistence and seasonal naive) before more complex models.
- Compare candidate models against baselines on the same periods and categories. Use appropriate metrics, including Precision@K for ranked opportunities and forecasting error where applicable.
- Log forecast issue time, cutoff date, target horizon, model/data version, input availability, prediction, uncertainty, and subsequent observed outcome.
- Distinguish model confidence, data coverage and commercial attractiveness. Do not describe a rising search index as guaranteed inventory demand.
- Beware of Google Trends normalization/scaling, ambiguous keyword-to-category mapping, missing data, regional differences and class imbalance.
- Do not introduce LLM-generated labels or external estimates as ground truth without validation.

## 6. Code quality and testing

Follow existing conventions first. For new modules:

- Prefer readable code, descriptive names, small cohesive functions and explicit dependencies over abstractions introduced for hypothetical future needs.
- Format and lint using the ecosystem's standard tools (Go: `gofmt`, `go vet`; Python: project-selected formatter/linter; TypeScript: configured ESLint/TypeScript checks).
- Add tests for business logic, transforms, parsing, category mapping and edge cases. Test connectors with deterministic fixtures or mocked HTTP; do not rely on live APIs in unit tests.
- Add integration tests for database persistence and migrations when a database is introduced.
- Avoid non-determinism in data-science tests; fix random seeds, dataset versions and time windows as applicable.
- Use structured logs with useful context. Avoid noisy logs, secrets, unnecessary personal data and raw tokens.
- Handle API timeouts, partial responses, pagination, pagination changes, empty payloads, invalid units and timezone boundaries explicitly.
- Keep documentation and runnable examples current when behavior or setup changes.

Do not claim to have run tests you did not run. If a command cannot execute, record the reason and the remaining verification steps.

## 7. Security, privacy, cost and operations

- Prefer least-privilege credentials and read-only access where possible.
- Keep `.env` files and secrets out of version control; include only safe `.env.example` templates.
- Do not log, publish or commit personal information, private datasets or licensed API responses without permission.
- Avoid new recurring costs and unbounded polling. Make collection frequency and quotas configurable.
- Never execute destructive database operations, wipe historical data or overwrite trained-model artifacts without an explicit safe migration/backup plan and appropriate approval.
- Favor local reproducibility before cloud deployment. Do not require the homelab Kubernetes cluster for development.

## 8. Spec-driven development and required repository structure

Use **Spec-Driven Development (SDD)** for each new feature or meaningful development stage. Each stage must have its own numbered directory under `specs/`, containing exactly these three core planning documents:

```text
market-trend-predictor/
├── specs/
│   ├── 001-data-ingestion/         # Illustrative name only
│   │   ├── spec.md
│   │   ├── plan.md
│   │   └── tasks.md
│   └── 002-data-normalization/     # Illustrative name only
│       ├── spec.md
│       ├── plan.md
│       └── tasks.md
├── src/
├── AGENTS.md
└── README.md
```

The example stage names above are **fictional placeholders**, not a prescribed roadmap. What is mandatory is the `specs/<number>-<stage-name>/` hierarchy and the `spec.md`, `plan.md`, `tasks.md` files within each stage. Use a zero-padded, monotonically increasing number (e.g. `001`, `002`) and a short descriptive kebab-case name. Put implementation code in `src/`, not in `specs/`.

Responsibilities of each document:

- **`spec.md` — What and why:** problem, objective, scope, functional requirements, acceptance criteria, constraints, edge cases and explicit non-goals. Avoid premature implementation details.
- **`plan.md` — How:** chosen technical approach, components and files affected, data contracts, dependencies, migration considerations, risks, tradeoffs and verification strategy. Ensure the plan satisfies the spec.
- **`tasks.md` — Execution:** ordered, atomic, testable implementation tasks, preferably with Markdown checkboxes (`- [ ]` / `- [x]`), each traceable to the spec and plan.

Agent rules:

1. Before implementing a **new feature or stage**, identify its directory under `specs/`. If none exists, create the three documents before coding. Do not create a new stage for tiny fixes or routine maintenance; reuse the relevant existing stage if applicable.
2. Work in this order: **spec → plan → tasks → implementation → validation**. Do not silently change scope during implementation; update the documents first when requirements change.
3. Review `spec.md`, `plan.md`, and `tasks.md` before coding. Treat their acceptance criteria as the definition of the stage's expected behavior, subject to repository-wide security and data integrity rules.
4. Implement the next incomplete task(s), keeping changes small and focused. Mark tasks complete **only after verifying** the relevant behavior. Record blockers and unfinished work honestly.
5. Keep documents consistent with the implementation and tests. Do not mark a stage finished if its acceptance criteria are not met or validated.
6. If the user explicitly asks only for planning, produce or update the planning documents **without starting implementation**.

## 9. Agent workflow for every task

1. **Inspect:** read relevant repository files, tests, documentation, and the active stage’s `spec.md`, `plan.md`, and `tasks.md`; identify what exists before editing.
2. **State the approach:** for nontrivial changes, give a short plan, impacted components, assumptions and validation strategy.
3. **Implement narrowly:** touch only files necessary for the requested behavior; follow established patterns.
4. **Verify:** run relevant formatting, linting and tests that the environment supports. For data/ML changes, also verify temporal boundaries and metric definitions.
5. **Report:** update verified checkboxes in the active `tasks.md`; summarize the change, files affected, commands/tests actually run, known limitations and suggested next step. Separate verified results from unverified expectations.

When building a new feature, aim for the smallest end-to-end slice that produces a real observable result. Do not replace real-data limitations with silently fabricated results; clearly label fixtures, simulations and mocks.

## 10. Documentation and decision discipline

- **Repository language:** write and maintain `README.md`, `AGENTS.md`, all `spec.md`, `plan.md`, and `tasks.md` files, code comments, commit messages, and technical documentation in English unless the owner explicitly requests otherwise. User-facing market labels may remain in Brazilian Portuguese when appropriate.

- Keep `README.md` focused on the project's purpose, setup, architecture and roadmap.
- Keep this file focused on **how agents should work**; avoid duplicating extensive product requirements here.
- Record material technical decisions and tradeoffs briefly under `docs/decisions/` (ADR format) once that directory exists.
- Document source limitations, availability and legal restrictions near their connectors.
- If a requirement conflicts with these instructions or with platform terms, explain the conflict and propose a compliant alternative.

## 11. Definition of done

A change is done when its behavior matches the task, relevant tests pass (or blockers are disclosed), data semantics and operational failure cases are considered, and docs/examples are updated where needed. For predictions, completion additionally requires an explicit target, time cutoff, baseline and reproducible evaluation method.

**Remember:** this is a decision-support tool for the owner's personal Mercado Livre resale research. Optimize for correctness, clarity and usefulness before sophistication.
