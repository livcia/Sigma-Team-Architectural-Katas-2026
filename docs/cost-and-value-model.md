# Cost and value model

This document describes the cost and value structure of the solution at a
category level, without naming specific vendor prices, consistent with
`../AGENT-CONTEXT.md`. It complements
[business-context.md](business-context.md) (value streams),
[requirements.md](requirements.md) and
[success-metrics.md](success-metrics.md) (how value is measured), and the
architecture in [../architecture/target-state.md](../architecture/target-state.md).

All figures here are **relative comparisons and explicit assumptions**, not
priced estimates. Where a number is used, it is labelled as an assumption or
design target, consistent with [assumptions.md](assumptions.md).

## Sources of business value

Reused from [business-context.md](business-context.md), value comes from four
linked streams:

1. **Admissions revenue** — more visitors, better-timed capacity, fewer
   congestion-driven complaints.
2. **Operational efficiency** — staff and maintenance effort matched to actual
   demand instead of guesswork ([Visitor
   Intelligence](../use-cases/visitor-intelligence.md)).
3. **Animal welfare and care cost** — earlier detection reduces treatment cost
   and animal loss ([Animal Health
   Intelligence](../use-cases/animal-health-intelligence.md)).
4. **Visitor lifetime value** — repeat visits and referrals reduce dependence
   on constant new-visitor acquisition ([Personalised Visitor
   Experience](../use-cases/personalised-visitor-experience.md)).

## Potential operational savings

| Source | Mechanism | Relative scale (assumption) |
|---|---|---|
| Reduced overstaffing | Staffing matched to forecast demand rather than fixed rosters | Moderate — realised gradually as forecast accuracy improves |
| Reduced veterinary/treatment cost | Earlier detection avoids escalation to costly interventions | Potentially significant per avoided incident, though incident frequency is currently unknown (assumption D4 in [assumptions.md](assumptions.md)) |
| Reduced manual monitoring effort | Keepers/operators spend time on prioritised cases instead of blanket manual checks | Moderate, grows as historical data accumulates |
| Reduced marketing spend for repeat visits | Return visits driven by personalisation rather than paid acquisition | Small initially, compounds over multiple seasons |

These savings are design goals to validate against real data once the
operational backbone exists, not committed figures.

## Potential revenue growth

- **Congestion reduction** keeps visitors engaged longer and reduces
  complaint-driven dissatisfaction, supporting both same-day spend and repeat
  visits.
- **Personalisation and next-visit recommendations** directly target the
  return-visit rate identified as a core problem in
  [business-context.md](business-context.md).
- **Better-informed investment decisions** (from popularity rankings in
  [Visitor Intelligence](../use-cases/visitor-intelligence.md)) aim to direct
  capital toward attractions/enclosures with the best demonstrated demand,
  rather than guesswork.
- Growth from 5,000 to 15,000 daily visitors (brief target) is the primary
  revenue driver; this solution's role is to make that growth sustainable
  without a matching cost increase, not to guarantee the growth itself.

## Cost categories

Costs are grouped by category, with relative comparisons rather than
absolute figures.

### MQTT device costs

- One-time/periodic hardware cost per device, scaled by the number of
  rides/enclosures instrumented, not by visitor count (assumption M1 in
  [assumptions.md](assumptions.md): devices at zone/enclosure level, not per
  animal).
- Ongoing costs: device maintenance, battery/power where applicable, and
  eventual replacement.
- This cost category is **largely fixed relative to visitor growth** — 40
  rides and 55 enclosures do not multiply with attendance, so this is a
  favourable cost driver for scaling from 5,000 to 15,000 visitors/day.

### Edge gateway costs

- One gateway per zone (see
  [../architecture/deployment-view.md](../architecture/deployment-view.md)):
  hardware, local storage for buffering, and networking equipment.
- Ongoing costs: maintenance, power, and eventual hardware refresh.
- Like MQTT devices, this scales with the **number of zones**, not with
  visitor count — another cost category that stays relatively flat as
  attendance grows, provided the estate does not add many new zones.

### Data storage costs

- **Operational Data Store**: smaller, current-state data; cost scales
  moderately with visitor count (more concurrent tickets, entries, alerts).
- **Analytical Data Store**: larger, historical data; cost grows over time
  regardless of daily visitor count, as more history accumulates, but at a
  predictable, roughly linear rate.
- Retaining only anonymised/aggregated visitor-flow data where possible (per
  [../architecture/security-and-privacy.md](../architecture/security-and-privacy.md))
  keeps this category smaller than storing fully identifiable data for every
  visitor interaction.

### AI inference costs

- Scales with **inference volume**: forecasts and alerts (periodic, not
  per-visitor, for [Visitor Intelligence](../use-cases/visitor-intelligence.md)
  and [Animal Health
  Intelligence](../use-cases/animal-health-intelligence.md)) versus
  per-interaction requests (for [Personalised Visitor
  Experience](../use-cases/personalised-visitor-experience.md), which does
  scale with visitor count and engagement).
- This is the cost category **most sensitive to visitor growth**, since
  personalisation requests naturally increase with more visitors using the
  app.
- Mitigated by the AI abstraction layer (allows swapping to a lower-cost
  model/provider) and by limiting RAG/generative use to only the
  information-delivery function, per
  [../adrs/ADR-005-rag-for-approved-visitor-information.md](../adrs/ADR-005-rag-for-approved-visitor-information.md).

### Maintenance and monitoring costs

- Ongoing effort (staff time and/or tooling) for the monitoring described in
  [validation-and-fitness-functions.md](validation-and-fitness-functions.md):
  golden dataset maintenance, drift testing, cost/latency/quality dashboards.
- This is largely a **fixed operational cost** (a small technical team) that
  grows sub-linearly with visitor count, since monitoring effort scales with
  the number of AI capabilities, not directly with traffic volume.

### Cost of false alerts

- Keeper/veterinarian or operator time spent reviewing and dismissing false
  positives (see [risks-and-mitigations.md](risks-and-mitigations.md), risk 2).
- This is a **quality-dependent cost**: it shrinks as alerting is tuned over
  time, but is highest early on with limited historical data (assumption A1 in
  [assumptions.md](assumptions.md)).
- Treated as a cost worth incurring deliberately in the early period, in
  exchange for the safety margin of over-alerting rather than under-alerting
  on animal health, consistent with
  [../adrs/ADR-003-animal-health-anomaly-detection.md](../adrs/ADR-003-animal-health-anomaly-detection.md).

### Cost of manual fallback

- When any AI capability falls back (rule-based alerting, manual staffing
  planning, static content), the estate incurs the same operational cost it
  would without AI at all for that period — effectively, a **temporary return
  to the current-state cost baseline** described in
  [../architecture/current-state.md](../architecture/current-state.md).
- This cost is bounded and self-limiting: fallback periods are expected to be
  occasional (triggered by a quality threshold breach or connectivity loss),
  not the normal operating mode.

## MVP model

An MVP should prove the **operational backbone plus one AI capability** before
building all three, given limited historical data at launch (assumption D4 in
[assumptions.md](assumptions.md)):

- Baseline architecture ([ticketing, access control, MQTT devices, edge
  gateway, event backbone](../architecture/target-state.md)) must exist
  regardless of which AI capability is added first — this is the larger,
  necessary fixed cost.
- Among the three AI capabilities, [Animal Health
  Intelligence](../use-cases/animal-health-intelligence.md) has the highest
  business urgency (direct cost and welfare impact) but requires the most
  cautious rollout (rule-based fallback, mandatory human confirmation) given
  the stakes described in
  [../adrs/ADR-003-animal-health-anomaly-detection.md](../adrs/ADR-003-animal-health-anomaly-detection.md).
- [Visitor Intelligence](../use-cases/visitor-intelligence.md) is a reasonable
  first AI capability for an MVP: lower risk if imperfect (falls back to
  manual planning), and it generates the demand/occupancy data that the other
  two use cases also depend on.
- [Personalised Visitor
  Experience](../use-cases/personalised-visitor-experience.md) benefits most
  from having Visitor Intelligence data already flowing (for congestion-aware
  route suggestions) and can be added once the visitor app and knowledge base
  exist.

This sequencing is a suggested approach, not a fixed requirement; the estate
may prioritise differently based on its own risk appetite.

## Scaling model: 5,000 to 15,000 daily visitors

| Cost category | How it scales |
|---|---|
| MQTT devices | Flat (scales with zones/enclosures, not visitors) |
| Edge gateways | Flat (scales with zones, not visitors) |
| Operational data storage | Moderate growth (more concurrent tickets/entries) |
| Analytical data storage | Steady growth over time (more history), largely independent of daily visitor count |
| AI inference (forecasting/anomaly detection) | Low growth (periodic, not per-visitor) |
| AI inference (personalisation) | Highest growth (scales with visitor app usage) |
| Maintenance/monitoring | Sub-linear growth (small technical team, more data to review) |
| False-alert handling | Should shrink over time as models mature, independent of visitor count |
| Manual fallback | Should stay flat or shrink as reliability improves |

The **overall design goal** is that infrastructure and AI costs grow more
slowly than visitor-driven revenue, so unit economics improve as attendance
grows toward 15,000/day — this is the target this cost model is built to
support, not a guaranteed outcome.

## Cost guardrails

- Each AI capability has a defined cost-monitoring guardrail (see
  [validation-and-fitness-functions.md](validation-and-fitness-functions.md),
  "Cost monitoring"), triggering review if inference cost per visitor or per
  alert exceeds a set threshold.
- Guardrail breaches trigger a defined response order: first, reduce
  inference frequency/scope (for example, less frequent forecasts, shorter
  personalisation context); second, consider a lower-cost model/provider via
  the AI abstraction layer
  ([../adrs/ADR-001-ai-model-abstraction.md](../adrs/ADR-001-ai-model-abstraction.md));
  only as a last resort, temporarily reduce the scope of a capability.
- Data storage guardrails (for example, retention periods for raw
  visitor-interaction logs, per
  [../use-cases/personalised-visitor-experience.md](../use-cases/personalised-visitor-experience.md))
  limit unbounded growth of the Analytical Data Store.

## Limiting AI costs

- **Prefer structured recommendation logic over generative AI** wherever a
  structured approach solves the problem equally well (route/alternative/
  group-fit recommendations use structured data, not RAG — see
  [../adrs/ADR-005-rag-for-approved-visitor-information.md](../adrs/ADR-005-rag-for-approved-visitor-information.md)),
  since generative inference is typically costlier per request.
- **Batch rather than per-request inference** for forecasting and anomaly
  detection, since these do not need to run per-visitor-interaction.
- **Cache stable recommendations** (for example, general animal/attraction
  information) rather than regenerating them for every identical question.
- **Right-size model capability to the task** behind the AI abstraction layer:
  not every function needs the most capable (and costly) available model.

## Local vs. cloud execution

| Decision | Where it runs | Why |
|---|---|---|
| MQTT ingestion, buffering | Local (Zone Edge Gateway) | Must work without cloud connectivity (see [../adrs/ADR-002-edge-buffering-and-offline-mode.md](../adrs/ADR-002-edge-buffering-and-offline-mode.md)); low compute need |
| Ticketing, entry validation | Cloud, with local tolerance for brief disconnection | Needs centralised consistency across zones; not compute-heavy |
| Forecasting, anomaly detection (AI) | Cloud | Needs the full historical/analytical dataset, which is centralised |
| Rule-based fallback alerting | Local or cloud, whichever remains reachable | Must work even when the cloud AI component is unavailable, per each use case's fallback design |
| Personalisation/RAG inference | Cloud | Needs the centralised, versioned approved knowledge base and current operational data |
| Monitoring/evaluation (golden datasets, drift testing) | Cloud | Needs the full dataset and is not time-critical for zone operations |

The general principle: anything required to **keep a single zone operating
during a Wi-Fi outage** runs locally; anything requiring a **cross-zone or
historical view** runs in the cloud. This mirrors the operational/analytical
separation already established in
[../architecture/target-state.md](../architecture/target-state.md).

## Assumptions used in this document

- No specific vendor pricing is assumed or estimated; all comparisons are
  relative (for example, "flat," "grows with visitor count," "grows with
  zones").
- The relative cost-scaling claims (for example, "AI inference for
  personalisation is the most visitor-sensitive cost") are architectural
  reasoning, not measured figures, and should be validated once real usage
  data exists.
- The suggested MVP sequencing (Visitor Intelligence first) is a
  recommendation based on relative risk and data dependency, not a
  requirement from the brief.
- Cost guardrail thresholds are left undefined numerically, consistent with
  avoiding unsupported precision (per `../AGENT-CONTEXT.md`).
