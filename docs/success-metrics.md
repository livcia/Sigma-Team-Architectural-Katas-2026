# Success metrics

This document defines, for each of the three AI use cases, the business metrics,
AI quality metrics, technical metrics, guardrails and fallback behaviour used to
judge whether the capability is working. It complements
[architecture-characteristics.md](architecture-characteristics.md) and the
measurable success criteria in [requirements.md](requirements.md) (SC1–SC5).

All specific numeric thresholds below are **design targets or illustrative
assumptions**, clearly labelled as such, to be replaced with real baselines once
the estate has operating data (consistent with [assumptions.md](assumptions.md),
D4). None should be read as a measured fact about the current estate.

Detailed per-use-case behaviour, actors and data flows are described in
[../use-cases/](../use-cases/) once available; this document focuses only on how
success and failure are measured and handled.

## Visitor Intelligence

Addresses: no visibility into popularity and demand (see
[business-context.md](business-context.md)).

### Business metrics
- Reduction in visitor-reported congestion complaints (baseline to be
  established; target is a relative reduction, not an absolute number).
- Improvement in staff-to-demand alignment, for example measured as reduced
  variance between planned and actual staffing needs per zone/day (design
  target).
- Contribution to sustained growth toward the 15,000 daily-visitor target (SC4)
  without a proportional rise in operational incidents.

### AI quality metrics
- Forecast accuracy for expected attendance/zone occupancy (for example, error
  between forecast and actual counts), evaluated against historical data once
  available.
- Precision/recall of congestion or queue-buildup alerts against operator-labelled
  ground truth (was this actually a congestion event?).
- Stability of forecasts under known confounders (weather, scheduled events) —
  tested rather than assumed.

### Technical metrics
- End-to-end latency from telemetry capture to an operator-visible forecast or
  alert (design target: fast enough to inform same-day staffing decisions).
- Data completeness rate (percentage of expected zone/occupancy telemetry
  actually received, accounting for buffered/delayed edge data).
- Availability of the forecasting/alerting service, tracked separately from core
  ticketing availability (per the separation of concerns in
  [architecture-characteristics.md](architecture-characteristics.md)).

### Guardrails
- Forecasts and congestion alerts must always be presented as recommendations
  with a confidence indicator, never as an instruction the system executes
  automatically (AI2 in [requirements.md](requirements.md)).
- Alerts must not fire on data recognised as significantly incomplete or stale;
  the system should indicate reduced confidence rather than a false-confident
  recommendation.
- A maximum acceptable false-alert rate should be defined operationally once
  baseline data exists, to prevent operator alert fatigue.

### Fallback after threshold breach
- If forecast quality or data completeness falls below its defined threshold,
  the system falls back to presenting **raw historical patterns or the last
  known reliable forecast**, clearly labelled as degraded, rather than a
  potentially misleading fresh prediction.
- If the forecasting/alerting service itself is unavailable, operators fall back
  to their existing manual planning process, with no change to core ticketing or
  entry operations (OE2 in [requirements.md](requirements.md)).

## Animal Health Intelligence

Addresses: high animal-care costs and late detection of health problems (see
[business-context.md](business-context.md)).

### Business metrics
- Reduction in time between an environmental or health anomaly developing and a
  keeper/veterinarian reviewing it, compared to the current undocumented process
  (SC2 in [requirements.md](requirements.md); baseline to be established).
- Reduction in cost or severity of veterinary interventions attributable to
  earlier detection (design target, requires historical cost data to validate).
- Reduction in preventable animal-health incidents over time (design target).

### AI quality metrics
- Precision and recall of anomaly detection against keeper/veterinarian-confirmed
  incidents (a labelled outcome dataset built over time, per AI4 in
  [requirements.md](requirements.md)).
- False-positive rate (alerts dismissed by a keeper/veterinarian as not genuine)
  and false-negative rate (missed incidents later confirmed through other means),
  tracked separately since they have different costs.
- Explainability coverage: percentage of alerts accompanied by a clear
  contributing-factor explanation (NFR5), which should be effectively 100% given
  the welfare stakes.

### Technical metrics
- Latency from sensor/keeper-observation capture to alert generation, including
  time added by edge buffering during Wi-Fi outages.
- Telemetry data completeness per enclosure (percentage of expected readings
  actually received).
- Availability and freshness of the anomaly-detection pipeline, monitored
  separately from the edge devices themselves so a device fault is
  distinguishable from a pipeline fault.

### Guardrails
- The system must never present an anomaly alert as a diagnosis or a treatment
  instruction; it only prioritises attention for a qualified human (AI2, FR9 in
  [requirements.md](requirements.md)).
- Every alert must be explainable and must be logged with the eventual human
  decision (confirm/dismiss/escalate) for audit (SPA4, FR10).
- A defined maximum false-negative rate on confirmed historical incidents should
  trigger mandatory model review before continued unsupervised use — this
  capability carries the highest welfare and reputational stakes of the three use
  cases and should be held to the strictest guardrail.

### Fallback after threshold breach
- If anomaly-detection quality degrades below its defined threshold (rising false
  negatives in particular), the capability should fall back to a **simpler
  rule-based threshold alerting** (for example, fixed environmental thresholds)
  while the model is reviewed, rather than being silently trusted or fully
  disabled.
- If connectivity to an enclosure's edge gateway is lost, buffered telemetry is
  synchronised once restored (OE1); in the interim, keepers rely on their normal
  manual observation routine, which remains the ultimate safety net (OE2).

## Personalised Visitor Experience

Addresses: few reasons for visitors to return (see
[business-context.md](business-context.md)).

### Business metrics
- Increase in repeat-visit rate within a defined period, attributable to
  personalised recommendations (SC3 in [requirements.md](requirements.md);
  baseline and period to be established once the visitor app exists).
- Visitor engagement with recommendations (for example, recommendation
  acceptance/click-through rate) as a leading indicator ahead of return-visit
  data being available.
- Positive/negative feedback ratio on recommendations (FR14), tracked as a proxy
  for perceived relevance and quality.

### AI quality metrics
- Groundedness rate: percentage of visitor-facing informational responses that
  are verifiably grounded in estate-approved content, with no unsupported claims
  (AI5 in [requirements.md](requirements.md)).
- Recommendation relevance, measured via visitor feedback (FR14) and, where
  available, acceptance of recommended routes/attractions.
- Rate of visitor-reported incorrect or misleading information, which should
  trend toward zero given the groundedness requirement.

### Technical metrics
- Latency from a visitor request to a recommendation being shown (design target:
  fast enough to be used in-the-moment, e.g., while deciding what to do next).
- Availability of the recommendation service, tracked separately from the
  fallback static content described below.
- Freshness of the availability/queue data used to ground recommendations
  (stale operational data undermines trust even if the AI itself is "correct").

### Guardrails
- Recommendations must only draw on an approved knowledge base and current
  operational data, not open-ended generation (FR12, AI5).
- No preference or behavioural data is used for personalisation unless the
  visitor has explicitly opted in (SPA5, V2 in [assumptions.md](assumptions.md)).
- The system must clearly distinguish current, verified information (for
  example, "open now") from potentially outdated content, and must not present
  stale information as current.
- Visitors must always have a visible way to report a bad recommendation (FR14).

### Fallback after threshold breach
- If groundedness or relevance quality falls below its defined threshold, or the
  AI/recommendation service is unavailable, the app falls back to **static,
  curated estate information and simple rule-based suggestions** (for example,
  "popular today" lists derived from Visitor Intelligence data rather than
  personalised AI output) (FR15, OE3 in [requirements.md](requirements.md)).
- Visitors without the app, or during a full fallback, still receive a complete,
  non-personalised experience (V3 in [assumptions.md](assumptions.md)).

## Cross-cutting note on thresholds

None of the three use cases should ship with an AI capability that lacks a
defined quality threshold and fallback — this is a requirement (AI3, AI4 in
[requirements.md](requirements.md)), not an optional refinement. Concrete
threshold values are intentionally left as "to be established" here because
setting them prematurely, without operational data, would be an unsupported
precision the estate cannot yet justify. The process for setting and revisiting
these thresholds is described further in
[validation-and-fitness-functions.md](validation-and-fitness-functions.md) once
available.
