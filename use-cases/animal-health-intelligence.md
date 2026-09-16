# Use case: Animal Health Intelligence

## Business problem

Von Digitalis Estates cares for more than 200 exotic animals across 55 displays
and enclosures. Veterinary and keeper time is scarce and expensive, and today
health or environmental problems are typically discovered only when they become
visibly serious — by which point treatment is costlier and the animal is at
greater risk (see [../docs/business-context.md](../docs/business-context.md)).
This use case addresses the second core problem: **high animal-care costs and
late detection of health problems**, by turning enclosure telemetry, feeding
records and keeper observations into prioritised, explainable attention for
keepers and veterinarians.

This document assumes the baseline operational architecture in
[../architecture/target-state.md](../architecture/target-state.md) already
exists, including MQTT devices, the Zone Edge Gateway, local buffering, cloud
synchronisation, and the Event Backbone. This use case adds an AI capability on
top, without changing that baseline.

## Actors

- **Animal keeper** — receives prioritised alerts, logs observations, and
  confirms/dismisses/escalates alerts (see
  [../docs/stakeholders-and-personas.md](../docs/stakeholders-and-personas.md)).
- **Veterinarian** — receives escalated alerts requiring specialist judgement and
  makes the final veterinary decision.
- **System administrator** — monitors the AI capability's health, manages
  model/provider changes, and can disable an underperforming model.

## Data inputs

- **Environmental telemetry** per enclosure: temperature, humidity and water
  quality, published by MQTT devices (`enclosure.telemetry` events).
- **Feeding records**: quantities, timing and type of feeding per enclosure.
- **Keeper observations**: structured or semi-structured notes logged through
  the Operator Console (`keeper.observation` events), for example about
  behaviour, appearance or appetite.
- **Activity data**: derived from telemetry or observation where available (for
  example, movement/activity indicators from enclosure sensors).
- **Incident history**: past confirmed health or environmental incidents for
  the same enclosure or animal, held in the Analytical Data Store.

## Data sources and edge handling

MQTT devices are deployed at the enclosure/zone level (assumption M1 in
[../docs/assumptions.md](../docs/assumptions.md)) and publish telemetry to their
**Zone Edge Gateway**, exactly as described in
[../architecture/target-state.md](../architecture/target-state.md) and the
deployment layout in
[../architecture/deployment-view.md](../architecture/deployment-view.md). When
Wi-Fi is unavailable, the gateway buffers telemetry in its **Local Buffer
Store**, preserving order and original timestamps, and forwards it to the
**Cloud Sync Service** once connectivity returns — the exact flow described in
[../architecture/data-flow.md](../architecture/data-flow.md), Flow 3. Feeding
records and keeper observations, entered through the Operator Console, follow
the same buffered path when a keeper is working in a zone with degraded
connectivity.

## Data flow

1. Enclosure telemetry (temperature, humidity, water quality) and, where
   available, activity signals are published by MQTT devices to the Zone Edge
   Gateway.
2. During a Wi-Fi outage, telemetry accumulates in the Local Buffer Store; the
   gateway keeps accepting new readings without loss.
3. Once connectivity resumes, the Cloud Sync Service forwards buffered events
   in original order, deduplicates them, and publishes them to the **Event
   Backbone**, marked as delayed where applicable.
4. Feeding records and keeper observations reach the Event Backbone the same
   way, whether entered directly (connected) or buffered (disconnected).
5. The Event Backbone updates the Operational Data Store (current enclosure
   state) and the Analytical Data Store (history, including confirmed past
   incidents).
6. The **Animal Health Intelligence AI component** reads current and historical
   data for each enclosure and produces anomaly detections, prioritised alerts,
   environmental-change flags and possible correlations.
7. Alerts and their explanations are written to the Operational Data Store and
   surfaced through the Operator Console to keepers and, when escalated, to
   veterinarians.
8. Keeper/veterinarian decisions (confirm, dismiss, escalate) are recorded back
   into the Operational Data Store, closing the feedback loop used to evaluate
   and improve the model.

See [../diagrams/ai-animal-health.md](../diagrams/ai-animal-health.md) for the
corresponding diagram.

## Role of AI

The AI component performs four functions, all advisory:

1. **Anomaly detection** — flags enclosure telemetry, feeding or activity
   patterns that deviate from what is normal for that enclosure or species,
   using historical data as a baseline.
2. **Prioritisation of checks** — ranks open flags by estimated severity and
   confidence, so keepers and veterinarians spend limited time on the cases most
   likely to matter.
3. **Unusual environmental change detection** — specifically watches for
   environmental shifts (temperature, humidity, water quality) that fall outside
   the normal range for an enclosure, even if no single reading looks extreme in
   isolation.
4. **Correlation indication** — surfaces possible relationships between signals
   (for example, reduced feeding activity coinciding with an environmental
   change) as a hypothesis for a human to investigate, never as a confirmed
   diagnosis.

## AI outputs and alert levels

Every AI output includes an explanation of its contributing signals
(explainability), consistent with NFR5 and FR9 in
[../docs/requirements.md](../docs/requirements.md). Alerts are assigned one of
three levels:

| Level | Meaning | Routed to |
|---|---|---|
| **Watch** | A minor deviation detected; likely not urgent but worth noting | Keeper, visible in their regular review queue |
| **Priority check** | A meaningful anomaly or correlation that should be checked soon | Keeper, flagged for same-day attention |
| **Escalate** | A significant or fast-developing anomaly, or one keeper action has not resolved | Veterinarian, in addition to the keeper |

Each alert states: which signals contributed, how the current pattern compares
to the enclosure's baseline, and (where relevant) similar past incidents from
the Analytical Data Store.

## Human decision and accountability

**The keeper or veterinarian always makes the final decision** on any
animal-health or safety matter; the AI component never diagnoses, prescribes
treatment, or takes an autonomous safety action (AI2 in
[../docs/requirements.md](../docs/requirements.md)). Specifically:

- A keeper reviews **Watch** and **Priority check** alerts and decides whether
  to inspect the enclosure, adjust care routines, or dismiss the alert.
- A veterinarian reviews **Escalate** alerts and any keeper-escalated case, and
  makes the clinical decision.
- Every decision (confirm, dismiss, escalate, or "treated as false positive") is
  recorded against the alert for audit and future model evaluation.
- Given the safety risk associated with some exotic species (see assumption A6
  in [../docs/assumptions.md](../docs/assumptions.md)), any alert with a
  potential safety dimension (for example, an enclosure containment or
  hazardous-animal concern) is always escalated to a human with appropriate
  authority; the AI never recommends a direct containment or safety action.

## Explainability

Every alert must be understandable without blind trust in the model:

- Alerts state the specific signals that triggered them (for example, "water
  temperature 3°C below this enclosure's typical range for the past 6 hours,
  combined with reduced feeding activity").
- Alerts reference the enclosure's own historical baseline, not a generic
  species-wide threshold, since normal ranges vary by enclosure and individual
  animal.
- Where a correlation is surfaced, it is explicitly labelled as a possible
  correlation for investigation, not a causal claim.

## False positives and false negatives

Both error types carry real cost and are tracked separately, per
[../docs/success-metrics.md](../docs/success-metrics.md):

- **False positives** (alerts later confirmed as not genuine) waste scarce
  keeper/veterinary time and risk alert fatigue, which can cause real alerts to
  be under-weighted over time.
- **False negatives** (missed incidents later confirmed through other means) are
  the more serious failure mode given the animal-welfare and cost stakes; this
  use case treats false-negative rate as the primary quality signal to monitor,
  consistent with the strictest guardrail defined in
  [../docs/success-metrics.md](../docs/success-metrics.md).
- Both rates are computed against a growing set of keeper/veterinarian-confirmed
  outcomes, which also serves as the model's evaluation dataset over time.

## Manual fallback

- If the AI component is unavailable, or its quality falls below the threshold
  defined in [../docs/success-metrics.md](../docs/success-metrics.md), the
  system falls back to **simple rule-based threshold alerting** (for example,
  fixed environmental ranges per enclosure) rather than disabling alerting
  entirely or continuing to trust a degraded model silently.
- Regardless of AI or rule-based alerting availability, **keepers' normal manual
  observation routine remains the ultimate safety net** and is never replaced by
  this capability (OE2 in [../docs/requirements.md](../docs/requirements.md)).
- During a Wi-Fi outage, buffered telemetry is not lost (see
  [../architecture/data-flow.md](../architecture/data-flow.md)), but is also not
  available for real-time alerting until synchronised; keepers rely on direct
  physical observation in the interim.

## Audit

- Every alert, its explanation, its level, and the eventual human decision are
  logged in a way that cannot be silently altered (SPA4 in
  [../docs/requirements.md](../docs/requirements.md)), building on the Event
  Backbone's ordered event log described in
  [../architecture/event-driven-architecture.md](../architecture/event-driven-architecture.md).
- Access to animal-health data and alerts is restricted to keepers,
  veterinarians and authorised operators (SPA3), as described in
  [../architecture/security-and-privacy.md](../architecture/security-and-privacy.md).
- The audit trail supports both animal-welfare accountability and defensibility
  of the estate's due diligence in case of an incident.

## Success metrics

Defined in full in [../docs/success-metrics.md](../docs/success-metrics.md)
("Animal Health Intelligence" section); summarised here:

- **Business**: reduction in time between an anomaly developing and
  keeper/veterinarian review; reduction in cost or severity of interventions
  attributable to earlier detection; reduction in preventable incidents over
  time (all baselines to be established).
- **AI quality**: precision and recall of anomaly detection against confirmed
  incidents, tracked separately for false positives and false negatives;
  explainability coverage (target: effectively 100% of alerts).
- **Technical**: latency from telemetry/observation capture to alert generation
  (including buffering delay); telemetry data completeness per enclosure;
  pipeline availability and freshness.

## Process for disabling an ineffective model

Because false negatives carry the highest stakes of the three AI use cases (see
[../docs/architecture-characteristics.md](../docs/architecture-characteristics.md)),
this capability has an explicit, mandatory disable path:

1. The system administrator monitors the false-negative rate against
   keeper/veterinarian-confirmed outcomes on an ongoing basis.
2. If the false-negative rate (or another defined quality threshold) is
   breached, the capability automatically falls back to rule-based threshold
   alerting (see "Manual fallback" above) and the AI-generated alerting is
   marked as under review.
3. The system administrator and a veterinarian jointly review the failure
   before the AI-generated alerting is re-enabled.
4. Any model or provider replacement follows the same abstraction-layer
   approach used across all AI capabilities (see [../README.md](../README.md)
   and the related ADR in [../adrs/](../adrs/)), so switching models does not
   require redesigning this use case's data flow.
5. All disable/re-enable actions are recorded in the audit trail described
   above.

## Risks and limitations

- **False negatives** are the most serious risk given animal-welfare and
  reputational stakes; the disable process above is the primary mitigation.
- **Alert fatigue from false positives** could cause keepers to under-weight
  genuine alerts over time; prioritisation levels and explainability are
  intended to reduce this, but require ongoing tuning.
- **Sparse historical/incident data early on** (greenfield system, assumption A1
  in [../docs/assumptions.md](../docs/assumptions.md)) limits the model's
  ability to distinguish normal variation from genuine anomalies until enough
  confirmed outcomes accumulate per enclosure.
- **Correlation is not causation** — surfaced correlations must always be
  labelled as hypotheses for investigation, to avoid a keeper or veterinarian
  mistakenly treating a statistical association as a diagnosis.
- **Delayed data during Wi-Fi outages** means real-time alerting has a gap
  during and immediately after an outage; manual observation is the safety net
  during this window, as noted above.
- **Safety-critical enclosures** (assumption A6 in
  [../docs/assumptions.md](../docs/assumptions.md)) require that any
  containment or hazardous-animal concern always reaches a human with
  appropriate authority; the AI must never be relied upon as the sole safety
  control.
