# Vector's Kubernetes Integration Development

This document details the architecture, development, and testing processes for Vector's Kubernetes integrations, particularly the `kubernetes_logs` source.

## Architecture

Vector's Kubernetes integration is largely based on [RFC 2221](../rfcs/2020-04-04-2221-kubernetes-integration.md).

### Operation Logic

-   The `kubernetes_logs` source connects to the Kubernetes API.
-   It performs a streaming watch request for `Pod`s running on the same `Node` as Vector.
-   Upon receiving the list of `Pod`s, it starts collecting logs from their corresponding log files.
-   Only plaintext (non-gzipped) log files are processed.
-   Log files are parsed into events, and these events are enriched with metadata from their respective `Pod`s, correlated via the log file path.
-   Processed events are then passed to the Vector topology.

### Codebase Location

-   **Kubernetes API Client**: `src/kubernetes` (custom client and machinery).
-   **`kubernetes_logs` Source**: `src/sources/kubernetes_logs`.
-   **E2E Test Framework**: `lib/k8s-test-framework`.
-   **E2E Tests**: `lib/k8s-e2e-tests`.
-   **Distribution Artifacts**: `distribution/docker`, `distribution/kubernetes`, and the external [Helm chart](https://github.com/vectordotdev/helm-charts/).
-   **Development Assistance**: `Tiltfile` and `tilt/` directory.

## Development

Developing Kubernetes-related components involves a specialized workflow for building and deploying Vector into a cluster.

### Requirements

-   [`tilt`](https://tilt.dev/)
-   [`docker`](https://www.docker.com/)
-   [`kubectl`](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
-   [`minikube`](https://minikube.sigs.k8s.io/) (or any other Kubernetes cluster)

### Automatic Development Workflow (Tilt)

`tilt` automates the process of detecting code changes, rebuilding the Vector image, and updating Kubernetes resources.

1.  Start your local Kubernetes cluster (e.g., `minikube start`).
2.  Run `tilt up` from the root of the Vector repository.

## Testing

End-to-End (E2E) tests are crucial for Kubernetes integrations due to their complexity.

### Integration Tests (E2E)

-   **Execution**: E2E tests are primarily run in CI, but can be executed manually.
-   **Requirements**:
    -   Kubernetes cluster (e.g., `minikube`).
    -   `docker`.
    -   `kubectl`.
    -   `bash`.
    -   `cross` (`cargo install cross`).
    -   `helm`.
    -   Ability to prepare Vector release artifacts (see Vector documentation).
-   **Notes**:
    -   `minikube` versions `1.12.x` had bugs affecting Vector's tests; use `1.13.0+`.
    -   `minikube` may have issues on ZFS systems; consider cloud clusters or `microk8s`.
    -   E2E tests require sufficient resources (e.g., 8GB RAM, 2 CPUs).

### Tutorial: Running E2E Tests

```bash
CONTAINER_IMAGE_REPO=<your_docker_username>/vector-test make test-e2e-kubernetes
```

**Optional Parameters**:

-   `QUICK_BUILD=true`: Uses a development build and image, speeding up preparation (useful for test/code development).
-   `USE_MINIKUBE_CACHE=true`: Loads the built Docker image directly into a `minikube` cluster node, bypassing registry push. Defaults to `auto` based on `kubectl` context.
-   `CONTAINER_IMAGE=<your_name>/vector-test:tag`: Skips image building and uses a specified image.
-   `SKIP_CONTAINER_IMAGE_PUBLISHING=true`: Skips the image publishing step.
-   `SCOPE`: Passes a filter to `cargo test` to run specific tests.

**Example with `QUICK_BUILD` and `USE_MINIKUBE_CACHE`**:

```bash
QUICK_BUILD=true USE_MINIKUBE_CACHE=true make test-e2e-kubernetes
```
