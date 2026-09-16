# Business context: Von Digitalis Estates

This document expands the business situation summarised in [README.md](../README.md)
without changing its meaning. It exists to give later documents (requirements,
architecture, use cases) a shared, stable description of the business to design
against.

## Situation

The 72nd Countess Von Digitalis has inherited a large estate that can no longer rely
on its previous business (explosive garden gnomes) for income. To make the estate
profitable, the Countess is converting it into a paid visitor destination built
around two very different assets:

- **40 restored 18th-century amusement rides**, which are physical, low-tech,
  high-value historical assets that need careful, low-risk operation; and
- **an exotic animal collection of 200+ animals across 55 displays/enclosures**,
  which requires ongoing, specialist welfare care and carries reputational and
  financial risk if health problems are missed.

The estate currently attracts approximately **5,000 visitors per day** and must grow
to **15,000 visitors per day within three years** (the brief's stated alternative —
selling the carnivorous plant collection — is treated as an undesirable fallback,
not a planning target).

Today the estate has minimal digital visibility into its own operations: ticket
sales happen, but there is no systematic record of which rides or enclosures draw
crowds, no early-warning system for animal health, and no mechanism for
understanding or influencing whether a visitor returns. Decisions about staffing,
investment and animal care are made without reliable data.

## Countess's goals

1. **Make the estate profitable** on an ongoing basis, not just through a one-time
   opening.
2. **Grow attendance three-fold within three years** without a proportional increase
   in operating cost or animal welfare risk.
3. **Protect the estate's unique assets** — the historic rides and the exotic
   animals — from costly, avoidable damage or health incidents.
4. **Build a destination people want to return to**, rather than a one-visit
   novelty attraction.
5. **Adopt technology, including AI, responsibly** — the Countess wants innovation,
   but the estate cannot afford AI mistakes that damage safety, animal welfare, or
   trust.

## Business problems

These map directly to the three core problems in [README.md](../README.md) and are
restated here with their business consequence, not their technical cause:

1. **No visibility into popularity and demand.** Without knowing which rides, zones
   or times are busy, the estate cannot allocate staff, plan maintenance windows, or
   justify investment in new attractions. Money and effort are spent on guesswork.
2. **High cost of animal care.** Veterinary and keeper time is expensive and finite.
   When health or environmental problems in an enclosure are discovered late, the
   cost of treatment rises and the risk of losing an animal (financial and
   reputational) increases. Early, well-prioritised attention is far cheaper than
   crisis response.
3. **Few reasons for visitors to return.** Ticket revenue today depends almost
   entirely on new-visitor acquisition. There is no relationship with a visitor once
   they leave, no personalisation, and no visible reason to prefer this estate over
   a one-off outing elsewhere.

## Value streams

The estate's revenue and cost structure can be grouped into four value streams that
the solution must support or protect:

| Value stream | Description | Primary risk if ignored |
|---|---|---|
| **Admissions revenue** | Individual and family tickets/passes for park access | Underpriced or mistimed capacity; lost revenue from congestion |
| **Operational efficiency** | Staffing, maintenance and investment matched to actual demand | Overstaffing low-demand areas; understaffing popular ones |
| **Animal welfare and care cost** | Keeper/veterinary time, treatment cost, animal survival | Late detection turns a cheap fix into an expensive, or fatal, one |
| **Visitor lifetime value** | Repeat visits, word-of-mouth, brand loyalty | Estate stays dependent on constant new-visitor acquisition |

## Relationship between experience, revenue and cost

These value streams are not independent; the solution is designed around their
interaction:

- A **better visitor experience** (shorter queues, relevant recommendations,
  confidence that animals are well cared for) increases the likelihood of **repeat
  visits and referrals**, directly growing admissions revenue without additional
  marketing spend.
- **Better demand visibility** allows staff, maintenance and capacity to be
  allocated where they are actually needed, which **reduces wasted operating cost**
  while simultaneously improving the experience (less crowding, shorter waits).
- **Earlier detection of animal-health issues** reduces treatment cost and animal
  loss, which protects both **operating cost** and the **reputation** that
  underpins visitor trust and return-visit intent.
- Conversely, cutting corners in any one area — congestion, animal welfare, or
  personalisation — erodes trust and directly undermines the three-year growth
  target, because growth from 5,000 to 15,000 visitors per day amplifies the
  consequences of any operational or welfare failure.

This is why the solution treats operational visibility, animal-health
intelligence and personalised experience as three tightly linked capabilities
rather than three unrelated features — see the use cases in
[../use-cases/](../use-cases/) once available, and the overall narrative in
[README.md](../README.md).

## Assumptions used in this document

Numeric targets (5,000 → 15,000 visitors, three-year horizon) are taken directly
from the competition brief ([architectural-katas-2026.md](../architectural-katas-2026.md))
and are not further estimates. Statements about current operational visibility,
staffing practices and animal-care processes are reasonable assumptions consistent
with the brief, since the brief does not describe them in detail. See
[assumptions.md](assumptions.md) for the full, explicit list of assumptions used
across this documentation set.
