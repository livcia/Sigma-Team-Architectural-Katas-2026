# Use case: Personalised Visitor Experience

## Business problem

Von Digitalis Estates has no way to personalise a visitor's experience or
maintain a relationship after the visitor leaves. Growth depends almost
entirely on new-visitor acquisition, with no systematic reason for someone to
come back (see [../docs/business-context.md](../docs/business-context.md)).
This use case addresses the third core problem: **few reasons for visitors to
return**, by helping visitors get more out of a single visit and giving them a
concrete reason to plan another one.

This document assumes the baseline operational architecture in
[../architecture/target-state.md](../architecture/target-state.md) already
exists: the Visitor App/Portal, Ticketing Service, Access Control Points, and
the Operational and Analytical Data Stores fed by the Event Backbone. This use
case adds an AI capability on top, without changing that baseline.

## Actors

- **Visitor / visitor group** — the primary user of this capability; receives
  recommendations and can give feedback (see
  [../docs/stakeholders-and-personas.md](../docs/stakeholders-and-personas.md)).
- **Family or visitor group** — a special case where recommendations must
  balance mixed ages and interests within one itinerary.
- **Park operator** — indirectly benefits from reduced congestion when visitors
  are steered toward less busy alternatives; does not directly operate this
  capability.
- **System administrator** — monitors quality (groundedness, relevance) and
  manages content approval and model/provider changes.

## Data inputs

- **Current availability and queue data** for rides and enclosures, sourced from
  the Operational Data Store (the same current-state data used by
  [Visitor Intelligence](visitor-intelligence.md)).
- **An approved knowledge base**: curated, estate-authored content about
  animals, attractions, history and events, maintained and signed off by estate
  staff — not open-ended or user-generated content.
- **Current events and schedule information** (for example, a keeper talk or a
  seasonal event happening today).
- **Visitor preferences**, collected only if the visitor opts in (for example,
  interests, group composition, accessibility needs).
- **Visitor feedback** on previous recommendations, where given.

## Data sources

Current availability and queue data reach the Operational Data Store through the
same event flow described in
[../architecture/target-state.md](../architecture/target-state.md) and
[../architecture/data-flow.md](../architecture/data-flow.md). The approved
knowledge base and event schedule are maintained separately by estate staff as
authored content, not derived from MQTT telemetry; this content is versioned so
the AI component can always identify which version it is grounding
recommendations in. Visitor preferences and feedback are captured directly
through the Visitor App/Portal, only with explicit opt-in (see "Data collected
with visitor consent" below).

## Data flow

1. The visitor opens the Visitor App/Portal and, optionally, provides
   preferences (group composition, interests) and consents to their use for
   personalisation.
2. The Visitor Intelligence-fed Operational Data Store provides current
   occupancy/queue state for rides and enclosures (reused, not duplicated, from
   [visitor-intelligence.md](visitor-intelligence.md)).
3. The **Personalised Visitor Experience AI component** combines current
   availability, the approved knowledge base, current events, and (if provided)
   visitor preferences to generate a recommendation.
4. The recommendation is shown in the Visitor App/Portal, tagged with what it is
   grounded in (for example, "based on today's queue times" or "from the
   estate's animal information") so the visitor can judge its currency.
5. The visitor may accept, ignore, or give feedback on the recommendation; this
   feedback is recorded and used to evaluate and improve future recommendations.
6. After the visit, and only for visitors who opted in, the same capability may
   suggest a reason for a future visit (for example, an upcoming seasonal event),
   delivered through the Visitor App/Portal.

See
[../diagrams/ai-personalised-visitor-experience.md](../diagrams/ai-personalised-visitor-experience.md)
for the corresponding diagram.

## Role of AI

The AI component performs five related functions:

1. **Route recommendation** — suggests an order in which to visit rides and
   enclosures based on current availability and (if provided) visitor
   interests.
2. **Alternative-attraction suggestion** — when a preferred ride or enclosure has
   a long queue, suggests a comparably appealing alternative that is currently
   less busy, reusing congestion data from
   [Visitor Intelligence](visitor-intelligence.md).
3. **Group-fit matching** — for a family or group pass, balances recommendations
   across different ages/interests within the group rather than optimising for
   a single visitor profile.
4. **Grounded information delivery** — answers visitor questions about animals,
   attractions and history using only the approved knowledge base.
5. **Next-visit recommendation** — suggests a reason to return (an upcoming
   event, a seasonal change, an attraction the visitor did not see), for
   opted-in visitors only.

### Why retrieval-augmented generation (RAG) is used, and only where justified

A retrieval-based approach (grounding generated responses in retrieved passages
from the approved knowledge base) is used specifically for **grounded
information delivery**, because visitor questions about animals and attractions
are open-ended in phrasing but must be answered only from vetted, factual
content — RAG lets the system retrieve the relevant approved passage and
compose a natural answer from it, rather than generating an answer from
unconstrained model knowledge.

RAG is **not** used for route recommendation, alternative-attraction suggestion
or group-fit matching, because these are better solved as structured
recommendation problems over current availability and preference data, not
open-ended text generation — using RAG there would add complexity without a
clear benefit. This selective use follows the general principle in
`../AGENT-CONTEXT.md` to prefer a small number of coherent components over
speculative complexity.

## Data collected with visitor consent

- Group composition (for example, ages present) and stated interests, if the
  visitor chooses to provide them.
- Accessibility needs, if the visitor chooses to share them, used only to avoid
  recommending unsuitable routes.
- Feedback on individual recommendations (helpful/not helpful, or a correction).
- Opt-in for post-visit, next-visit suggestions (for example, via the app or an
  email/notification channel the visitor explicitly enables).

No preference or behavioural data is collected covertly; every data point used
for personalisation is something the visitor actively provided or explicitly
enabled (SPA5 and assumption V2 in [../docs/assumptions.md](../docs/assumptions.md)
and [../docs/requirements.md](../docs/requirements.md)).

## Privacy protection

- Preference data is kept separate from ticketing/payment identity data where
  possible, limiting the impact of any single data exposure (see
  [../architecture/security-and-privacy.md](../architecture/security-and-privacy.md)).
- Visitors can view, and withdraw consent for, their stored preferences at any
  time through the Visitor App/Portal.
- Route and queue data used to generate recommendations does not require
  identifying other visitors; it is the same anonymised/aggregated occupancy
  data described in
  [../architecture/security-and-privacy.md](../architecture/security-and-privacy.md).
- Data used to reach a visitor after their visit (for next-visit
  recommendations) is only used for the purpose the visitor opted into, not
  repurposed for unrelated marketing without further consent.

## Data that should not be stored

- Raw free-text visitor questions or conversations should not be retained
  beyond what is needed to generate the immediate response and to support
  aggregate quality monitoring; they should not be stored linked to an
  identifiable visitor for longer than necessary.
- Precise, continuously tracked visitor location/movement is not collected;
  only the zone-level occupancy data already used by
  [Visitor Intelligence](visitor-intelligence.md) is reused, which does not
  require individual tracking.
- Inferred sensitive characteristics (for example, health conditions beyond a
  visitor-declared accessibility need) must not be inferred or stored.
- Any preference or feedback data must not be retained after a visitor
  withdraws consent, beyond the minimum required for legal/financial records
  already covered by ticketing.

## Avoiding hallucination

- Grounded information delivery only answers from the approved knowledge base
  via retrieval; if no relevant approved passage is found, the system responds
  that it does not have approved information on that topic rather than
  generating an unsupported answer (AI5 in
  [../docs/requirements.md](../docs/requirements.md)).
- Responses are checked against the retrieved source passage before being shown,
  so the visitor-facing answer does not introduce claims not present in the
  approved content.
- Route, availability and queue-based recommendations are generated from
  structured operational data, not free-text generation, which further limits
  the surface area for hallucination.

## Distinguishing current from outdated information

- Every piece of approved knowledge-base content carries an effective date and
  a review status; the AI component prefers content flagged as current and
  avoids presenting outdated content as current.
- Recommendations that depend on live operational data (queue times, today's
  event schedule) are explicitly labelled with the time they reflect (for
  example, "as of a few minutes ago"), so a visitor can judge freshness.
- Static informational content (for example, general facts about an animal
  species) is distinguished in the app from time-sensitive content (for
  example, "open now" or "today's schedule"), so a visitor does not mistake one
  for the other.

## Rule-based fallback

When the AI component or its data sources are degraded or unavailable, the
Visitor App/Portal falls back to (FR15, OE3 in
[../docs/requirements.md](../docs/requirements.md)):

- **Static, curated content** from the approved knowledge base, browsable
  without personalisation.
- **Simple rule-based suggestions**, for example a "popular today" list derived
  directly from [Visitor Intelligence](visitor-intelligence.md) data rather than
  a personalised AI recommendation.
- A clear indication in the app that recommendations are currently running in a
  simplified mode, so the visitor's expectations are set correctly.

Visitors who never used the app, or who are in a full fallback state, still
have a complete, non-personalised experience and are not excluded from any part
of the estate (assumption V3 in [../docs/assumptions.md](../docs/assumptions.md)).

## Reporting a bad recommendation

- Every recommendation shown in the Visitor App/Portal has a visible, low-effort
  way to flag it as unhelpful or incorrect (FR14 in
  [../docs/requirements.md](../docs/requirements.md)).
- Flagged recommendations are recorded with enough context (what was
  recommended, what data grounded it) to support review and model evaluation,
  without requiring the visitor to explain in detail.
- A pattern of flags against the same type of recommendation or knowledge-base
  entry triggers a review by estate staff responsible for the approved content
  or the AI capability's quality.

## Success metrics

Defined in full in [../docs/success-metrics.md](../docs/success-metrics.md)
("Personalised Visitor Experience" section); summarised here:

- **Business**: increase in repeat-visit rate attributable to recommendations;
  recommendation acceptance/click-through rate as a leading indicator;
  positive/negative feedback ratio.
- **AI quality**: groundedness rate (responses verifiably grounded in approved
  content); recommendation relevance via visitor feedback; rate of
  visitor-reported incorrect information.
- **Technical**: latency from request to recommendation; recommendation-service
  availability; freshness of the availability/queue data used to ground
  recommendations.

## Risks and limitations

- **Over-personalisation without enough opted-in data** could make
  recommendations feel generic for visitors who choose not to share
  preferences; the rule-based fallback ensures they still get a useful,
  non-personalised experience.
- **Hallucinated or outdated information**, if the groundedness and
  freshness controls above are not maintained, would directly damage visitor
  trust — the opposite of this use case's goal; this is why groundedness rate
  is tracked as a primary AI-quality metric.
- **Consent fatigue or mistrust** if preference collection is not clearly
  explained could reduce opt-in rates, limiting how much personalisation is
  possible; framing and transparency are a product concern to manage
  alongside this architecture.
- **Congestion feedback loops**: if many visitors are steered toward the same
  "less busy alternative," that alternative could itself become congested;
  recommendations should be informed by real-time occupancy so this is
  self-correcting, but it remains a limitation of any popularity-based steering
  approach.
- **Approved-content staleness** is a content-operations risk, not just a
  technical one — the effective-date/review-status mechanism only works if
  estate staff keep the knowledge base current.
