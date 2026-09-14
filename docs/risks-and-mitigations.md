# Risks and mitigations

This document lists the main risks associated with the AI-driven solution for
Von Digitalis Estates and how each is managed. It complements
[validation-and-fitness-functions.md](validation-and-fitness-functions.md) (how
we detect problems) with an explicit view of likelihood, impact, mitigation,
ownership and contingency for each risk — going beyond model/provider risk to
cover the full range of AI, data and operational risks relevant to this estate.

Likelihood and impact ratings below (Low/Medium/High) are **qualitative
judgements**, not measured probabilities, and should be revisited once real
operational data exists (consistent with [assumptions.md](assumptions.md)).

## How to read this document

Each risk includes:
- **Likelihood** and **Impact** (Low/Medium/High).
- **Mitigation mechanism** — the architectural or process control that reduces
  likelihood or impact.
- **Owner** — the role accountable for responding if the risk materialises (see
  [stakeholders-and-personas.md](stakeholders-and-personas.md)).
- **Contingency plan** — what happens if the risk materialises despite
  mitigation.

## 1. Hallucinated or fabricated visitor information

- **Likelihood:** Medium — inherent risk in any generative response to
  open-ended questions.
- **Impact:** High — a visitor given incorrect information about a hazardous
  animal, ride safety, or an event could be harmed or misled, directly damaging
  trust (the core problem this use case exists to solve).
- **Mitigation:** Retrieval-augmented generation restricted to the approved
  knowledge base, with mandatory refusal when no relevant approved passage
  exists, and response-to-source consistency checking (see
  [../adrs/ADR-005-rag-for-approved-visitor-information.md](../adrs/ADR-005-rag-for-approved-visitor-information.md)
  and prompt testing in
  [validation-and-fitness-functions.md](validation-and-fitness-functions.md)).
- **Owner:** System administrator (technical control), estate staff responsible
  for the knowledge base (content accuracy).
- **Contingency:** Visitor-reported incorrect information (FR14) triggers
  immediate review of the relevant knowledge-base entry and retrieval
  behaviour; the capability falls back to static content if groundedness
  quality breaches its threshold.

## 2. False alerts (false positives) causing alert fatigue

- **Likelihood:** Medium — expected during initial deployment before
  thresholds are tuned.
- **Impact:** Medium (Visitor Intelligence: wasted staffing effort) to High
  (Animal Health Intelligence: keepers may start discounting genuine alerts).
- **Mitigation:** Explainable, leveled alerts (Watch/Priority check/Escalate)
  so low-confidence alerts are visibly distinguished from high-confidence ones;
  false-positive rate tracked continuously (see
  [../adrs/ADR-006-ai-monitoring-and-evaluation.md](../adrs/ADR-006-ai-monitoring-and-evaluation.md)).
- **Owner:** System administrator, with the accountable operational role
  (park operator or keeper) providing confirm/dismiss feedback.
- **Contingency:** Sustained high false-positive rate triggers threshold
  re-tuning or temporary fallback to simpler rule-based alerting.

## 3. Missed alerts (false negatives)

- **Likelihood:** Low to Medium, but the most consequential failure mode for
  Animal Health Intelligence.
- **Impact:** High — a missed health or environmental anomaly can become
  costly or irreversible for animal welfare, and reputationally damaging for
  the estate.
- **Mitigation:** Per-enclosure baseline anomaly detection (see
  [../adrs/ADR-003-animal-health-anomaly-detection.md](../adrs/ADR-003-animal-health-anomaly-detection.md)),
  mandatory disable-and-review process on false-negative threshold breach, and
  the keeper's manual observation routine retained as an independent safety
  net regardless of AI status.
- **Owner:** System administrator and veterinarian jointly.
- **Contingency:** Immediate fallback to rule-based threshold alerting;
  incident review to determine root cause before AI-generated alerting is
  re-enabled.

## 4. Data leak or unauthorised access to sensitive data

- **Likelihood:** Low, if controls are implemented as designed; Medium if
  access control or encryption is misconfigured.
- **Impact:** High — exposure of visitor personal/payment data or animal-health
  records would cause regulatory, financial and reputational harm.
- **Mitigation:** Separation of operational and analytical data, role-scoped
  access control, encryption in transit, and audit logging of access to
  sensitive data (see
  [../architecture/security-and-privacy.md](../architecture/security-and-privacy.md)).
- **Owner:** System administrator.
- **Contingency:** Defined incident-response process to revoke access,
  investigate the audit trail, and notify affected parties if personal data
  exposure is confirmed.

## 5. Bias in AI recommendations or alerts

- **Likelihood:** Medium — any model trained or tuned on limited early data
  risks skewing toward the patterns most represented in that data (for
  example, better-monitored enclosures or more common visitor profiles).
- **Impact:** Medium — could mean some enclosures or visitor groups receive
  systematically lower-quality service (less accurate anomaly detection, less
  relevant recommendations) than others.
- **Mitigation:** Per-enclosure (not species-wide) baselines for Animal Health
  Intelligence reduce one form of bias; golden datasets and drift testing (see
  [validation-and-fitness-functions.md](validation-and-fitness-functions.md))
  are checked for representativeness across enclosures/zones, not only
  aggregate accuracy.
- **Owner:** System administrator, with input from keepers/veterinarians and
  park operators on whether any group is being systematically underserved.
- **Contingency:** If bias is identified, affected enclosures/zones are
  flagged for closer manual attention while the underlying model or data
  collection is adjusted.

## 6. Data drift as the estate scales

- **Likelihood:** High over a multi-year horizon — attendance is expected to
  grow from 5,000 to 15,000 daily visitors, and the animal collection or ride
  set may change.
- **Impact:** Medium — gradually degrades forecast and detection quality if
  unaddressed, though it develops slowly rather than suddenly.
- **Mitigation:** Scheduled drift testing comparing production data
  distributions to golden datasets (see
  [validation-and-fitness-functions.md](validation-and-fitness-functions.md)),
  triggering re-evaluation before quality visibly degrades.
- **Owner:** System administrator.
- **Contingency:** Scheduled model re-evaluation or retraining/reconfiguration;
  temporary reliance on wider confidence bands or rule-based fallback if drift
  is severe.

## 7. Cloud unavailability

- **Likelihood:** Low but non-zero — any cloud platform can have an outage.
- **Impact:** Medium to High, depending on duration — analytics, AI outputs and
  the Visitor App/Portal depend on the cloud platform, though core zone-level
  data capture does not (see
  [../architecture/target-state.md](../architecture/target-state.md)).
- **Mitigation:** Separation of transactional, operational and analytical
  concerns means a cloud outage does not stop MQTT telemetry capture or local
  buffering; each AI capability's fallback (manual planning, rule-based
  alerting, static content) does not depend on the cloud being reachable.
- **Owner:** System administrator.
- **Contingency:** Estate operates in full offline/fallback mode across all
  three use cases until cloud connectivity is restored; buffered zone data
  synchronises once it returns (see
  [../architecture/data-flow.md](../architecture/data-flow.md)).

## 8. Loss of estate Wi-Fi connectivity

- **Likelihood:** High — explicitly described as "patchy" in the brief; treated
  as a normal operating condition, not a rare event.
- **Impact:** Low to Medium, given the mitigation already designed in from the
  start.
- **Mitigation:** Zone Edge Gateway with Local Buffer Store per zone, preserving
  event order and timestamps until connectivity returns (see
  [../adrs/ADR-002-edge-buffering-and-offline-mode.md](../adrs/ADR-002-edge-buffering-and-offline-mode.md)).
- **Owner:** System administrator (technical), keepers/operators (manual
  fallback during the outage).
- **Contingency:** Manual observation and existing planning processes remain
  the safety net during an outage; no data is lost, only delayed.

## 9. Rising AI inference costs as usage scales

- **Likelihood:** Medium to High — inference volume grows with visitor
  numbers and with richer personalisation.
- **Impact:** Medium — could erode the cost savings this solution is meant to
  deliver if unmanaged.
- **Mitigation:** Continuous cost-per-capability monitoring (see
  [../adrs/ADR-006-ai-monitoring-and-evaluation.md](../adrs/ADR-006-ai-monitoring-and-evaluation.md)),
  with a defined cost guardrail that triggers review; the AI abstraction layer
  (see
  [../adrs/ADR-001-ai-model-abstraction.md](../adrs/ADR-001-ai-model-abstraction.md))
  allows switching to a lower-cost model/provider without redesigning the
  capability.
- **Owner:** System administrator, with cost trade-offs reviewed further in
  [cost-and-value-model.md](cost-and-value-model.md) once available.
- **Contingency:** Reduce inference frequency or scope (for example, batching
  forecasts less often, limiting personalisation depth) while a longer-term
  cost fix is evaluated.

## 10. AI model/provider becomes outdated, re-priced or discontinued

- **Likelihood:** Medium to High over a multi-year horizon — explicitly raised
  as a risk in the competition brief.
- **Impact:** Medium — disruptive if not planned for, but well mitigated by
  design.
- **Mitigation:** AI abstraction/gateway layer (see
  [../adrs/ADR-001-ai-model-abstraction.md](../adrs/ADR-001-ai-model-abstraction.md))
  keeps model/provider swaps isolated from consuming components; golden
  datasets allow a replacement to be validated before adoption.
- **Owner:** System administrator.
- **Contingency:** Roll back to the previous model/provider (see
  [validation-and-fitness-functions.md](validation-and-fitness-functions.md),
  "Rollback process") while a replacement is evaluated.

## Summary table

| # | Risk | Likelihood | Impact | Owner |
|---|---|---|---|---|
| 1 | Hallucinated visitor information | Medium | High | System administrator / knowledge-base staff |
| 2 | False alerts (false positives) | Medium | Medium–High | System administrator / operator or keeper |
| 3 | Missed alerts (false negatives) | Low–Medium | High | System administrator / veterinarian |
| 4 | Data leak / unauthorised access | Low–Medium | High | System administrator |
| 5 | Bias in AI outputs | Medium | Medium | System administrator |
| 6 | Data drift with scale | High | Medium | System administrator |
| 7 | Cloud unavailability | Low | Medium–High | System administrator |
| 8 | Wi-Fi connectivity loss | High | Low–Medium | System administrator / operators / keepers |
| 9 | Rising inference cost | Medium–High | Medium | System administrator |
| 10 | Model/provider obsolescence | Medium–High | Medium | System administrator |

This list is not exhaustive; it reflects the risks judged most material given
the business context in [business-context.md](business-context.md) and the
requirements in [requirements.md](requirements.md). New risks identified during
further design or operation should be added here rather than left undocumented.
