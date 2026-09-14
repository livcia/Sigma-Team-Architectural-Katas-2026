Date: 2026-09-14

# ADR-002: Edge buffering and offline mode

## Status
Proposed

## Context

Wi-Fi coverage across Von Digitalis Estates is explicitly patchy (technical
constraint TC1 in [../docs/requirements.md](../docs/requirements.md)). MQTT
devices spread across 40 rides and 55 enclosures, plus Access Control Points,
must keep capturing ticketing, entry and telemetry data even when a zone
temporarily loses connectivity to the cloud. Both the amusement-park and
animal-care sides of the business depend on this data not being lost, since it
feeds [Visitor Intelligence](../use-cases/visitor-intelligence.md) and, more
critically, [Animal Health Intelligence](../use-cases/animal-health-intelligence.md).

Alternatives considered:

- **Assume continuous connectivity** and treat outages as rare failures to be
  retried. Rejected: the brief states connectivity is patchy as a normal
  condition, not an edge case; this would risk regular data loss.
- **Devices buffer locally themselves**, each holding its own outbound queue
  until they can reach the cloud directly. Rejected: with potentially many
  devices per zone, this multiplies the number of places that need durable
  storage, retry logic and monitoring, and devices are typically more
  resource-constrained than a dedicated gateway.
- **A local edge gateway per zone that buffers on behalf of its devices**,
  forwarding to the cloud once connectivity allows, as described in
  [../architecture/target-state.md](../architecture/target-state.md) and
  [../architecture/deployment-view.md](../architecture/deployment-view.md).

## Decision

Each estate zone has a **Zone Edge Gateway** with a co-located **Local Buffer
Store**. MQTT devices and Access Control Points in that zone publish to their
local gateway, never directly to the cloud. The gateway buffers events durably,
preserving original capture timestamps and event order, whenever the zone's
Wi-Fi is unavailable, and forwards buffered events to the **Cloud Sync
Service** once connectivity returns (see
[../architecture/data-flow.md](../architecture/data-flow.md), Flow 3). Every
AI capability built on top of this data defines an explicit degraded/offline
mode (see individual use cases) rather than assuming fresh data is always
available.

## Consequences

**Positive:**
- Core operations (ticketing, entry, telemetry capture) do not depend on
  continuous cloud reachability, directly supporting the resilience
  characteristic in
  [../docs/architecture-characteristics.md](../docs/architecture-characteristics.md).
- A Wi-Fi outage is isolated to the affected zone; other zones and core
  ticketing continue unaffected.
- Buffered data preserves original timestamps and order, so downstream
  consumers (data stores, AI capabilities) can reconstruct the true sequence
  of events even when they arrive late, supporting the data-integrity
  characteristic.

**Negative / trade-offs:**
- Requires physical infrastructure (a gateway and durable local storage) per
  zone, which is additional hardware and operational surface compared to a
  single centralised ingestion point.
- Data used by any AI capability may be **stale during an outage and for a
  period after it**, until buffered events are synchronised; every AI
  capability must handle this explicitly (reduced confidence, fallback to
  rules, or reliance on manual observation), rather than silently treating
  delayed data as current.
- Deduplication and out-of-order handling across gateways add complexity to
  the Cloud Sync Service that would not exist with a single always-connected
  ingestion path.

**Operational impact:**
- The system administrator is responsible for monitoring gateway health,
  buffer capacity, and synchronisation lag per zone (see
  [../docs/architecture-characteristics.md](../docs/architecture-characteristics.md),
  operability characteristic).
- Buffer capacity must be sized to cover realistic outage durations; this is an
  operational sizing decision to be validated with real connectivity data, not
  a fixed architectural parameter.

**Reversibility:**
- If Wi-Fi coverage across the estate improves substantially over time, the
  buffering mechanism can remain in place with lower utilisation at negligible
  cost; removing it entirely would only be considered if connectivity became
  reliably continuous estate-wide, which is not assumed in this design.
