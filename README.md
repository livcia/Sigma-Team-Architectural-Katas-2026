# Von Digitalis Estates: AI-Driven Visitor and Animal Care Intelligence

*Architectural Katas 2026 submission — Sigma Team*

## Executive summary

The 72nd Countess Von Digitalis has inherited a large estate and needs it to become
profitable by combining an 18th-century amusement park with an exotic animal
collection. The estate must grow from roughly 5,000 to 15,000 visitors per day
within three years, while patchy Wi-Fi, budget-limited hardware and animal welfare
constraints must all be respected.

This solution proposes a **baseline operational platform** — ticketing, MQTT-based
sensing, an edge gateway for intermittent connectivity, and cloud-based analytics —
augmented by **three AI capabilities** that turn existing and new estate data into
decisions people can trust: understanding demand, catching animal-health problems
early, and giving every visitor a reason to come back. AI is treated as a
recommendation and alerting layer behind an abstraction boundary, never as an
unsupervised decision-maker for safety, veterinary or financial outcomes.

## Business problem

Von Digitalis Estates today operates with very limited operational visibility. Ticket
sales exist, but there is no systematic way to understand which rides and enclosures
draw crowds, why animals get sick, or what makes a visitor come back. Without this
insight the estate cannot decide where to invest, how to staff each day, or how to
build a lasting relationship with its visitors — all while trying to triple daily
attendance in three years without proportionally tripling operating costs or animal
welfare risk.

### The three core problems

1. **No visibility into popularity and demand.** Nobody know reliably which rides,
   zones or times draw crowds or cause congestion, so staffing, maintenance and
   investment decisions are made on instinct rather than evidence.
2. **High cost of animal care.** With 200+ exotic animals across 55 enclosures, health
   problems are often discovered late, after they have become expensive or
   irreversible; keeper and veterinary time is scarce and needs to be prioritised.
3. **Few reasons for visitors to return.** The estate has no way to personalise the
   experience or maintain a relationship with a visitor after they leave the gates,
   so growth depends entirely on new-visitor acquisition.

## Solution narrative

We separate the estate's transformation into two layers that must both exist for AI
to be useful:

1. **An operational backbone first.** Ticketing, entry/zone occupancy, ride and
   enclosure telemetry over MQTT, a local edge gateway that buffers data during
   Wi-Fi gaps, and synchronisation to the cloud once connectivity returns. This
   backbone produces the trustworthy, timely data that any AI capability depends on.
2. **Three targeted AI capabilities layered on top**, each addressing exactly one of
   the three core problems, each producing explainable recommendations or alerts
   rather than autonomous actions, and each with a defined human owner, a fallback
   when AI or connectivity is unavailable, and metrics that show whether it is
   actually helping.

AI is isolated behind a model/provider-agnostic gateway so that today's best model
can be replaced tomorrow without redesigning the estate's architecture — a direct
response to the volatility of the AI provider landscape.

## Three AI use cases

| Use case | Problem addressed | What AI does | Who decides |
|---|---|---|---|
| **[Visitor Intelligence](use-cases/visitor-intelligence.md)** | No visibility into popularity/demand | Forecasts demand, flags congestion, highlights popular/underused zones using ticketing, occupancy, queue, weather and event data | Park operators act on recommendations; staffing/pricing changes stay human-approved |
| **[Animal Health Intelligence](use-cases/animal-health-intelligence.md)** | High animal-care costs, late detection | Detects anomalies in enclosure conditions, feeding and activity telemetry; prioritises checks | Keepers and veterinarians always make the final call |
| **[Personalised Visitor Experience](use-cases/personalised-visitor-experience.md)** | Few reasons to return | Recommends routes, attractions and next-visit ideas from approved estate content, current availability and (opt-in) preferences | Visitor chooses; system offers safe rule-based fallback |

Each use case is documented with its data flow, AI role, human checkpoints, offline
behaviour, and success metrics; see the [diagrams](diagrams/) for the corresponding
Mermaid views.

## Expected business outcomes

- **Better resource allocation**: staffing, maintenance and investment guided by
  actual demand and congestion patterns instead of guesswork.
- **Lower animal-care costs and better welfare**: earlier detection of health and
  environmental issues reduces the cost and severity of interventions.
- **Higher repeat-visit rate**: personalised experiences and next-visit
  recommendations build an ongoing relationship instead of a one-off transaction.
- **Sustainable growth path**: the same backbone and AI capabilities scale from
  5,000 to 15,000 daily visitors without a proportional increase in operational or
  care-giving cost.

Concrete, measurable targets are assumptions until validated with the estate; see
[docs/success-metrics.md](docs/success-metrics.md) for the proposed metrics per use
case.

## Handling patchy Wi-Fi, MQTT, edge and cloud

Intermittent connectivity is treated as a first-class architectural constraint, not
an edge case:

- **MQTT-capable devices** across rides, enclosures and zones publish telemetry and
  occupancy events locally.
- **A local edge gateway** subscribes to this MQTT traffic, buffers it durably when
  the estate's Wi-Fi is unavailable, and forwards it once connectivity is restored.
- **The cloud platform** receives synchronised data, separating transactional,
  operational and analytical concerns so that analytics and AI never block
  day-to-day operations.
- Every AI capability defines an explicit **degraded/offline mode**: when fresh data
  or AI is unavailable, the system falls back to the last known state, static
  content or simple rules rather than failing silently or blocking operations.

Details are in [architecture/target-state.md](architecture/target-state.md) and
[architecture/data-flow.md](architecture/data-flow.md).

## Human-in-the-loop, safety and AI risk

AI in this solution is designed to **inform and prioritise, not to act
autonomously** on safety-critical, veterinary, financial or irreversible matters:

- Every AI output is a **recommendation, alert or ranking**, paired with an
  explanation of the contributing signals.
- A named human role (operator, keeper, veterinarian, or the visitor) always makes
  the final decision at the point where risk or cost is material.
- AI components sit behind an **abstraction/gateway layer**, so models and providers
  can be swapped as the market changes without redesigning the surrounding system.
- Governance covers auditability of AI-driven decisions, rollback of a
  misbehaving model, and clear ownership for each capability.

See [adrs/](adrs/) for the specific decisions and trade-offs, in particular AI
model abstraction and human oversight/governance.

## Validation and measuring success

Because AI outputs are non-deterministic, each capability is validated on three
levels:

1. **Quality of the AI itself** — accuracy/precision-recall style metrics against
   labelled or historical data, tested against golden datasets before and after any
   model change.
2. **Business impact** — did the recommendation change an outcome that matters
   (queue times reduced, health issue caught earlier, visitor returned)?
3. **Operational health** — latency, availability, cost per inference, and rate of
   fallback to degraded mode, monitored continuously in production.

Thresholds trigger escalation to a human reviewer or automatic fallback rather than
letting a degrading model run unchecked. Full detail is in
[docs/validation-and-fitness-functions.md](docs/validation-and-fitness-functions.md)
and [docs/success-metrics.md](docs/success-metrics.md).

## Repository map

This README is the entry point; supporting detail lives in the folders below. Some
files referenced here are produced by later steps of this project's documentation
plan (see `AGENT-PROMPTS.md`) and may not exist yet at every point in the
repository's history — treat this as the target map for the finished submission.

```text
/
├── README.md                       This document — the 5-minute path for judges
├── AGENT-CONTEXT.md                Working conventions for contributors/agents
├── architectural-katas-2026.md     Original competition brief
├── docs/
│   ├── business-context.md         Estate situation, goals, value streams
│   ├── requirements.md             Functional/non-functional/AI requirements
│   ├── assumptions.md              Explicit assumptions and out-of-scope items
│   ├── stakeholders-and-personas.md
│   ├── architecture-characteristics.md
│   ├── success-metrics.md          Per-use-case business/AI/technical metrics
│   ├── validation-and-fitness-functions.md
│   ├── risks-and-mitigations.md
│   └── cost-and-value-model.md
├── architecture/
│   ├── current-state.md
│   ├── target-state.md             Baseline operational architecture
│   ├── data-flow.md
│   ├── event-driven-architecture.md
│   ├── deployment-view.md
│   └── security-and-privacy.md
├── use-cases/
│   ├── visitor-intelligence.md
│   ├── animal-health-intelligence.md
│   └── personalised-visitor-experience.md
├── diagrams/
│   ├── ai-visitor-intelligence.md
│   ├── ai-animal-health.md
│   └── ai-personalised-visitor-experience.md
└── adrs/
    ├── ADR-001-ai-model-abstraction.md
    ├── ADR-002-edge-buffering-and-offline-mode.md
    ├── ADR-003-animal-health-anomaly-detection.md
    ├── ADR-004-human-oversight-and-ai-governance.md
    ├── ADR-005-rag-for-approved-visitor-information.md
    └── ADR-006-ai-monitoring-and-evaluation.md
```

## Assumptions and constraints

- **No specific technologies or cloud vendors are named** at this stage; the brief
  explicitly does not require it, and naming one prematurely would undermine the
  model/provider-swap risk this solution is designed to mitigate.
- **Numeric targets** (visitor growth, cost savings, adoption rates) are treated as
  assumptions or design goals unless stated otherwise, and are labelled as such in
  the linked documents rather than presented as measured facts.
- **A visitor-facing app or portal** is assumed to exist as the delivery channel for
  personalised recommendations; the brief does not specify one explicitly.
- **MQTT hardware budget** is assumed sufficient to instrument rides, enclosures and
  zones at a level useful for occupancy and environmental sensing, but not assumed
  to cover every individual animal.
- **This is an architecture and documentation exercise**, not a production
  implementation; deliverables are written so a judge can follow the reasoning
  without needing to ask the team follow-up questions.
- Detailed data-ownership, staffing and animal-specific assumptions are documented
  in [docs/assumptions.md](docs/assumptions.md) once written.