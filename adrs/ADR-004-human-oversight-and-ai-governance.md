Date: 2026-09-14

# ADR-004: Human oversight and AI governance

## Status
Proposed

## Context

All three AI capabilities ([Visitor
Intelligence](../use-cases/visitor-intelligence.md), [Animal Health
Intelligence](../use-cases/animal-health-intelligence.md), [Personalised
Visitor Experience](../use-cases/personalised-visitor-experience.md)) touch
outcomes with real consequences: operational cost, animal welfare, visitor
trust, and in some enclosures, physical safety (assumption A6 in
[../docs/assumptions.md](../docs/assumptions.md) notes hazardous species such
as the estate's exotic collection). The competition brief and
`../AGENT-CONTEXT.md` both require that AI support, not replace, human
decision-making on safety-critical, financial, veterinary or irreversible
matters.

Without an explicit governance model, individual use cases could each invent
their own rules for when AI output is trusted, who can override it, and how
mistakes are reviewed — leading to inconsistent risk exposure across the
estate and making it hard for judges (or future operators) to understand where
accountability sits.

Alternatives considered:

- **Per-use-case, ad hoc oversight** left to each capability's own design.
  Rejected: risks inconsistent standards, and makes it harder to reason about
  overall AI risk across the estate.
- **Fully autonomous AI action** for at least the lowest-risk capability
  (for example, letting Visitor Intelligence automatically adjust staffing).
  Rejected: even "lower risk" operational decisions have real cost and morale
  impact if wrong, and the brief's judging criteria explicitly value
  demonstrated human oversight.
- **A single, consistent governance model applied across all three use
  cases**, defining decision rights, alert levels, and audit requirements once,
  and referenced by each use case. This is the approach adopted.

## Decision

A consistent governance model applies across all three AI capabilities:

1. **AI produces recommendations, alerts, rankings or explanations — never
   autonomous actions** on safety-critical, financial, veterinary or
   irreversible matters (AI2 in [../docs/requirements.md](../docs/requirements.md)).
2. **A named human role owns the final decision** for each capability: the
   park operator for Visitor Intelligence, the keeper/veterinarian for Animal
   Health Intelligence, and the visitor for Personalised Visitor Experience
   (see [../docs/stakeholders-and-personas.md](../docs/stakeholders-and-personas.md)).
3. **Every AI output must be explainable** — accompanied by the contributing
   signals or grounding sources, so a human is not asked to blindly trust a
   result (NFR5 in [../docs/requirements.md](../docs/requirements.md)).
4. **Every human decision on an AI output is recorded** (confirm, dismiss,
   escalate, accept, reject) for audit, forming the basis for the evaluation
   approach in ADR-006 (SPA4 in [../docs/requirements.md](../docs/requirements.md)).
5. **Any capability can be suspended in favour of its defined fallback**
   (rule-based alerting, manual planning, or static content) without waiting
   for a full model replacement, if governance review determines it is
   underperforming or unsafe.

## Consequences

**Positive:**
- Judges and future operators can reason about AI risk consistently across all
  three use cases, rather than needing to evaluate three different governance
  approaches.
- Keeping humans as the final decision-makers directly addresses the brief's
  concern about "how does AI improve operations" without introducing
  unmanaged risk.
- A consistent audit requirement across all capabilities simplifies compliance
  review and incident investigation.

**Negative / trade-offs:**
- Human-in-the-loop review adds latency and operational cost compared to fully
  autonomous action; for example, a congestion alert still requires an
  operator to act, rather than automatically reallocating staff.
- Requiring explainability for every output constrains which AI techniques are
  practical to use (see ADR-003 and ADR-005), potentially ruling out
  higher-accuracy but less explainable approaches in some cases.
- Consistent governance may feel heavier than necessary for genuinely
  low-stakes recommendations (for example, a minor route suggestion); this is
  accepted as the cost of a single, understandable governance model rather
  than differentiated rules per capability.

**Operational impact:**
- The Countess/estate management is the ultimate owner of risk-acceptance
  decisions, such as approving a new AI capability or accepting a documented
  trade-off (see
  [../docs/stakeholders-and-personas.md](../docs/stakeholders-and-personas.md)).
- The system administrator implements and monitors the technical controls
  (explainability, audit logging, suspension mechanism) that this governance
  model requires.

**Reversibility:**
- This governance model can be relaxed for a specific, genuinely low-risk
  recommendation type if experience shows the oversight is unnecessary
  overhead; any such change should be documented as a new ADR superseding this
  one for that specific case, not a silent deviation.
