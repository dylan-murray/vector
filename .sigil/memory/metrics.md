# Vector's Internal Metrics and Instrumentation

Vector utilizes a robust internal metrics system to provide observability into its operation. This document outlines the principles, naming conventions, and emission guidelines for metrics and internal events.

## Core Concepts

-   **Event-Driven Instrumentation**: Telemetry is primarily driven by internal events, which then trigger the emission of metrics and logs.
-   **Metrics Crate**: The [`metrics`](https://docs.rs/metrics/latest/metrics/) crate is used for emitting counters, gauges, and histograms.
-   **Tracing Crate**: The [`tracing`](https://docs.rs/tracing/latest/tracing) crate is used for logging and distributed tracing (spans).

## Naming Conventions

### Namespaces

Internal events and metrics are namespaced based on their origin:

-   `Component`: For events/metrics related to sources, transforms, and sinks.
-   `Buffer`: For events/metrics related to internal buffers.
-   `Topology`: For events/metrics related to the overall pipeline topology.

### Event Naming

Internal event names follow a strict `CamelCase` convention:

-   **Format**: `<Namespace><Noun><Verb>[Error]`
    -   `Namespace`: The internal domain (e.g., `Component`, `Buffer`).
    -   `Noun`: The subject of the event (e.g., `Bytes`, `Events`).
    -   `Verb`: Past tense verb describing the action (e.g., `Received`, `Sent`, `Processed`).
    -   `[Error]`: Optional suffix if the event represents an error.
-   **Characters**: Must only contain ASCII alphanumeric and lowercase characters.

### Metric Naming

Metric names generally follow [Prometheus metric naming standards](https://prometheus.io/docs/practices/naming/) and use `snake_case`.

-   **Format**: `<namespace>_<name>_<unit>_[total]`
    -   `namespace`: The internal domain (e.g., `component`, `buffer`).
    -   `name`: Describes the measurement (e.g., `memory_rss`, `requests`).
    -   `unit`: A single [base unit](https://en.wikipedia.org/wiki/SI_base_unit) in plural form (e.g., `seconds`, `bytes`).
    -   `_total`: Suffix for counters.
-   **Tags**: Metrics should be broad in purpose, using tags to differentiate characteristics (e.g., `host_cpu_seconds_total{cpu="0",mode="idle"}`).

## Emission Guidelines

### Batching

-   Instrumentation *SHOULD* be batched whenever possible to improve performance.
-   Telemetry *SHOULD* be emitted for entire event batches, not individual events.
-   Benchmarking *SHOULD* prove performance benefits of batching.

### Events

-   Instrumentation *SHOULD* be event-driven, with individual events driving metric and log emission.
-   Metrics and logs *SHOULD NOT* be emitted directly, except where impossible (e.g., external crates).

#### `Error` Event

-   *MUST* be emitted when an error occurs during component execution.
-   **Properties**:
    -   `error_code`: An error code (e.g., `decoder_frame`, `encoder_serialize`).
    -   `error_type`: The type of error condition (e.g., `PARSER_FAILED`, `IO_FAILED`).
    -   `stage`: The stage at which the error occurred (`receiving`, `processing`, `sending`).
-   **Metrics**: Increments `<namespace>_errors_total` with properties as tags.
-   **Logs**: Logs a descriptive error message at `error` level, rate-limited to 10 seconds.
-   **Dropped Events**: *MUST* emit an `EventsDropped` event if the error causes events to be dropped.

#### `EventsDropped` Event

-   *MUST* be emitted when events are dropped.
-   *MUST NOT* be emitted for operations that Vector will retry (unless the retry fails).
-   *MUST NOT* be emitted before events are created (e.g., source decode failure is `ComponentError`, not `EventsDropped`).
-   **Properties**:
    -   `count`: Number of events dropped.
    -   `intentional`: Boolean indicating if the drop was intentional (e.g., `filter` transform) or unintentional (e.g., error).
    -   `reason`: Short, user-friendly reason for the drop.
-   **Metrics**: Increments `<namespace>_discarded_events_total`.
-   **Logs**: Logs at `debug` (intentional) or `error` (unintentional) level, rate-limited to 10 seconds.

#### `ComponentEventsReceived` (Deprecated)

-   *MUST* be emitted immediately after creating or receiving Vector events.
-   **Metrics**: Increments `component_received_events_total` and `component_received_event_bytes_total`.

#### `ComponentBytesReceived` (Deprecated)

-   *Sources* *MUST* emit after receiving, decompressing, and filtering bytes.
-   **Properties**: `byte_size`, `protocol`, `http_path`.
-   **Metrics**: Increments `component_received_bytes_total`.

#### `ComponentBytesSent`

-   *Sinks* *MUST* emit immediately after *successful* transmission of bytes.
-   *MUST NOT* emit for sinks that only expose data (e.g., `prometheus_exporter`).
-   **Properties**: `byte_size`, `protocol`, `endpoint`, `file`.
-   **Metrics**: Increments `component_sent_bytes_total`.

#### `ComponentEventsSent`

-   *MUST* be emitted immediately after *successful* transmission of Vector events to downstream components.
-   *MUST NOT* emit for pull-based sinks.
-   **Properties**: `count`, `byte_size`, `output` (optional).
-   **Metrics**: Increments `component_sent_events_total` and `component_sent_event_bytes_total`.

#### `SinkNetworkBytesSent` (To be implemented)

-   *Sinks* *MUST* emit after egress of *raw network bytes* (after processing like encryption/compression).
-   **Properties**: `byte_size` (closest representation of raw bytes).
-   **Metrics**: Increments `component_sent_network_bytes_total`.

#### `SourceNetworkBytesReceived` (To be implemented)

-   *Sources* *MUST* emit after ingress of *raw network bytes* (before processing like decryption/decompression).
-   **Properties**: `byte_size`.
-   **Metrics**: Increments `component_received_network_bytes_total`.

## Disabling Internal Log Rate Limiting

During development, internal log rate limiting can be disabled:

-   **Globally**: Use `vector --config vector.yaml -r 1` or `VECTOR_INTERNAL_LOG_RATE_LIMIT=1` environment variable.
-   **Per Log Statement**: Use `internal_log_rate_limit = false` or `internal_log_rate_secs = <seconds>` in `tracing` macros.
