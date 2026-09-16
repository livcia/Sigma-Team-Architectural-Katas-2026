Date: 2026-09-14

# ADR-006: AI monitoring and evaluation

## Status
Proposed

## Context

AI outputs across all three use cases are non-deterministic and can degrade
silently over time (for example, due to data drift, a changed visitor
population as attendance grows toward 15,000/day, or a model/provider change
under ADR-001). The competition brief explicitly asks how the team will detect
when AI "starts misbehaving in production," and
[../docs/architecture-characteristics.md](../docs/architecture-characteristics.md)
identifies testability as a top architecture characteristic. Without a
monitoring and evaluation approach defined once and applied consistently, each
use case might only discover a problem when a human notices a bad outcome —
which is especially unacceptable for
[Animal Health Intelligence](../use-cases/animal-health-intelligence.md), given
the welfare stakes described in ADR-003.

Alternatives considered:

- **No systematic monitoring**, relying only on ad hoc complaints or incident
  reports from operators, keepers or visitors. Rejected: this is exactly the
  "late detection" pattern this whole solution is designed to move away from,
  now applied to the AI itself.
- **Monitoring only technical metrics** (latency, error rate, cost) without
  tracking AI quality (accuracy, precision/recall, groundedness). Rejected:
  a technically healthy but low-quality model (for example, one producing
  plausible-looking but wrong forecasts or alerts) would not be caught.
- **A consistent monitoring and evaluation approach** combining technical
  health, AI quality (using the human decisions recorded under ADR-004 as
  ground truth), and defined thresholds that trigger fallback or human review,
  applied the same way across all three use cases. This is the approach
  adopted.

## Decision

Each AI capability is monitored and evaluated using three consistent
categories, detailed per use case in
[../docs/success-metrics.md](../docs/success-metrics.md):

1. **AI quality metrics** (for example, forecast accuracy, precision/recall,
   groundedness rate), evaluated continuously against a growing set of
   human-confirmed outcomes captured through the governance model in ADR-004
   (confirm/dismiss/escalate decisions, visitor feedback).
2. **Technical health metrics** (latency, data completeness, service
   availability, cost per inference), monitored separately from quality so a
   technically available but low-quality model is not mistaken for a healthy
   one.
3. **Golden/regression datasets**, used to evaluate any proposed model or
   provider change (under ADR-001) before it is deployed, and re-run
   periodically in production to detect drift.

Each capability has an explicit **quality threshold** (for example, a maximum
acceptable false-negative rate for Animal Health Intelligence, per ADR-003)
that, when breached, automatically triggers that capability's defined fallback
(rule-based alerting, manual planning, or static content) and flags the
capability for human review — rather than continuing to serve degraded AI
output or being silently disabled. The system administrator owns this
monitoring; re-enabling a capability after a breach requires review by the
relevant accountable human role (for example, jointly with a veterinarian for
Animal Health Intelligence, per ADR-003).

## Consequences

**Positive:**
- Degrading AI quality is caught through defined thresholds rather than
  waiting for a visible real-world failure, directly addressing the brief's
  validation-and-verification judging criterion.
- Using recorded human decisions as ground truth (from ADR-004) means
  evaluation data accumulates naturally from normal operation, rather than
  requiring a separate, costly labelling exercise.
- A consistent approach across all three use cases makes it easier to reason
  about overall AI risk and to build shared tooling/dashboards, rather than
  three bespoke monitoring setups.

**Negative / trade-offs:**
- Meaningful quality metrics require enough confirmed outcomes to be
  statistically useful; early on (greenfield system, assumption A1 in
  [../docs/assumptions.md](../docs/assumptions.md)), thresholds may need to be
  set conservatively and revised as real data accumulates.
- Maintaining golden/regression datasets and re-running them periodically adds
  ongoing operational effort, distinct from one-off testing at launch.
- Automatic fallback on threshold breach could itself cause disruption (for
  example, reverting to rule-based alerting) if thresholds are set too
  sensitively; threshold tuning is an ongoing responsibility, not a one-time
  configuration.

**Operational impact:**
- The system administrator monitors technical and quality metrics on an
  ongoing basis and initiates the fallback/review process on breach.
- Accountable human roles (park operator, keeper/veterinarian) participate in
  generating the ground-truth data this approach depends on, by consistently
  recording their decisions per ADR-004 — without this participation,
  evaluation quality degrades.
- Cost monitoring here also feeds into
  [../docs/cost-and-value-model.md](../docs/cost-and-value-model.md) once
  available, since inference cost is one of the technical health metrics
  tracked.

**Reversibility:**
- Specific thresholds and dataset composition can be revised at any time as
  more operational data accumulates, without changing this decision's core
  structure (three metric categories, threshold-triggered fallback, human-owned
  re-enablement). A move to a different evaluation approach entirely (for
  example, continuous automated A/B testing across live traffic) would warrant
  a new ADR superseding this one.
