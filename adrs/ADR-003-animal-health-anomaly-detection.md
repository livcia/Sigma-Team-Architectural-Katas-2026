Date: 2026-09-14

# ADR-003: Animal health anomaly detection approach

## Status
Proposed

## Context

[Animal Health Intelligence](../use-cases/animal-health-intelligence.md) must
turn enclosure telemetry (temperature, humidity, water quality), feeding
records, keeper observations and activity signals into prioritised attention
for keepers and veterinarians, catching problems earlier than today's
undocumented, manual process. This capability carries the highest stakes of
the three AI use cases: animal welfare, keeper/veterinarian time cost, and
estate reputation are all directly affected by both missed incidents (false
negatives) and excessive false alarms (false positives) — see
[../docs/architecture-characteristics.md](../docs/architecture-characteristics.md).

Alternatives considered:

- **Fixed, species-wide threshold rules** (for example, a single acceptable
  temperature range per species) applied uniformly across enclosures. Simple
  and fully explainable, but does not account for enclosure-specific baselines,
  individual animal variation, or combinations of weaker signals that together
  indicate a problem; this was the estate's implicit status quo and is exactly
  what produces late detection today.
- **A fully autonomous diagnostic/decision system** that would recommend
  specific treatments or interventions. Rejected outright: this would cross
  into veterinary decision-making, which must remain a human responsibility
  (AI2 in [../docs/requirements.md](../docs/requirements.md)); it also risks
  false confidence in an area with direct welfare and safety consequences.
- **Anomaly detection against each enclosure's own historical baseline**,
  combined with simple rule-based thresholds as a fallback, producing
  prioritised, explainable alerts for a human to act on. This is the approach
  adopted.

## Decision

Animal Health Intelligence uses a **per-enclosure anomaly-detection approach**:
telemetry, feeding and activity data are compared against that specific
enclosure's own historical baseline (not a generic species-wide threshold),
producing alerts at three levels (Watch, Priority check, Escalate) as described
in [animal-health-intelligence.md](../use-cases/animal-health-intelligence.md).
Every alert must state its contributing signals (explainability) and never
constitutes a diagnosis or treatment instruction. A **simple rule-based
threshold fallback** is defined and used automatically if the anomaly-detection
model's quality (particularly its false-negative rate) degrades below a defined
threshold, as described in ADR-006.

This decision does not select a specific anomaly-detection algorithm or
vendor; the technique sits behind the AI abstraction layer defined in ADR-001.

## Consequences

**Positive:**
- Per-enclosure baselines account for natural variation between enclosures and
  individual animals, which fixed species-wide thresholds cannot, directly
  addressing the "late detection" part of the core business problem (see
  [../docs/business-context.md](../docs/business-context.md)).
- Explainable, leveled alerts let keepers and veterinarians prioritise limited
  time without being overwhelmed, and keep the human as the final decision
  maker (AI2 in [../docs/requirements.md](../docs/requirements.md)).
- The rule-based fallback ensures alerting never silently stops or continues
  on a known-degraded model, protecting animal welfare even during a model
  review period.

**Negative / trade-offs:**
- A per-enclosure baseline approach needs enough historical data per enclosure
  to be reliable; early on (greenfield system, assumption A1 in
  [../docs/assumptions.md](../docs/assumptions.md)), confidence will be lower
  for newly monitored enclosures until sufficient data accumulates.
- Explaining every alert with contributing signals adds design and engineering
  effort compared to a simple threshold breach notification.
- False positives and false negatives both carry real cost (keeper/vet time
  versus welfare risk); tuning the model to balance them is an ongoing
  operational effort, not a one-time setup.

**Operational impact:**
- Keepers and veterinarians must actively confirm, dismiss or escalate alerts
  for the feedback loop that evaluates precision/recall to function (see
  [animal-health-intelligence.md](../use-cases/animal-health-intelligence.md)
  and ADR-006); without this participation, model quality cannot be measured.
- The system administrator, jointly with a veterinarian, owns the decision to
  suspend AI-generated alerting and rely on the rule-based fallback if quality
  degrades (see ADR-006).

**Reversibility:**
- The anomaly-detection technique itself can be replaced behind the AI
  abstraction layer (ADR-001) without changing this decision's core choice:
  per-enclosure baselines, explainable leveled alerts, and human-owned final
  decisions. If per-enclosure baselining proves impractical at scale, a hybrid
  approach (enclosure-type baselines with per-enclosure adjustment) could be
  adopted as a superseding decision.
