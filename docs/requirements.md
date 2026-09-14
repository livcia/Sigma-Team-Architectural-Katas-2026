# Requirements

This document translates the business context
([business-context.md](business-context.md)) into requirements that later
architecture and use-case documents must satisfy. It does not select technologies
or designs — see [../architecture/](../architecture/) for the resulting design.

## Functional requirements

### Ticketing and access
- FR1: Sell individual and family/group passes for park access (day passes at
  minimum; other pass types are a design choice for later documents).
- FR2: Record entry/exit events per visitor or group at a granularity sufficient to
  support zone occupancy and demand analysis.

### Visitor and demand intelligence
- FR3: Capture attraction usage, zone occupancy and queue-related data across the
  estate.
- FR4: Combine estate data with external context (weather, scheduled events) to
  support demand forecasting.
- FR5: Present popularity, congestion and demand information to park operators in a
  form that supports staffing and operational decisions.
- FR6: Recommend operational actions (for example, staffing or queue-management
  suggestions) without silently executing them.

### Animal health and welfare
- FR7: Collect enclosure telemetry (for example environmental conditions),
  feeding records, keeper observations and animal activity data.
- FR8: Detect anomalies in this data and prioritise cases for keeper or
  veterinary attention.
- FR9: Present alerts with enough explanation (contributing signals) that a keeper
  or veterinarian can assess them without blindly trusting the model.
- FR10: Allow a keeper or veterinarian to confirm, dismiss or escalate an alert,
  and record that decision.

### Personalised visitor experience
- FR11: Recommend routes, attractions or activities to a visitor based on current
  availability, queues and (where provided) visitor preferences.
- FR12: Provide information about animals, attractions and events grounded in
  estate-approved content, not open-ended generation.
- FR13: Recommend reasons or offers for a future visit to encourage return visits.
- FR14: Allow a visitor to give feedback on a recommendation (helpful/not helpful,
  or a correction).
- FR15: Provide a non-AI fallback experience (static information, simple rules)
  when personalisation is unavailable.

## Non-functional requirements

- NFR1 (Resilience): The system must continue core operations (ticketing, entry,
  data capture) during Wi-Fi outages in parts of the estate, buffering data locally
  until connectivity returns.
- NFR2 (Latency): Visitor-facing recommendations and alerts must be provided
  quickly enough to be operationally useful (this is a per-use-case, measurable
  target — see [success-metrics.md](success-metrics.md) once available).
- NFR3 (Scalability): The system must handle growth from approximately 5,000 to
  15,000 daily visitors without redesign, only added capacity.
- NFR4 (Maintainability): AI models and providers must be replaceable without
  rewriting the surrounding system (see ADR on AI model abstraction).
- NFR5 (Explainability): Any AI output used for a health, safety or operational
  decision must be accompanied by a human-understandable explanation of its
  contributing factors.
- NFR6 (Auditability): All AI-driven alerts, recommendations and the human
  decisions taken on them must be recorded for later review.
- NFR7 (Observability): The system must expose enough monitoring (latency, error
  rate, fallback rate, AI quality metrics) to detect degraded behaviour in
  production.

## Technical constraints

- TC1: Wi-Fi coverage across the estate is patchy; the system cannot assume
  continuous connectivity from every device or zone.
- TC2: Cloud services may be used, but estate data must be transported to the cloud
  deliberately — the design must define how and when this happens.
- TC3: A budget exists specifically for MQTT-capable hardware devices; sensing and
  telemetry should be designed around MQTT as the primary device protocol.
- TC4: The physical estate consists of 40 amusement rides and 55 animal
  displays/enclosures spread across a large area, meaning any local infrastructure
  (gateways, buffering) must be zonal, not centralised in one location.

## AI-specific requirements

- AI1: Every AI capability must be replaceable at the model/provider level via an
  abstraction layer, without redesigning the capability's data flow or consumers.
- AI2: No AI capability may autonomously execute a safety-critical, veterinary,
  financial or irreversible action; it may only recommend, alert, or prioritise for
  a human.
- AI3: Every AI capability must define a fallback behaviour for when the model,
  its data, or connectivity is unavailable.
- AI4: Every AI capability must have defined quality metrics (for example
  precision/recall or forecast accuracy) evaluated against historical or labelled
  data before and after any model change.
- AI5: Any AI capability that generates visitor-facing text or recommendations must
  ground its output in estate-approved information, and must avoid presenting
  unverified or fabricated claims as fact.
- AI6: AI capabilities must be monitored in production for degraded quality (drift,
  rising false-positive/false-negative rate, rising cost or latency) with a defined
  escalation and rollback path.

## Offline/edge requirements

- OE1: MQTT telemetry and events generated in zones with unreliable Wi-Fi must be
  buffered locally (at an edge gateway) and forwarded once connectivity is
  restored, without data loss under normal outage durations.
- OE2: Core operational functions (ticketing, entry, basic ride/enclosure
  monitoring) must not depend on the cloud or on AI being reachable.
- OE3: Each AI-driven capability must degrade gracefully — to cached results,
  static content, or simple deterministic rules — when edge-to-cloud connectivity
  or the AI service itself is unavailable.
- OE4: Synchronisation from edge to cloud must preserve event ordering and
  timestamps sufficient for later analysis, even when events arrive out of order
  due to buffering.

## Security, privacy and audit requirements

- SPA1: Visitor personal data (identity, payment, preferences) must be protected
  with access control appropriate to its sensitivity, and separated from
  operational/analytical data used for aggregate demand analysis.
- SPA2: Data used for visitor-flow and demand analytics should be anonymised or
  aggregated wherever individual identity is not required for the analysis.
- SPA3: Animal-health data, while not personal data, must be access-controlled
  and auditable given its welfare and reputational sensitivity.
- SPA4: All AI-driven alerts and recommendations, and the human decisions made in
  response, must be logged for audit in a way that is tamper-evident.
- SPA5: Any collection of visitor preference data for personalisation must be
  opt-in, with a clear description of what is collected and why.
- SPA6: Data in transit between edge devices, gateways and the cloud must be
  protected against interception and tampering.

## Measurable success criteria

These are stated at a high level here; per-use-case detail is in
[success-metrics.md](success-metrics.md) once available. All numeric values below
are design targets/assumptions, not measured facts, unless sourced from the brief.

- SC1: Reduction in unplanned understaffing/overstaffing incidents attributable to
  better demand visibility (baseline and target to be established operationally).
- SC2: Reduction in average time between an animal-health issue developing and a
  keeper/veterinarian reviewing it, compared to the current undocumented process.
- SC3: Increase in repeat-visit rate (visitors returning within a defined period)
  attributable to personalised recommendations.
- SC4: The estate sustains growth toward 15,000 daily visitors (brief target)
  without a proportional rise in operational incidents or animal-care escalations.
- SC5: AI capabilities maintain their defined quality thresholds in production, with
  breaches triggering fallback or human review rather than silent degradation.

## Assumptions used in this document

Several requirements above assume estate-side capabilities (a visitor app, opt-in
preference collection, keeper/veterinary workflows) that the brief does not describe
in detail. These are listed explicitly in [assumptions.md](assumptions.md).
