# Vector — High-Performance Observability Data Pipeline (Rust/GitHub Actions)

Vector is an open-source, end-to-end observability data pipeline written in Rust. It functions as both an agent and an aggregator, enabling users to collect, transform, and route logs, metrics, and traces. The project emphasizes reliability, performance (up to 10x faster than alternatives), and control over observability data.

## Core Functionality

- **Collect**: Ingests data from various sources (e.g., files, sockets, APIs, specific services like Kubernetes logs, Prometheus, Kafka).
- **Transform**: Modifies data using the Vector Remap Language (VRL) or Lua for tasks like parsing, filtering, and enrichment.
- **Route**: Sends processed data to diverse sinks (e.g., Elasticsearch, S3, Datadog, Loki, Kafka).

## Technology Stack

- **Language**: Rust (edition 2024, MSRV 1.92)
- **CI/CD**: GitHub Actions (extensive workflows for testing, building, releasing)
- **Build System**: Cargo, Make
- **Containerization**: Docker, Podman (for dev environment and deployments)
- **Configuration**: TOML, YAML, CUE (for documentation generation)
- **Testing**: `cargo test`, `cargo nextest`, `proptest`, integration tests with Docker/Kubernetes, blackbox tests with `vector-test-harness`.
- **Documentation**: Hugo (for website), CUE (for reference docs generation), Markdown.
- **Metrics/Tracing**: `metrics` crate, `tracing` crate.

## Project Structure

- `src/`: Main application logic, sources, transforms, sinks.
- `lib/`: Shared Rust crates (e.g., `lib/codecs`, `lib/vector-core`, `lib/vector-vrl`).
- `config/`: Default and example configurations.
- `distribution/`: Packaging and deployment artifacts (Debian, RPM, Docker, MSI, Kubernetes Helm charts).
- `docs/`: Internal developer documentation.
- `website/`: Public-facing documentation and marketing site.
- `.github/`: GitHub Actions workflows, issue templates, code owners.

## Development Workflow

Developers typically use `cargo` commands and `make` targets for building, testing, and linting. A Docker/Podman-based development environment is provided for consistency and ease of setup. Code changes are managed via pull requests, adhering to conventional commit messages and undergoing thorough review processes.

## Community & Support

Vector has a large and active community, with over 500 contributors. Support and discussions happen on [vector.dev/community](https://vector.dev/community/). The project is maintained by Datadog's Community Open Source Engineering team.
