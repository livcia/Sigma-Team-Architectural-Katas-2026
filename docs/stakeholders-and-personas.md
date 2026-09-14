# Stakeholders and personas

This document describes the people whose needs and constraints shape the
architecture. It complements [business-context.md](business-context.md) and
[requirements.md](requirements.md) and is referenced by the use cases in
[../use-cases/](../use-cases/).

## Visitor

**Who they are:** A single paying guest visiting the estate for the rides, the
animal collection, or both.

**Goals:** Enjoy the visit, avoid long queues, feel confident the animals are well
cared for, and get help finding things worth seeing.

**Pain points today:** No way to know which rides or enclosures are busy before
arriving there; no personalised suggestions; no reason to plan a return visit
beyond general goodwill.

**Interaction with the solution:** Uses the visitor app/portal (see
[assumptions.md](assumptions.md), V1) to buy tickets, view recommendations from
[Personalised Visitor Experience](../use-cases/personalised-visitor-experience.md),
and optionally give feedback on suggestions.

## Family or visitor group

**Who they are:** A family or group of friends visiting together, often with mixed
ages and interests (for example, young children wanting rides, adults wanting the
animal collection).

**Goals:** Find an itinerary that works for the whole group, minimise time spent
queueing or negotiating what to do next, and have a smooth day that justifies the
cost of a family pass.

**Pain points today:** No way to plan a day that balances different group members'
interests; no visibility into which attractions suit which age groups or how busy
they currently are.

**Interaction with the solution:** Purchases a family/group pass (FR1), receives
group-oriented route and attraction recommendations from
[Personalised Visitor Experience](../use-cases/personalised-visitor-experience.md)
that account for mixed interests, and benefits indirectly from
[Visitor Intelligence](../use-cases/visitor-intelligence.md) through reduced
congestion.

## Park operator

**Who they are:** Estate staff responsible for day-to-day running of the
amusement-park side of the business — staffing rides, managing queues, planning
maintenance windows.

**Goals:** Deploy staff and resources where they are actually needed, reduce
congestion and visitor complaints, and use ride/zone data to justify investment
decisions to the Countess or estate management.

**Pain points today:** No reliable way to know which zones are busy or why; staffing
and maintenance decisions rely on informal observation and past experience.

**Interaction with the solution:** Primary consumer of
[Visitor Intelligence](../use-cases/visitor-intelligence.md) forecasts and
recommendations; retains final authority over staffing, pricing and operational
changes (AI2 in [requirements.md](requirements.md) — AI recommends, does not act
autonomously).

## Animal keeper

**Who they are:** Staff responsible for the daily care, feeding and welfare
monitoring of animals in one or more enclosures.

**Goals:** Catch health or environmental problems early, prioritise limited time
across many enclosures, and avoid both missing real issues and being overwhelmed by
false alarms.

**Pain points today:** With 200+ animals across 55 enclosures, keepers cannot
continuously monitor every enclosure manually; problems are often discovered only
when they become visibly serious.

**Interaction with the solution:** Primary recipient of alerts from
[Animal Health Intelligence](../use-cases/animal-health-intelligence.md); logs
observations that feed back into the system (D3 in
[assumptions.md](assumptions.md)); confirms, dismisses or escalates AI-generated
alerts (FR10).

## Veterinarian

**Who they are:** A qualified veterinary professional (on staff or on call)
responsible for diagnosing and treating animal-health issues, especially for
exotic species requiring specialist knowledge.

**Goals:** Be alerted to genuine health risks with enough explanation to
prioritise effectively, and avoid being paged for non-issues (alert fatigue).

**Pain points today:** No systematic prioritisation of which animals or enclosures
need attention first; reliance on keeper escalation alone.

**Interaction with the solution:** Receives escalated, explainable alerts from
[Animal Health Intelligence](../use-cases/animal-health-intelligence.md) (FR9);
makes the final veterinary decision — the AI never diagnoses or prescribes
treatment autonomously (AI2).

## Countess / estate management

**Who they are:** The Countess and any senior estate management supporting her,
accountable for the estate's overall profitability and reputation.

**Goals:** Grow visitor numbers sustainably to the three-year target, control
operating and animal-care costs, and protect the estate's reputation (a single
animal-welfare or safety failure could be very damaging).

**Pain points today:** Decisions about where to invest (new rides, more keepers,
marketing) are made without data connecting spend to outcomes.

**Interaction with the solution:** Consumer of aggregated business outcomes (see
[README.md](../README.md#expected-business-outcomes) and
[success-metrics.md](success-metrics.md) once available); ultimate owner of
risk-acceptance decisions such as adopting a new AI capability or accepting a
trade-off documented in an ADR.

## System administrator

**Who they are:** Technical staff (in-house or contracted) responsible for keeping
the estate's ticketing, MQTT devices, edge gateways, cloud services and AI
components running.

**Goals:** Keep the system available despite patchy Wi-Fi, detect and respond to
degraded AI behaviour or outages quickly, and manage costs (device fleet, cloud,
AI inference) as the estate scales from 5,000 to 15,000 daily visitors.

**Pain points today:** Not applicable yet (greenfield system, A1 in
[assumptions.md](assumptions.md)), but anticipated pain points include device
fleet management across 40 rides and 55 enclosures, and diagnosing failures in a
distributed, intermittently-connected environment.

**Interaction with the solution:** Owns monitoring, alerting, rollback and
model/provider replacement processes (NFR7, AI1, AI6 in
[requirements.md](requirements.md)); accountable for the audit trail required by
SPA4.

## Summary: decision rights

A recurring theme across personas is that **AI recommends and humans decide** for
anything safety-critical, veterinary, financial or irreversible:

| Persona | Decides | AI supports with |
|---|---|---|
| Park operator | Staffing, pricing, operational changes | Demand forecasts, congestion alerts |
| Animal keeper / Veterinarian | Animal-health actions, treatment | Anomaly detection, prioritised alerts, explanations |
| Visitor / Family | What to do, when to return | Route and attraction recommendations |
| Countess / management | Investment, risk acceptance, AI adoption | Aggregated business outcome metrics |
| System administrator | Model/provider changes, rollback, incident response | Monitoring, quality and cost metrics |
