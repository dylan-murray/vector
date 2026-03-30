# Vector's Internal Architecture — Component-Based Data Processing Graph

Vector's architecture is centered around a directed, acyclic graph (DAG) of components: sources, transforms, and sinks. This logical graph directly maps to the internal execution model, where each component runs as a Tokio task and communicates via channels.

## Core Concepts

- **Components**: Fundamental building blocks of the data pipeline (sources, transforms, sinks).
- **Topology**: The overall graph structure defined by the user's configuration.
- **Tokio Tasks**: Each component is typically executed within one or more asynchronous Tokio tasks.
- **Channels**: Components communicate by sending events through asynchronous channels.

## Component Construction and Wiring

1.  **Configuration Parsing**: User-defined TOML/YAML configuration is parsed into `SourceConfig`, `TransformConfig`, and `SinkConfig` structs.
2.  **Component Building**: Each `*Config` struct's `build` method constructs the actual runtime component.
3.  **Channel Setup**: Channels are established between components based on the `inputs` and `outputs` declarations in the configuration.

### Sources

-   **Server Task**: The primary task for the source, responsible for ingesting data.
-   **Pump Task**: Forwards events from the source to downstream components.
-   **`SourceSender`**: Manages sending events to multiple outputs defined by `SourceConfig::outputs`.
-   **`Fanout`**: Distributes events from a single source output to all connected downstream inputs.

### Transforms

Transforms process events received from upstream and send them downstream. They can be categorized:

-   **Synchronous Transforms (`Function`, `Synchronous`)**:
    -   Process events synchronously, often in batches.
    -   If `enable_concurrency` is true, new tasks are spawned for processing chunks of events, leveraging Tokio's work-stealing scheduler. The main transform task ensures output order is preserved.
-   **Task Transforms**:
    -   Perform arbitrary, asynchronous, stream-based transformations.
    -   Define most of their logic internally and are applied by passing the input channel to their `transform` method.

### Sinks

Sinks are responsible for sending processed events to external destinations. They involve additional complexities:

-   **Healthchecks**: One-off tasks run at startup to validate configuration against external systems (e.g., connectivity, permissions). Designed to prevent false negatives.
-   **Buffers**: Configurable mechanisms for handling backpressure.
    -   **Memory Buffers**: Store events in RAM.
    -   **Disk Buffers**: Persist events to disk, ensuring durability across restarts and reloads. These are managed carefully during topology changes to maintain data integrity.
-   **Finalization**: Sinks defer event finalization until successful delivery, crucial for end-to-end acknowledgements.

## Connecting Components

-   The `RunningTopology` (in `src/topology/running.rs`) manages the connections.
-   A component's input is the sender side of a channel/buffer.
-   A component's output is a `fanout::ControlChannel` that allows dynamically adding/removing destinations.
-   The `connect_diff` function wires components by sending clones of downstream inputs to upstream output control channels, allowing `Fanout` instances to distribute events.

## Spawning the Topology

-   After all components are built and wired, their respective tasks are spawned.
-   Bookkeeping includes setting up tracing spans, error handlers, and shutdown management.

## Key Design Principles

-   **Modularity**: Components are self-contained and interchangeable.
-   **Asynchronous Processing**: Leverages Tokio for high concurrency and I/O efficiency.
-   **Reliability**: Built-in mechanisms for error handling, backpressure, and data durability.
-   **Observability**: Extensive internal telemetry (logging, metrics, tracing) for monitoring pipeline health and performance.
