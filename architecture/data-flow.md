# Data flow

This document traces how data moves through the baseline architecture described
in [target-state.md](target-state.md), with particular attention to the case that
matters most for this estate: **an intermittent Wi-Fi connection**. Component
names match [target-state.md](target-state.md).

## Flow 1: Ticket purchase and entry

A visitor buys a ticket and enters the estate.

```mermaid
flowchart LR
    Visitor[Visitor] -->|1\. Requests ticket| App[Visitor App/Portal]
    App -->|2\. Ticket request| Ticketing[Ticketing Service]
    Ticketing -->|3\. Ticket sold event| Backbone[[Event Backbone]]
    Backbone -->|4\. Update| OpStore[(Operational Data Store)]
    Backbone -->|4\. Update| AnalyticalStore[(Analytical Data Store)]
    Visitor -->|5\. Scans pass| AccessPoint[Access Control Point]
    AccessPoint -->|6\. Entry event| Gateway[Zone Edge Gateway]
    Gateway -->|7\. Forward when connected| Sync[Cloud Sync Service]
    Sync -->|8\. Publish| Backbone
```

This flow is entirely within the cloud platform except for the Access Control
Point, which sits in a zone and is therefore subject to Wi-Fi gaps — hence step 6
goes through the same Zone Edge Gateway as other zone telemetry, not directly to
the cloud.

## Flow 2: MQTT telemetry during normal connectivity

A ride or enclosure sensor publishes telemetry while the zone's Wi-Fi is
available.

```mermaid
flowchart LR
    Device[MQTT Device] -->|1\. Publish telemetry| Gateway[Zone Edge Gateway]
    Gateway -->|2\. Forward immediately| Sync[Cloud Sync Service]
    Sync -->|3\. Deduplicate, order| Sync
    Sync -->|4\. Publish event| Backbone[[Event Backbone]]
    Backbone -->|5\. Update| OpStore[(Operational Data Store)]
    Backbone -->|5\. Update| AnalyticalStore[(Analytical Data Store)]
```

## Flow 3: MQTT telemetry during a Wi-Fi outage (the critical path)

This is the flow the architecture is explicitly designed around (resilience
characteristic in
[../docs/architecture-characteristics.md](../docs/architecture-characteristics.md)).

```mermaid
flowchart TD
    Device[MQTT Device] -->|1\. Publish telemetry| Gateway[Zone Edge Gateway]
    Gateway -->|2\. Wi-Fi unavailable| Buffer[(Local Buffer Store)]
    Buffer -->|3\. Retain with original timestamp and order| Buffer
    Gateway -->|4\. Continue accepting new telemetry| Buffer
    Connectivity{Wi-Fi restored?}
    Buffer --> Connectivity
    Connectivity -->|Yes| Sync[Cloud Sync Service]
    Sync -->|5\. Forward buffered events in order| Sync
    Sync -->|6\. Deduplicate against already-received events| Sync
    Sync -->|7\. Publish event| Backbone[[Event Backbone]]
    Backbone -->|8\. Update, marked as delayed| OpStore[(Operational Data Store)]
    Backbone -->|8\. Update| AnalyticalStore[(Analytical Data Store)]
```

Key properties of this flow, required by OE1 and OE4 in
[../docs/requirements.md](../docs/requirements.md):

- The Zone Edge Gateway never blocks or drops new telemetry because of a Wi-Fi
  outage; it keeps buffering locally.
- Original event timestamps and ordering are preserved in the Local Buffer Store,
  so downstream consumers (including future AI capabilities) can reconstruct the
  true sequence of events even though they arrive late.
- The Cloud Sync Service is responsible for deduplication, since a gateway may
  retry forwarding after a partial failure.
- Data updated from a delayed/buffered batch is marked as such in the
  Operational Data Store, so operators and any AI capability can distinguish
  "fresh" from "recently caught up" data — this supports the explainability and
  data-integrity characteristics described in
  [../docs/architecture-characteristics.md](../docs/architecture-characteristics.md).

## Flow 4: Operator and keeper access to data

Operators, keepers and veterinarians consume both operational and analytical
data through a single interface.

```mermaid
flowchart LR
    Console[Operator Console] -->|Current occupancy, open alerts| OpStore[(Operational Data Store)]
    Console -->|Trends, history| AnalyticalStore[(Analytical Data Store)]
    Operator[Park Operator / Keeper / Veterinarian] -->|Views, acts| Console
```

No AI component appears in this baseline data flow. Once the AI use cases are
layered on top (see [../use-cases/](../use-cases/)), they read from the Event
Backbone and Analytical Data Store and write recommendations/alerts back into the
Operational Data Store for the Operator Console and Visitor App/Portal to
surface — the data-flow shape described here does not change.

## Data ownership summary

| Data | Origin | Primary store | Consumers |
|---|---|---|---|
| Ticket sales | Ticketing Service | Operational Data Store | Visitor App/Portal, Operator Console |
| Entry/exit events | Access Control Point | Operational Data Store, then Analytical Data Store | Operator Console, future Visitor Intelligence |
| Ride/enclosure telemetry | MQTT Device | Operational Data Store (current state), Analytical Data Store (history) | Operator Console, future Animal Health / Visitor Intelligence |
| Keeper observations | Operator Console input | Operational Data Store | Operator Console, future Animal Health Intelligence |

See [../docs/assumptions.md](../docs/assumptions.md) (D1–D3) for the assumptions
behind these data origins.
