# Vector's Testing Strategy

Vector employs a multi-faceted testing strategy to ensure reliability, correctness, and performance. This document outlines the different types of tests and how to run them.

## Unit Tests

-   **Scope**: Focus on individual functions, modules, or small components in isolation.
-   **Characteristics**: Do not require external services, making them fast and suitable for local development.
-   **Execution**: Run with `cargo test`.

    ```bash
    cargo test
    ```

-   **Faster Iteration**: For specific component unit tests, use `cargo-watch`:

    ```bash
    cargo install cargo-watch
    cargo watch -s clear -s 'cargo test --lib --no-default-features --features=<component type>-<component id> <component type>::<component id>'
    # Example for `reduce` transform:
    cargo watch -s clear -s 'cargo test --lib --no-default-features --features=transforms-reduce transforms::reduce'
    ```

## Integration Tests

-   **Scope**: Verify that Vector components correctly interact with external services.
-   **Characteristics**: Require external services, typically run in Docker containers.
-   **Rules for New Integration Tests**:
    -   Services must run in Docker containers.
    -   Services must use unique, configurable ports via environment variables.
    -   Add a `test-integration-<name>` target to the root `Makefile` to start the service before testing.
    -   Add the integration name to the `test-integration` job's matrix in `.github/workflows/integration-test.yml`.
-   **Execution**: Run with `make test-integration-<name>`.

    ```bash
    make test-integration-aws
    # Run all integration tests:
    make test-integration
    ```

## Blackbox Tests

-   **Tool**: [Vector's test harness](https://github.com/vectordotdev/vector-test-harness).
-   **Scope**: End-to-end testing, performance benchmarking, and correctness validation in realistic environments.
-   **Execution**: Typically run in CI, but can be triggered manually via PR comments (see `docs/CONTRIBUTING.md`).

## Property Tests

-   **Tool**: [Proptest](https://github.com/proptest-rs/proptest).
-   **Scope**: Generates diverse inputs to test code invariants and edge cases, complementing example-based tests.
-   **Usage**: Preferred for testing complex logic where exhaustive input coverage is difficult.

## Kubernetes E2E Tests

-   **Scope**: End-to-end validation of Kubernetes-specific integrations (e.g., `kubernetes_logs` source).
-   **Characteristics**: Complex, resource-intensive tests that run against a Kubernetes cluster.
-   **Prerequisites**:
    -   `tilt`
    -   `docker`
    -   `kubectl`
    -   `minikube` (or other k8s cluster)
    -   `bash`
    -   `cross` (`cargo install cross`)
    -   `helm`
-   **Execution**:
    -   **Automatic (Development)**: Use `tilt up` from the Vector root directory to detect changes, rebuild images, and update Kubernetes resources.
    -   **Manual (Full E2E)**: Use `make test-e2e-kubernetes`.

    ```bash
    # Example for full E2E test:
    CONTAINER_IMAGE_REPO=<your_docker_username>/vector-test make test-e2e-kubernetes
    # Faster development build:
    QUICK_BUILD=true USE_MINIKUBE_CACHE=true make test-e2e-kubernetes
    ```

## General Testing Tips

-   **`sccache`**: Use `sccache` to speed up Rust compilation by caching build artifacts. Install it and configure your `~/.cargo/config`.
-   **`make check-all`**: Runs a comprehensive suite of checks (formatting, clippy, docs, examples, scripts, deny, generated docs, licenses).
-   **`make check-clippy`**: Runs the Clippy linter.
-   **`make check-fmt`**: Checks code formatting.
-   **`make check-licenses`**: Verifies third-party license compliance.
-   **`make check-generated-docs`**: Ensures auto-generated documentation is up-to-date.
-   **`make check-events`**: Validates internal metrics and event patterns.

## Flaky Tests

-   If a test fails intermittently, first verify it's not related to your changes.
-   Check if it's failing on `master` or if rerunning CI resolves it.
-   Attempt to reproduce locally.
-   File or link an existing issue for the flaky test.
