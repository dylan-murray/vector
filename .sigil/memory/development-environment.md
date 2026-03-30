# Vector's Development Environment Setup

This document outlines the recommended ways to set up a development environment for contributing to Vector, catering to different preferences and operating systems.

## Using a Docker or Podman Environment

This is the **recommended method** for most contributors, especially for first-timers or those who prefer a consistent, isolated environment. It mirrors the CI setup, ensuring reproducibility.

### Prerequisites

-   **Docker or Podman**: Install via your official package manager or from [Docker](https://docs.docker.com/get-docker/) or [Podman](https://podman.io/) websites.
-   **SELinux (Linux only)**: If running SELinux in Enforcing mode, relabel the Vector source code checkout with `container_home_t` context:

    ```bash
    cd your/checkout/of/vector/
sudo semanage fcontext -a "${PWD}(/.*)?" -t container_file_t
sudo restorecon . -R
    ```

### Usage

1.  **Optional: Build your own environment image** (if `ENVIRONMENT_AUTOBUILD` is not `true` or you want to ensure the latest build):

    ```bash
    make environment-prepare
    ```

2.  **Enter the development shell**:

    ```bash
    make environment
    # Or with Podman:
    make environment CONTAINER_TOOL="podman"
    # Or with extra CLI options (e.g., port publishing):
    make environment CLI_OPTS="--publish 3000:2000"
    ```

    This command launches a container with optimized mounts, providing a full Rust toolchain and necessary dependencies.

3.  **Run commands inside the environment**: Once inside the container, you can use `cargo` and `make` commands as if they were natively installed.

    ```bash
    # Example: Validate code can compile
    make check
    # Example: Run unit tests for a specific source
    make test SCOPE="sources::example"
    ```

4.  **Run commands from outside the environment**: You can also execute `make` targets directly from your host, which will automatically spin up the container, run the command, and tear it down.

    ```bash
    make check ENVIRONMENT=true
    make test-integration ENVIRONMENT=true
    ```

## Bring Your Own Toolbox

This method is required for MSVC/macOS/FreeBSD toolchains and can be used for any environment. It involves installing all development tools directly on your host system.

### Prerequisites

-   **Rust Toolchain**: `rustup` (stable channel).
-   **Build Tools**:
    -   `protoc` (Protobuf compiler)
    -   C++/C build tools (LLVM, GCC, or MSVC, depending on your OS).
    -   `make` (GNU Make preferred).
    -   `bash`.
    -   `cmake`.
    -   `GNU coreutils`.
    -   `autotools`.
-   **Libraries**:
    -   `libsasl2`: Required for `kafka`-related features.
        -   **Ubuntu**: `sudo apt-get install -y libsasl2-dev`
        -   **macOS**: `brew install cyrus-sasl` (and set `LDFLAGS`, `CPPFLAGS`, `PKG_CONFIG_PATH` environment variables).
-   **Rust Utilities**:
    -   `cargo-nextest`: For running tests (`cargo install cargo-nextest`).
    -   `remarshal`: For `make check-component-features`.
    -   `dd-rust-license-tool`: For `make check-licenses` or `make build-licenses`.
    -   `cue`: For `make generate-docs`.

### Common Development Commands

-   **Check Code**: `cargo check` or `make check`
-   **Build (Dev Mode)**: `cargo build` or `make build-dev`
-   **Run Unit Tests**: `cargo test` or `make test`
-   **Run Specific Unit Tests**: `cargo test sources::example` or `make test SCOPE="sources::example"`
-   **Run Integration Tests**: `make test-integration` (starts Docker services) or `make test-integration AUTOSPAWN=false` (for live services).
-   **Run Benchmarks**: `cargo bench transforms::example` or `make bench SCOPE="transforms::example"`
-   **Format Code**: `cargo fmt` or `make fmt`
-   **Generate Documentation**: `make generate-docs`

## Makefile

The root `Makefile` provides a high-level interface for common development tasks. Run `make help` to see a list of available targets and their descriptions.
