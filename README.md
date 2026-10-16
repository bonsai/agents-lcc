# agents-lcc

**AI Agent Lifecycle Cost Intelligence — from first token to long-term maintenance.**

agents-lcc is a proposed SaaS platform for measuring, explaining, and reducing the total lifecycle cost of AI-assisted software development. It connects model usage, multi-agent orchestration, MCP/tool calls, automated evaluation, human review, infrastructure, and post-release maintenance into one traceable cost ledger.

> **Status: Research / MVP planning.** This repository currently defines the product direction and implementation backlog. Features described below are goals, not claims that a production service is already available.

## Why

Token counts alone do not tell you whether an AI workflow is economical. A cheaper run may fail acceptance tests, require several repair cycles, consume more reviewer time, or create higher maintenance costs later.

agents-lcc aims to answer:

- What did it cost to deliver a quality-qualified software change?
- Which lifecycle phase, model, agent, MCP tool, or evaluation step drove the cost?
- How much did retries, failed runs, rework, and human approval contribute?
- Does a multi-agent or MCP-enabled workflow pay for its setup and orchestration overhead?
- How does the cost evolve from planning and implementation through operations and maintenance?

## Product principles

- **Lifecycle cost, not token vanity metrics.** Optimize cost-to-quality and long-term outcomes, not token count in isolation.
- **Traceable by default.** Connect project → Intent → Unit/task → Bolt/run → event, retaining GitHub Issue/PR references where available.
- **Evidence over guesses.** Separate provider-reported usage, actual billed amounts, calculated estimates, and allocated shared costs.
- **Unknown is not zero.** Missing telemetry or pricing is surfaced as incomplete data.
- **Human work counts.** Record active review/approval time separately from elapsed waiting time.
- **Provider-neutral.** Keep the core event schema independent of any one model provider, agent framework, or MCP server.
- **Privacy-first.** Do not collect API secrets or raw prompts/responses by default.
- **Reproducible.** Recalculate reports from append-only JSONL events and a pinned pricing version.

## Lifecycle model

The product maps AI-driven development to measurable lifecycle stages, informed by AI-DLC concepts.

| Lifecycle | What is measured |
| --- | --- |
| Inception | Intent clarification, research, requirements, planning, decomposition, approval |
| Construction | Design, agent/model execution, coding, MCP calls, tests, reviews, retries, rework |
| Release | Build, CI, deployment preparation, release verification |
| Operations | Runtime infrastructure, monitoring, incidents, feedback, operational agent runs |
| Maintenance | Bug fixes, dependency updates, regressions, repeated evaluations, human support |
| Retirement | Migration, shutdown, data retention/deletion, replacement costs |

AI-DLC terminology used for traceability:

- **Intent** — the business or user outcome and its success criteria.
- **Unit** — a cohesive work unit derived from an Intent.
- **Bolt** — a bounded implementation/feedback iteration.
- **Human gate** — an explicit decision or approval that may stop, redirect, or accept work.

The mapping is a measurement model, not a requirement to adopt a particular AI-DLC implementation.

Reference: [AI-DLC overview on Qiita](https://qiita.com/ryu-ki/items/a70ec13e4b622a37cd6f).

## Planned architecture

```text
AI coding agents / model providers / MCP servers / CI / human work
                              |
                              v
                    Collectors and adapters
                              |
                              v
                   Append-only JSONL ledger
                              |
                 +------------+------------+
                 |                         |
                 v                         v
        Versioned pricing data      Evaluation evidence
                 |                         |
                 +------------+------------+
                              |
                              v
                      LCC calculation engine
                              |
                              v
                SaaS dashboard / exports / API
```

### Core components

1. **Telemetry collectors** — normalize model usage, agent hierarchy, MCP/tool calls, CI/evaluation, human effort, and lifecycle events.
2. **Usage and cost ledger** — preserve source, timestamp, event IDs, run/task relationships, and measurement confidence.
3. **Pricing registry** — version model/tool prices by provider, model, token category, currency, and effective date.
4. **LCC engine** — calculate task-level and cumulative costs with explicit attribution and deduplication rules.
5. **Evaluation layer** — connect cost to acceptance tests, quality thresholds, retries, and successful outcomes.
6. **SaaS dashboard** — explore cost by project, task, lifecycle phase, agent, model, tool, and time; export evidence for audits and research.

This is a planned architecture. Provider adapters, billing integrations, authentication, and production hosting will be added incrementally.

## Metrics

| Metric | Definition |
| --- | --- |
| Total lifecycle cost | AI/model + tools/MCP + CI/evaluation + infrastructure + human labor + allocated setup cost, with actual and estimated amounts shown separately |
| Cost per successful task | Attributed cost / tasks meeting the agreed acceptance criteria |
| Quality-adjusted cost | Attributed cost / outputs meeting the defined quality threshold |
| First-pass rate | Tasks passing acceptance criteria on the first attempt / evaluated tasks |
| Rework cost ratio | Cost attributable to retries, fixes, and repeated work / total attributed cost |
| Evaluation cost ratio | Evaluation and repair-loop cost / total attributed cost |
| Human minutes per success | Active human work minutes / successful tasks |
| Maintenance cost ratio | Observed maintenance cost / lifecycle cost within the selected observation window |
| Cache hit rate | Cached input tokens / eligible input tokens, when the provider exposes compatible usage fields |
| Workflow break-even | Work volume required for measured savings to offset setup and orchestration overhead |

Metric definitions, denominators, exclusions, and data coverage must accompany every report. A cache hit rate is not itself a monetary saving; savings must be calculated using the applicable price schedule.

## Cost accounting

The calculation engine is intended to distinguish:

- **Actual spend** — provider or invoice amounts supported by source evidence.
- **Calculated estimate** — usage multiplied by a versioned price schedule.
- **Human labor** — measured active minutes multiplied by a configurable rate.
- **Infrastructure and CI** — directly measured charges where possible, otherwise clearly marked estimates.
- **Shared/setup costs** — allocated using a documented policy, displayed separately from actual cash spend.

Parent-agent summaries must not be added to child-agent charges when they describe the same underlying usage. Retries and failed tasks remain part of cost even when they do not count as successful outcomes. Unknown usage or missing prices must never silently become zero.

## Security and data handling goals

- Never store provider API keys, access tokens, or other secrets in telemetry.
- Avoid raw prompts and model responses by default; prefer metadata, hashes, identifiers, and evaluation outcomes.
- Design tenant isolation and role-based access controls before exposing customer data in a hosted service.
- Define retention, deletion, export, and audit policies before production use.
- Treat billing exports and telemetry as sensitive operational data.
- Document which fields are collected, why they are needed, and whether they are measured or inferred.

These are design requirements; production security controls must be implemented and independently tested before the service is used with sensitive workloads.

## SaaS direction

The intended service is a multi-tenant SaaS, with a provider-neutral core and integrations added in stages.

Potential product capabilities:
- Organization/workspace and project-level cost views.
- API/CLI ingestion for local or CI agent runs.
- GitHub Issue/PR linkage and workflow comparisons.
- Budget thresholds and cost anomaly alerts.
- JSONL/CSV export and reproducible reports.
- Team-level permissions, retention controls, and billing/usage reconciliation.

Pricing, service tiers, availability guarantees, and a hosted signup flow are **not yet defined**. The first milestone is a trustworthy local ledger and reproducible LCC engine before production SaaS operations.

## Research plan

1. Define the lifecycle methodology, event contract, and JSONL schema.
2. Implement append-only collectors for agent, MCP, evaluation, CI, and human effort.
3. Implement versioned pricing and the LCC calculation engine.
4. Map AI-DLC phases, Intent/Unit/Bolt, and human approval gates to measurable events.
5. Benchmark equivalent tasks across single-agent, multi-agent, MCP-enabled, and automated-evaluation workflows.
6. Reconcile estimates against provider usage exports and actual bills.
7. Add the SaaS dashboard and production-grade tenant/security controls after the measurement core is validated.

The initial implementation backlog is tracked in [GitHub Issues](https://github.com/bonsai/agents-lcc/issues).

## Development status

| Area | Status |
| --- | --- |
| Product framing and lifecycle model | Drafted |
| Research backlog | Created |
| JSONL schema | Planned |
| Collectors | Planned |
| LCC engine | Planned |
| Benchmark and dashboard | Planned |
| Hosted multi-tenant SaaS | Planned |

## Contributing

The initial focus is on clear definitions and reproducible measurement. Before adding a new integration, document:

1. Which event types it emits.
2. Where usage values originate and what may be missing.
3. How duplicate events and retries are handled.
4. How costs are calculated and which pricing version applies.
5. How its results are tested without paid credentials.

## License

License has not yet been selected. Do not assume the repository is open for reuse until a license is added.
