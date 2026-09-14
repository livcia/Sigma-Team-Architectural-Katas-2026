Date: 2026-09-14

# ADR-001: AI model abstraction

## Status
Proposed

## Context

Von Digitalis Estates' solution relies on three AI capabilities ([Visitor
Intelligence](../use-cases/visitor-intelligence.md), [Animal Health
Intelligence](../use-cases/animal-health-intelligence.md) and [Personalised
Visitor Experience](../use-cases/personalised-visitor-experience.md)). AI
technology and the provider landscape change quickly: today's best forecasting
technique, anomaly-detection approach, or generative model may be outdated,
repriced, or discontinued within the multi-year horizon of this project
(the competition brief explicitly raises this risk — see
[../architectural-katas-2026.md](../architectural-katas-2026.md)).

If AI models and providers are called directly from ticketing, telemetry
processing, and visitor-facing code, replacing a model means changing every
place that calls it, re-testing unrelated functionality, and risking a large,
high-stakes migration under time pressure. The three use cases also have very
different AI needs (forecasting/anomaly-detection style models for Visitor
Intelligence and Animal Health Intelligence, versus a retrieval-and-generation
style model for Personalised Visitor Experience), so a single hard-coded
integration approach would not fit all of them equally well.

Alternatives considered:

- **Direct integration** of each AI capability's consuming code with a specific
  model/provider API. Simplest to build initially, but ties the architecture
  characteristics of testability, operability and resilience (see
  [../docs/architecture-characteristics.md](../docs/architecture-characteristics.md))
  to a single vendor's roadmap and pricing.
- **A shared AI abstraction/gateway layer** that each use case calls through a
  stable, logical interface, with the actual model/provider selected and
  configured behind that layer.
- **A full multi-model orchestration platform** with automatic routing between
  providers based on cost/latency. Rejected as premature complexity for the
  estate's actual scale (5,000–15,000 daily visitors — see
  [../docs/assumptions.md](../docs/assumptions.md), D5); this can be
  reconsidered later if justified.

## Decision

Every AI capability is accessed through a **logical AI abstraction/gateway
layer**, not called directly by consuming components (Operator Console,
Visitor App/Portal, or the event-processing pipeline). Each use case defines a
stable input/output contract (for example, "given current occupancy and
historical data, return a demand forecast with a confidence indicator") against
this layer. The specific model, technique, or provider behind the layer is an
implementation detail that can be swapped, A/B tested, or rolled back without
changing the contract or the consuming components.

This decision is deliberately technology-agnostic: no specific model family or
cloud AI provider is named here, consistent with `../AGENT-CONTEXT.md` and
[../README.md](../README.md). The abstraction is a logical decision, not a
product choice.

## Consequences

**Positive:**
- Consuming components (Operator Console, Visitor App/Portal) are insulated
  from AI provider changes, directly mitigating the "what if the best model
  today becomes outdated / a provider changes pricing or shuts down" risks
  raised in the brief.
- Each AI capability can be evaluated, tested and rolled back independently
  (supports the testability characteristic in
  [../docs/architecture-characteristics.md](../docs/architecture-characteristics.md)
  and the monitoring approach in ADR-006).
- Different AI techniques appropriate to each use case (forecasting/anomaly
  detection versus retrieval-augmented generation, see ADR-005) can coexist
  behind their own contracts without forcing a one-size-fits-all integration.

**Negative / trade-offs:**
- Introduces an extra layer of indirection and a contract to design and
  maintain for each capability, which is additional upfront design effort
  compared to direct integration.
- The abstraction must be genuinely stable; a poorly designed contract that
  leaks provider-specific concepts would undermine the whole benefit of this
  decision.

**Operational impact:**
- The system administrator persona (see
  [../docs/stakeholders-and-personas.md](../docs/stakeholders-and-personas.md))
  owns the process of evaluating, swapping and rolling back models behind this
  layer, as described further in ADR-004 and ADR-006.

**Reversibility:**
- This decision can be revisited if a single AI capability turns out to need
  provider-specific features that cannot be reasonably abstracted; in that
  case, the abstraction for that specific capability could be narrowed, without
  affecting the other two use cases.
