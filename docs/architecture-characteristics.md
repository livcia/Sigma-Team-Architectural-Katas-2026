# Architecture characteristics

This document selects the architecture characteristics ("-ilities") that matter
most for Von Digitalis Estates and explains why. It builds on
[business-context.md](business-context.md), [requirements.md](requirements.md) and
[assumptions.md](assumptions.md), and is used to justify decisions in
[../architecture/](../architecture/) and [../adrs/](../adrs/).

Judges evaluate "fit with constraints" and "handling uncertainty"; these six
characteristics are the ones this solution is explicitly optimised for. Other
characteristics (for example raw throughput or UI polish) are treated as secondary
given the estate's actual scale (5,000–15,000 visitors/day — see
[assumptions.md](assumptions.md), D5).

## Why only six

The brief and business context point to a small, coherent set of forces: patchy
connectivity, a small but expert operations team, animal-welfare and safety
stakes, and an AI landscape that changes quickly. Rather than listing every
possible "-ility," this document picks the ones that would cause the solution to
fail its stated goals if neglected.

## 1. Resilience / fault tolerance

**Why it matters:** Wi-Fi coverage across the estate is explicitly patchy
(TC1 in [requirements.md](requirements.md)). Ticketing, telemetry capture and
basic operations must continue to function through connectivity gaps; an
architecture that assumes continuous connectivity would fail during normal daily
operation, not just as an edge case.

**Architectural decisions that support it:**
- Local edge gateways per zone that buffer MQTT telemetry and events durably
  during Wi-Fi outages (OE1 in [requirements.md](requirements.md)).
- Core operational functions (ticketing, entry, basic monitoring) designed to not
  depend on cloud or AI reachability (OE2).
- Defined degraded/offline modes for every AI capability (OE3, AI3) — cached
  results, static content or simple deterministic rules instead of blocking.
- Separation of transactional, operational and analytical concerns so an
  analytics or AI outage cannot cascade into a ticketing or entry outage.

**Risk if neglected:** A Wi-Fi gap in one zone could stop ticket sales, lose
telemetry needed for animal-health detection, or make the visitor app unusable
estate-wide — directly undermining trust and the three-year growth target.

**How to measure it:**
- Percentage of zones/time where core operations remain available during a
  simulated or observed Wi-Fi outage (design target, to be validated: assumption,
  not a measured fact).
- Data loss rate for buffered telemetry after outages of a defined duration.
- Mean time to recover synchronisation after connectivity is restored.

## 2. Testability

**Why it matters:** AI outputs are non-deterministic, and judges explicitly ask
how the team verifies AI is "actually working and not breaking." Without
testability designed in, neither the operational backbone nor the AI
capabilities can be trusted, especially given animal-welfare and safety stakes.

**Architectural decisions that support it:**
- AI components isolated behind an abstraction/gateway layer (see ADR on AI model
  abstraction), enabling models to be evaluated and swapped independently of the
  rest of the system.
- Golden datasets and historical/labelled data used to test AI quality before and
  after any model or provider change (AI4 in [requirements.md](requirements.md)).
- Clear separation of transactional, operational and analytical data so test data
  can be constructed and replayed without touching production systems.
- Explicit fallback paths (rules, cached results) that can themselves be tested
  independently of AI behaviour.

**Risk if neglected:** A degrading or silently wrong AI model (for example, missed
animal-health anomalies or bad demand forecasts) could go undetected until
real-world harm occurs — expensive, and potentially irreversible for animal
welfare.

**How to measure it:**
- Existence and coverage of golden/regression datasets per AI capability.
- Percentage of AI changes validated against these datasets before deployment.
- Defect/incident rate attributable to AI behaviour, tracked post-deployment.

## 3. Data integrity

**Why it matters:** Both operational decisions (staffing, pricing) and
welfare-critical decisions (animal-health alerts) depend on trustworthy data.
Buffered, edge-originated data that arrives out of order or duplicated could
silently corrupt forecasts or mask a genuine health anomaly.

**Architectural decisions that support it:**
- Edge buffering preserves event ordering and timestamps through outages (OE4 in
  [requirements.md](requirements.md)).
- Synchronisation design accounts for out-of-order and duplicate delivery when
  connectivity resumes.
- Clear ownership boundaries between transactional (ticketing), operational
  (telemetry) and analytical (aggregated/derived) data stores, so corrections in
  one layer do not silently propagate errors into another.
- Audit logging of AI-driven alerts/recommendations and the human decisions made
  on them (SPA4 in [requirements.md](requirements.md)), which also serves as a
  data-integrity check on the pipeline itself.

**Risk if neglected:** Corrupted or misordered telemetry could cause a real
animal-health anomaly to be missed, or a demand forecast to be built on flawed
input — both directly undermine the core value proposition.

**How to measure it:**
- Rate of detected duplicate/out-of-order events after edge synchronisation.
- Reconciliation checks between edge-buffered counts and cloud-received counts
  after an outage.
- Audit-log completeness (percentage of AI alerts/recommendations with a
  recorded human decision, where one is required).

## 4. Security and privacy

**Why it matters:** The system handles visitor personal and payment data,
optionally-collected visitor preferences, and animal-health data with
reputational sensitivity. A breach or misuse would damage visitor trust directly,
undermining the return-visit goal this solution is meant to support.

**Architectural decisions that support it:**
- Visitor personal data separated from operational/analytical data used for
  aggregate demand analysis, with access control appropriate to sensitivity
  (SPA1 in [requirements.md](requirements.md)).
- Anonymisation/aggregation of visitor-flow data wherever individual identity is
  not required (SPA2).
- Opt-in-only collection of preference data for personalisation (SPA5 in
  [requirements.md](requirements.md); V2 in [assumptions.md](assumptions.md)).
- Encryption of data in transit between edge devices, gateways and the cloud
  (SPA6).
- Access control and audit logging on animal-health data given its welfare and
  reputational sensitivity (SPA3, SPA4).

**Risk if neglected:** Exposure or misuse of visitor personal data could trigger
regulatory and reputational harm; unrestricted access to animal-health data could
enable undetected neglect or falsified records.

**How to measure it:**
- Coverage of access-control policies across data categories (visitor PII,
  operational telemetry, animal-health data).
- Percentage of preference data collection points that are opt-in and documented.
- Results of periodic access and audit-log reviews (design target: regular
  cadence, exact frequency to be defined operationally).

## 5. Scalability

**Why it matters:** The estate must grow from approximately 5,000 to 15,000 daily
visitors within three years (business target from the brief) without a
proportional rise in operating cost — this is a stated success criterion (SC4 in
[requirements.md](requirements.md)), not just a technical nice-to-have.

**Architectural decisions that support it:**
- Zonal edge gateways so device and connectivity load is distributed across the
  estate rather than centralised in one bottleneck.
- Clear separation of operational and analytical data paths, so analytics/AI load
  growth does not affect core ticketing/entry throughput.
- AI capabilities designed around batch/near-real-time recommendation and
  alerting rather than hard real-time control, allowing capacity to be added
  incrementally.
- Logical, vendor-neutral component design (per AGENT-CONTEXT.md) so capacity can
  be added at the infrastructure level without architectural rework.

**Risk if neglected:** A design that works at 5,000 visitors/day but degrades or
requires a rebuild at 15,000 would jeopardise the three-year growth target and
force costly rework mid-growth.

**How to measure it:**
- System behaviour (latency, error rate) under load tests simulating 15,000
  visitors/day (design target, not yet measured).
- Cost per visitor for infrastructure and AI inference at increasing load levels,
  tracked to confirm it does not scale linearly with visitor count (see
  [cost-and-value-model.md](cost-and-value-model.md) once available).

## 6. Operability and maintainability

**Why it matters:** The estate's own technical staff (system administrator
persona, see [stakeholders-and-personas.md](stakeholders-and-personas.md)) must be
able to run and evolve the system, including replacing AI models/providers as the
market changes (a judged criterion: "handling uncertainty").

**Architectural decisions that support it:**
- AI components isolated behind an abstraction/gateway layer so models and
  providers can be replaced without redesigning the surrounding system (NFR4,
  AI1 in [requirements.md](requirements.md)).
- Production monitoring of AI quality, latency, cost and fallback rate (NFR7,
  AI6), giving administrators early warning of degradation.
- A defined rollback path for a misbehaving model, rather than relying on ad-hoc
  intervention.
- Preference for a small number of coherent components over speculative
  microservices (per AGENT-CONTEXT.md), reducing operational surface area for a
  lean technical team.

**Risk if neglected:** Without clear operability, a single administrator or small
team could be unable to diagnose or fix issues in a distributed, intermittently
connected estate — leading to prolonged outages or an inability to react to a
misbehaving AI model before it causes harm.

**How to measure it:**
- Mean time to detect and mean time to resolve for AI-quality degradation
  incidents.
- Time required to roll back or swap an AI model/provider in a test/staging
  exercise.
- Number of distinct components/services an administrator must understand to
  operate the system (a proxy for operational complexity).

## Assumptions used in this document

Specific numeric thresholds (percentages, time-to-recover targets) are stated as
design targets to be validated once real operational data exists, not as measured
facts — consistent with [assumptions.md](assumptions.md). The selection of these
six characteristics, and the exclusion of others (for example, strict real-time
performance or UI/UX polish), is a judgement call based on the estate's stated
constraints and is itself an assumption open to revision if new requirements
emerge.
