# Use case: Visitor Intelligence

## Business problem

Von Digitalis Estates has no reliable visibility into which rides, zones or times
draw crowds, cause congestion, or go underused. Staffing, maintenance and
investment decisions are made on instinct rather than evidence, which is
unsustainable as the estate grows from approximately 5,000 to 15,000 daily
visitors (see [../docs/business-context.md](../docs/business-context.md)). This
use case turns existing and new estate data into demand and popularity
intelligence that a park operator can act on.

This document assumes the baseline operational architecture described in
[../architecture/target-state.md](../architecture/target-state.md) already
exists: ticketing, access control, MQTT telemetry, the Zone Edge Gateway, the
Event Backbone, and the Operational and Analytical Data Stores. This use case
adds an AI capability on top, without changing that baseline.

## Actors

- **Visitor / visitor group** — generates the underlying demand signal (entries,
  ride usage, dwell time) but does not interact with this capability directly.
- **Park operator** — the primary consumer of forecasts, congestion alerts and
  staffing recommendations; retains authority over any operational change (see
  [../docs/stakeholders-and-personas.md](../docs/stakeholders-and-personas.md)).
- **System administrator** — monitors the AI capability's health and manages
  model/provider changes (see
  [../docs/stakeholders-and-personas.md](../docs/stakeholders-and-personas.md)).

## Data inputs

- Ticket sales and pass types (`ticket.sold` events).
- Entry/exit events per zone (`visitor.entered` / `visitor.exited` events).
- Ride usage and status telemetry (`ride.telemetry` events).
- Queue-related signals where available (for example, dwell time near a ride or
  enclosure, derived from entry/exit and telemetry timing).
- Weather data and scheduled-event information (external context, not produced
  by the estate itself).
- Historical attendance and occupancy patterns, once accumulated in the
  Analytical Data Store.

## Data sources

All estate-originated inputs flow through the components defined in
[../architecture/target-state.md](../architecture/target-state.md) and
[../architecture/event-driven-architecture.md](../architecture/event-driven-architecture.md):
the Ticketing Service, Access Control Points, and MQTT Devices, all reaching the
Event Backbone via the Zone Edge Gateway and Cloud Sync Service. Weather and
event data are external sources ingested directly into the Analytical Data
Store; no assumption is made about a specific external provider (per
`../AGENT-CONTEXT.md`).

## Data flow

1. Ticketing, entry/exit and ride telemetry events reach the Event Backbone as
   described in [../architecture/data-flow.md](../architecture/data-flow.md),
   including the buffered path used during a Wi-Fi outage.
2. These events update both the Operational Data Store (current state: today's
   occupancy, active queues) and the Analytical Data Store (historical trends).
3. External weather and event data is ingested into the Analytical Data Store on
   a scheduled or event-driven basis.
4. The **Visitor Intelligence AI component** reads current-state data from the
   Operational Data Store and historical/contextual data from the Analytical
   Data Store to produce forecasts, congestion alerts, popularity analysis and
   staffing recommendations.
5. Outputs are written back to the Operational Data Store and surfaced through
   the Operator Console.
6. Operator actions and feedback on recommendations flow back into the
   Operational Data Store, closing the loop for future model evaluation.

See [../diagrams/ai-visitor-intelligence.md](../diagrams/ai-visitor-intelligence.md)
for the corresponding diagram.

## Role of AI

The AI component performs four related functions, all read-only with respect to
estate operations:

1. **Demand forecasting** — predicts expected attendance and zone occupancy for
   upcoming periods (for example, later today or the next few days), using
   historical patterns plus weather and scheduled-event context.
2. **Congestion and queue detection** — identifies zones currently experiencing,
   or likely to soon experience, unusually high occupancy or queue build-up,
   using near-real-time occupancy and telemetry data.
3. **Zone/attraction popularity analysis** — aggregates historical usage to rank
   rides, enclosures and zones by relative popularity over time, supporting
   investment and maintenance-planning discussions.
4. **Staffing recommendation** — suggests where additional staff or resources
   would likely reduce congestion or improve service, based on the forecast and
   current occupancy, expressed as a recommendation with a rationale.

The AI component sits behind an abstraction layer consistent with the model/
provider-swap approach described in [../README.md](../README.md), so the
specific forecasting or anomaly-detection technique used is not fixed by this
document.

## AI outputs

- A **demand forecast** per zone/time window, with an associated confidence
  indicator.
- A **congestion alert** for a specific zone, including the contributing
  signals (for example, current occupancy vs. typical level, queue growth
  rate).
- A **popularity ranking** of zones/attractions over a selected historical
  period.
- A **staffing recommendation**, expressed as a suggested action (for example,
  "consider adding staff to Zone C between 13:00–15:00") together with the
  forecast or alert that motivated it.

All outputs are recommendations or informational analysis, never direct changes
to schedules, pricing, staffing systems or physical ride operation.

## Operator decisions

The park operator, not the AI component, decides:

- Whether and how to change staffing levels or shift patterns.
- Whether to open/close overflow capacity, alter queue management, or
  communicate wait times to visitors.
- Whether a popularity ranking justifies an investment or maintenance-priority
  decision.
- Whether to dismiss a forecast or alert as not credible (for example, if it
  contradicts the operator's on-the-ground knowledge), with that decision
  recorded for later review.

This directly implements AI2 in [../docs/requirements.md](../docs/requirements.md):
AI must not autonomously execute safety-critical, financial or irreversible
actions — staffing and operational changes remain human-approved.

## Behaviour with missing data

- If entry/exit or telemetry data for a zone is incomplete or delayed (for
  example, still buffered at a Zone Edge Gateway during a Wi-Fi outage — see
  [../architecture/data-flow.md](../architecture/data-flow.md)), the AI
  component marks any forecast or alert for that zone as **reduced confidence**
  rather than silently treating stale data as current.
- If weather or event context is unavailable, forecasts fall back to
  historical-pattern-only predictions, clearly labelled as such.
- If historical data for a zone is sparse (for example, a newly opened
  attraction), the AI component indicates low confidence rather than producing
  an unqualified forecast, consistent with the limited-historical-data
  assumption (D4 in [../docs/assumptions.md](../docs/assumptions.md)).

## Behaviour when AI is unavailable

- If the Visitor Intelligence AI component itself is unavailable, the Operator
  Console falls back to displaying **raw current occupancy and historical
  patterns** directly from the Operational and Analytical Data Stores, without
  a forecast or recommendation layer.
- Core ticketing, entry and telemetry capture are unaffected, since they do not
  depend on this AI component (OE2 in
  [../docs/requirements.md](../docs/requirements.md)).
- Operators fall back to their existing manual planning process for staffing
  decisions until the capability is restored.

## Success metrics

Defined in full in [../docs/success-metrics.md](../docs/success-metrics.md)
("Visitor Intelligence" section); summarised here:

- **Business**: reduction in congestion complaints; improved staffing-to-demand
  alignment; contribution to sustained growth toward the 15,000-visitor target
  without a proportional rise in operational incidents.
- **AI quality**: forecast accuracy against actuals; precision/recall of
  congestion alerts against operator-confirmed events.
- **Technical**: latency from telemetry capture to forecast/alert; data
  completeness rate; service availability.

## Risks and limitations

- **False congestion alerts** could cause operators to over-allocate staff
  unnecessarily; a maximum acceptable false-alert rate should be defined once
  baseline data exists (see
  [../docs/success-metrics.md](../docs/success-metrics.md)).
- **Missed congestion events** (false negatives) undermine trust in the
  capability and directly harm the visitor experience it is meant to protect.
- **Sparse historical data early on** (greenfield system, assumption A1 in
  [../docs/assumptions.md](../docs/assumptions.md)) limits forecast quality
  until sufficient data accumulates; this is an expected, temporary limitation,
  not a design flaw.
- **Weather/event data dependency** introduces reliance on an external data
  source outside the estate's control; the fallback to historical-pattern-only
  forecasting mitigates but does not eliminate this.
- **Operator over-reliance** on AI recommendations is a governance risk if not
  actively managed; the requirement that AI only recommends, and that operators
  can dismiss and record disagreement, is the primary mitigation (AI2 in
  [../docs/requirements.md](../docs/requirements.md)).
