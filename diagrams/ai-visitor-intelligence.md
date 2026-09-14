# Diagram: AI Visitor Intelligence

This diagram supports [../use-cases/visitor-intelligence.md](../use-cases/visitor-intelligence.md).
It uses simple boxes and arrows, consistent with the Mermaid conventions in
`../AGENT-CONTEXT.md`, and reuses the component names from
[../architecture/target-state.md](../architecture/target-state.md) and
[../architecture/event-driven-architecture.md](../architecture/event-driven-architecture.md).

## Diagram

```mermaid
flowchart TD
    subgraph Data Inputs
        Ticketing[Ticketing Service:\nticket.sold]
        Access[Access Control Points:\nvisitor.entered / visitor.exited]
        Rides[MQTT Devices - Rides:\nride.telemetry]
        External[External Context:\nWeather, Scheduled Events]
    end

    Backbone[[Event Backbone]]

    subgraph Data Stores
        OpStore[(Operational Data Store\ncurrent occupancy, active queues)]
        AnalyticalStore[(Analytical Data Store\nhistorical trends, context)]
    end

    AI{{Visitor Intelligence\nAI Component}}

    subgraph AI Outputs
        Forecast[Demand Forecast]
        Congestion[Congestion / Queue Alert]
        Popularity[Zone Popularity Ranking]
        Staffing[Staffing Recommendation]
    end

    Operator[Park Operator]
    Console[Operator Console]

    Ticketing --> Backbone
    Access --> Backbone
    Rides --> Backbone
    External --> AnalyticalStore

    Backbone --> OpStore
    Backbone --> AnalyticalStore

    OpStore --> AI
    AnalyticalStore --> AI

    AI --> Forecast
    AI --> Congestion
    AI --> Popularity
    AI --> Staffing

    Forecast --> Console
    Congestion --> Console
    Popularity --> Console
    Staffing --> Console

    Console --> Operator
    Operator -->|Approves, dismisses,\nor acts manually| Console
    Operator -->|Decision + feedback| OpStore
    OpStore -.->|Feedback loop:\nimproves future forecasts| AI

    classDef input fill:#fef3d0,stroke:#b8860b;
    classDef store fill:#e0f0ff,stroke:#3366aa;
    classDef ai fill:#f3e0ff,stroke:#7a3fa0;
    classDef output fill:#e0ffe6,stroke:#2e7d32;
    classDef person fill:#ffe9d6,stroke:#cc6600;

    class Ticketing,Access,Rides,External input;
    class Backbone,OpStore,AnalyticalStore store;
    class AI ai;
    class Forecast,Congestion,Popularity,Staffing output;
    class Operator,Console person;
```

## Legend

| Shape / colour | Meaning |
|---|---|
| Yellow boxes | Data inputs: estate event sources and external context |
| Blue boxes / cylinders | Event Backbone and data stores (operational and analytical) |
| Purple hexagon | The Visitor Intelligence AI component (behind a model/provider abstraction, see [../README.md](../README.md)) |
| Green boxes | AI outputs: recommendations and analysis, never direct system actions |
| Orange boxes | Human operator and the console they use |
| Dashed arrow | Feedback loop: operator decisions and outcomes feed back to improve future AI outputs |

## Notes

- The AI component only reads from the Operational and Analytical Data Stores;
  it never writes directly to ticketing, staffing or ride-control systems (see
  [../use-cases/visitor-intelligence.md](../use-cases/visitor-intelligence.md),
  "Operator decisions").
- The feedback loop (operator decision recorded back into the Operational Data
  Store) is what allows forecast and alert quality to be evaluated and improved
  over time, per AI4 in [../docs/requirements.md](../docs/requirements.md).
- When any data input is delayed or missing (for example, buffered during a
  Wi-Fi outage, see [../architecture/data-flow.md](../architecture/data-flow.md)),
  the AI component still produces outputs but marks them as reduced-confidence,
  as described in the use case document.
