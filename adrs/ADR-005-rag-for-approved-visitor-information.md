Date: 2026-09-14

# ADR-005: RAG for approved visitor information

## Status
Proposed

## Context

[Personalised Visitor Experience](../use-cases/personalised-visitor-experience.md)
must answer open-ended visitor questions about animals, attractions, history
and events. Visitors phrase questions in many different ways, but the answers
must come only from estate-approved, factual content — the estate cannot risk
a visitor being told something inaccurate about a hazardous exotic animal, a
historic ride, or a scheduled event (AI5 in
[../docs/requirements.md](../docs/requirements.md)).

Route recommendations, alternative-attraction suggestions and group-fit
matching (also part of the same use case) are a different kind of problem:
they depend on structured, current operational data (queue times, occupancy),
not open-ended factual content.

Alternatives considered:

- **Unconstrained generative responses**, where a model answers visitor
  questions from its general training knowledge. Rejected: this cannot
  guarantee the answer reflects the estate's actual animals, attractions or
  current events, and risks hallucinated or outdated claims — unacceptable
  given the trust this use case is meant to build.
- **Fully static, keyword-searchable content** (no AI), where visitors browse
  or search pre-written articles directly. This would be safe from
  hallucination but would not handle the open-ended, conversational phrasing
  visitors actually use, reducing usefulness.
- **Retrieval-augmented generation (RAG)** restricted to the approved
  knowledge base: the system retrieves the most relevant approved passage(s)
  for a visitor's question and composes a response grounded in that retrieved
  content, refusing to answer if no relevant approved passage exists.
- Applying RAG (or any generative technique) to **route/alternative/group-fit
  recommendations** as well. Rejected: these are structured recommendation
  problems over current availability data, and forcing them through a
  retrieval-and-generation pipeline would add complexity without a clear
  benefit, contrary to the principle in `../AGENT-CONTEXT.md` of preferring a
  small number of coherent components over speculative complexity.

## Decision

RAG is used **only for grounded information delivery** within Personalised
Visitor Experience — answering visitor questions about animals, attractions,
history and events from the approved, versioned knowledge base. Route
recommendation, alternative-attraction suggestion and group-fit matching are
implemented as structured recommendations over current operational data,
without RAG or open-ended generation.

When RAG is used, the response must be traceable to a specific retrieved
passage from the approved knowledge base; if no sufficiently relevant approved
passage is found, the system states that it does not have approved information
on that topic rather than generating an unsupported answer (see
[personalised-visitor-experience.md](../use-cases/personalised-visitor-experience.md),
"Avoiding hallucination"). No specific RAG implementation, embedding technique
or model provider is chosen here; this sits behind the AI abstraction layer in
ADR-001.

## Consequences

**Positive:**
- Visitors get natural, conversational answers to open-ended questions while
  the estate retains control over what factual content is ever surfaced,
  directly supporting the groundedness quality metric in
  [../docs/success-metrics.md](../docs/success-metrics.md).
- Restricting RAG to one function keeps the rest of the use case (route,
  alternatives, group-fit) simpler and easier to test, since those remain
  deterministic recommendation logic over structured data.
- The refusal behaviour (declining to answer when no approved passage exists)
  directly reduces hallucination risk, a named risk in
  [../docs/risks-and-mitigations.md](../docs/risks-and-mitigations.md) once
  available.

**Negative / trade-offs:**
- The approved knowledge base must be actively maintained and kept current by
  estate staff; RAG only mitigates hallucination if the underlying content is
  accurate and well-organised — this is a content-operations responsibility,
  not something the architecture alone guarantees.
- Visitors may occasionally ask questions the knowledge base does not yet
  cover; the "no approved information" refusal is honest but could feel like a
  gap in visitor experience until content coverage improves.
- Maintaining two different technical approaches within one use case (RAG for
  information delivery, structured recommendation for routing) adds some
  design and testing surface compared to a single uniform technique, though
  this is judged to be less complex overall than forcing one technique to fit
  both problems.

**Operational impact:**
- Estate staff responsible for the approved knowledge base must review and
  version content regularly (see
  [personalised-visitor-experience.md](../use-cases/personalised-visitor-experience.md),
  "Distinguishing current from outdated information"); this is a content
  governance process alongside the technical one.
- Visitor-reported bad recommendations (FR14) that stem from information
  delivery specifically should be routed to knowledge-base review, separate
  from routing/recommendation-quality review.

**Reversibility:**
- If a future need arises for AI-assisted routing recommendations to also
  reference free-text content (for example, a personalised written summary of
  a visitor's day), that could be introduced as a new, separately justified
  decision rather than by expanding this ADR's scope.
