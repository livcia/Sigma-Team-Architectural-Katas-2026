# Validation and fitness functions

This document describes how the team verifies that the AI capabilities in
[../use-cases/](../use-cases/) actually work, and how it detects when one starts
misbehaving in production — the validation-and-verification concern the
competition brief raises explicitly (see
[../architectural-katas-2026.md](../architectural-katas-2026.md)). It implements
the monitoring approach decided in
[../adrs/ADR-006-ai-monitoring-and-evaluation.md](../adrs/ADR-006-ai-monitoring-and-evaluation.md)
and the governance model in
[../adrs/ADR-004-human-oversight-and-ai-governance.md](../adrs/ADR-004-human-oversight-and-ai-governance.md),
and complements the per-use-case metrics already defined in
[success-metrics.md](success-metrics.md).

All specific numeric thresholds mentioned below are **design targets**, to be
set and refined using real operational data once available, consistent with
[assumptions.md](assumptions.md).

## Why "fitness functions"

Following the architecture characteristics selected in
[architecture-characteristics.md](architecture-characteristics.md) (resilience,
testability, data integrity, security/privacy, scalability, operability), each
validation technique below is tied to one or more of these characteristics, so
validation is not treated as an afterthought bolted onto AI, but as a
continuous check that the system still satisfies the properties it was
designed for.

## 1. Golden datasets

**What it is:** A curated, versioned set of inputs with known-correct or
human-confirmed expected outputs, built separately for each AI capability:

- **Visitor Intelligence**: historical attendance/occupancy periods with known
  actual outcomes, used to check forecast accuracy and congestion-alert
  correctness.
- **Animal Health Intelligence**: enclosure telemetry/observation sequences
  with keeper/veterinarian-confirmed outcomes (genuine anomaly vs. not), built
  from the audit trail described in
  [../use-cases/animal-health-intelligence.md](../use-cases/animal-health-intelligence.md).
- **Personalised Visitor Experience**: a set of representative visitor
  questions paired with the approved knowledge-base passage(s) that should
  ground the answer, plus cases where no approved answer should exist.

**When used:** Before any model or provider change is deployed (per
[../adrs/ADR-001-ai-model-abstraction.md](../adrs/ADR-001-ai-model-abstraction.md)),
and re-run periodically in production to catch drift (see "Data drift testing"
below).

**Ownership:** The system administrator maintains the datasets; the relevant
accountable human role (park operator, keeper/veterinarian) contributes
confirmed outcomes as they occur.

## 2. Regression testing

Every change to an AI capability — a new model version, a changed
provider, an updated prompt/retrieval configuration, or a change to the
underlying data pipeline — is run against its golden dataset before release,
and the results are compared against the previous version's results. A
regression is any statistically meaningful drop in the quality metrics defined
in [success-metrics.md](success-metrics.md) (for example, forecast accuracy,
anomaly precision/recall, groundedness rate). Regressions block deployment
until reviewed.

## 3. Prompt testing (Personalised Visitor Experience)

Because [Personalised Visitor
Experience](../use-cases/personalised-visitor-experience.md) uses
retrieval-augmented generation for grounded information delivery (see
[../adrs/ADR-005-rag-for-approved-visitor-information.md](../adrs/ADR-005-rag-for-approved-visitor-information.md)),
prompt and retrieval behaviour is tested explicitly:

- A representative set of visitor question phrasings (including ambiguous,
  off-topic, and adversarial phrasing) is tested against the approved
  knowledge base to confirm the system retrieves the correct passage, refuses
  correctly when no approved passage exists, and does not introduce claims not
  present in the retrieved content.
- Prompt/retrieval configuration changes are tested the same way as a model
  change: against the golden dataset, with results compared before and after.

## 4. Classification and anomaly-detection quality testing

For [Visitor
Intelligence](../use-cases/visitor-intelligence.md) (congestion/queue
detection) and [Animal Health
Intelligence](../use-cases/animal-health-intelligence.md) (anomaly detection),
quality is tested using standard classification-style metrics against the
golden datasets and against ongoing human-confirmed outcomes:

- Precision and recall (or equivalent), tracked separately, since false
  positives and false negatives have different costs in each use case (see
  [success-metrics.md](success-metrics.md)).
- Per-enclosure baseline behaviour for Animal Health Intelligence is tested
  specifically for enclosures with limited historical data, to confirm the
  system correctly signals low confidence rather than false certainty (per
  [../adrs/ADR-003-animal-health-anomaly-detection.md](../adrs/ADR-003-animal-health-anomaly-detection.md)).

## 5. Resilience testing: missing and delayed data

Directly testing the resilience characteristic in
[architecture-characteristics.md](architecture-characteristics.md):

- Each AI capability is tested with deliberately incomplete input data (missing
  telemetry, missing weather/event context) to confirm it degrades to
  reduced-confidence output rather than failing silently or producing
  false-confident results, as described in each use case's "behaviour with
  missing data" section.
- Each capability is tested with **delayed/buffered data** arriving out of
  order, simulating a Wi-Fi outage and subsequent synchronisation (see
  [../architecture/data-flow.md](../architecture/data-flow.md), Flow 3), to
  confirm delayed data is correctly marked and does not silently overwrite
  more recent state.

## 6. Offline testing

- Each AI capability's defined fallback (rule-based alerting for Animal Health
  Intelligence, manual planning for Visitor Intelligence, static content for
  Personalised Visitor Experience) is tested independently of the AI
  component, to confirm the fallback works correctly on its own, not only as a
  theoretical description in the use case documents.
- A full "AI unavailable" test simulates the AI abstraction layer (see
  [../adrs/ADR-001-ai-model-abstraction.md](../adrs/ADR-001-ai-model-abstraction.md))
  being unreachable, confirming that core ticketing, entry and telemetry
  capture (which do not depend on AI, per OE2 in
  [requirements.md](requirements.md)) continue unaffected.

## 7. Data drift testing

- Golden datasets and production input distributions (for example, typical
  telemetry ranges, typical visitor question topics) are compared periodically
  to detect drift — a sign that the estate's actual conditions have diverged
  from what the model was built or last validated against (for example, as
  attendance grows toward 15,000/day).
- Detected drift beyond a defined threshold triggers a scheduled
  re-evaluation of the affected capability against updated data, rather than
  waiting for a visible quality failure.

## 8. Cost monitoring

- Inference cost per AI capability is tracked continuously as one of the
  technical health metrics defined in
  [../adrs/ADR-006-ai-monitoring-and-evaluation.md](../adrs/ADR-006-ai-monitoring-and-evaluation.md).
- A defined cost-per-visitor or cost-per-alert guardrail (design target, to be
  set once real usage data exists) triggers review if breached, feeding into
  [cost-and-value-model.md](cost-and-value-model.md) once available.

## 9. Latency monitoring

- End-to-end latency (from data capture to a visible recommendation or alert)
  is monitored per capability against the technical metrics defined per
  use case in [success-metrics.md](success-metrics.md).
- Latency breaches are distinguished from AI-quality breaches, since a slow
  but accurate capability and a fast but low-quality one require different
  responses.

## 10. False positive / false negative monitoring

- Each capability's false-positive and false-negative rates are computed
  continuously against the human-confirmed outcomes recorded under
  [../adrs/ADR-004-human-oversight-and-ai-governance.md](../adrs/ADR-004-human-oversight-and-ai-governance.md).
- For [Animal Health
  Intelligence](../use-cases/animal-health-intelligence.md), the false-negative
  rate is treated as the primary quality signal given the welfare stakes (per
  [../adrs/ADR-003-animal-health-anomaly-detection.md](../adrs/ADR-003-animal-health-anomaly-detection.md)),
  and breaching its threshold triggers the mandatory disable process described
  in that use case document.

## 11. Audit of decisions

- Every AI output (recommendation, alert, ranking) and every human decision
  made on it (confirm, dismiss, escalate, accept, reject) is logged in a
  tamper-evident way, building on the Event Backbone's ordered event log (see
  [../architecture/event-driven-architecture.md](../architecture/event-driven-architecture.md)
  and [../architecture/security-and-privacy.md](../architecture/security-and-privacy.md)).
- This audit trail is the source of truth for all quality metrics above and
  supports incident investigation if a serious error occurs.

## 12. Rollback process

- Every AI capability's model/provider sits behind the abstraction layer
  defined in
  [../adrs/ADR-001-ai-model-abstraction.md](../adrs/ADR-001-ai-model-abstraction.md),
  allowing a previous, known-good version to be restored without redesigning
  the capability's data flow or consumers.
- A rollback is triggered by a regression detected in golden-dataset testing,
  a production quality-threshold breach, or a manual decision by the
  accountable human role and the system administrator.
- Rollback restores the previous version's contract behaviour; any data
  produced under the faulty version is flagged in the audit trail for review,
  not silently discarded.

## 13. Escalation thresholds to a human

Each capability defines explicit thresholds (set as design targets pending real
data, per [success-metrics.md](success-metrics.md)) at which quality
degradation automatically triggers escalation to a human role, beyond the
routine human-in-the-loop review already required by
[../adrs/ADR-004-human-oversight-and-ai-governance.md](../adrs/ADR-004-human-oversight-and-ai-governance.md):

- A breach of the defined false-negative threshold for Animal Health
  Intelligence escalates to the system administrator and a veterinarian
  jointly (per
  [../adrs/ADR-003-animal-health-anomaly-detection.md](../adrs/ADR-003-animal-health-anomaly-detection.md)).
- A breach of forecast-accuracy or false-alert thresholds for Visitor
  Intelligence escalates to the system administrator and the park operator.
- A breach of groundedness or visitor-reported-error thresholds for
  Personalised Visitor Experience escalates to the system administrator and
  the estate staff responsible for the approved knowledge base.

## 14. Disabling a specific AI function

Any single AI function (for example, only the "correlation indication" part of
Animal Health Intelligence, or only "next-visit recommendation" within
Personalised Visitor Experience) can be disabled independently of the rest of
its use case, falling back to that function's defined fallback behaviour,
without disabling the entire capability. This fine-grained control follows
from the same abstraction-layer design in
[../adrs/ADR-001-ai-model-abstraction.md](../adrs/ADR-001-ai-model-abstraction.md)
and is intended to let the estate respond proportionately to a localised
problem rather than losing an entire capability over one underperforming
function.

## Summary: validation coverage per use case

| Technique | Visitor Intelligence | Animal Health Intelligence | Personalised Visitor Experience |
|---|---|---|---|
| Golden dataset | Historical attendance/occupancy | Confirmed telemetry/observation outcomes | Question/approved-answer pairs |
| Regression testing | Forecast/alert accuracy vs. previous version | Precision/recall vs. previous version | Groundedness/relevance vs. previous version |
| Prompt testing | Not applicable (no generative text) | Not applicable | Yes — retrieval and refusal behaviour |
| Classification/anomaly testing | Congestion-alert precision/recall | Anomaly precision/recall, per-enclosure baseline confidence | Not applicable |
| Missing/delayed data testing | Yes | Yes | Yes (availability/queue data) |
| Offline testing | Manual-planning fallback | Rule-based threshold fallback | Static content/rule-based fallback |
| Data drift testing | Yes | Yes | Yes |
| Cost/latency monitoring | Yes | Yes | Yes |
| False positive/negative monitoring | Yes | Yes (primary signal: false negatives) | N/A (tracked as incorrect-information rate) |
| Audit of decisions | Yes | Yes | Yes |
| Rollback | Yes | Yes | Yes |
| Escalation thresholds | Yes | Yes (strictest) | Yes |
| Disable a specific function | Yes | Yes | Yes |
