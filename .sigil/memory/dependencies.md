# Vector's Dependency Management

This document outlines the guidelines and considerations for managing external dependencies within the Vector project.

## Dependency Selection Criteria

Dependencies are carefully selected to maintain project quality, performance, and security. Before adding a new dependency, consider the following questions:

1.  **Cost-Benefit Analysis**: Is the dependency's functionality worth its overhead (e.g., increased binary size, compilation time, maintenance burden)?
2.  **Maintenance Status**: Is the dependency actively and professionally maintained? Avoid unmaintained or experimental crates for core functionality.
3.  **Maturity**: Is the dependency experimental or in an early development phase? Prefer stable and widely adopted solutions.
4.  **Community Support**: How large and active is the dependency's community? A larger community often implies better support and faster bug fixes.
5.  **Security History**: Does the dependency have a history of security vulnerabilities? Regularly check `cargo deny` advisories.
6.  **Portability**: Will this dependency affect the portability of Vector across different operating systems and architectures (e.g., Linux, Windows, macOS, ARM, x86_64)?
7.  **License Compatibility**: Does the dependency have a license compatible with Vector's MPL-2.0 license? (See `LICENSE-3rdparty.csv` and `deny.toml`).

## Implementation Guidelines

-   **Feature Flags**: When a dependency is only required by one or multiple components (sources, transforms, sinks) and not by Vector's core, it *MUST* be made optional and added to the corresponding feature flags in `Cargo.toml`.
    -   This allows users to build customized Vector binaries with only the necessary components, reducing binary size and compilation time.
-   **`default-features = false`**: Always set `default-features = false` in `[workspace.dependencies]` for all dependencies. This gives explicit control over which features are enabled.
-   **Explicit Feature Declaration**: For feature declarations:
    -   If only one crate needs a feature, declare it in that crate's own `Cargo.toml`.
    -   If multiple crates need a feature, declare it in `[workspace.dependencies]`.
-   **Comments**: Add short comments near the dependency declaration in `Cargo.toml` when the feature setup is non-obvious or requires specific context.

## Auditing Dependencies

-   **`cargo deny`**: Vector uses `cargo deny` to check for security vulnerabilities and license incompatibilities. This check runs on every pull request.
    -   Configuration is in `deny.toml`.
    -   Ignored advisories are listed in `deny.toml` with justifications.
-   **`dd-rust-license-tool`**: Used to generate and update the `LICENSE-3rdparty.csv` file, which inventories all third-party licenses.
    -   Run `make build-licenses` to regenerate this file.
-   **`Dependabot`**: Configured in `.github/dependabot.yml` to automate dependency updates and alert on security vulnerabilities.
-   **`cargo tree`**: Use `cargo tree -e features` to inspect the full feature tree of the workspace or `cargo tree -e features -i <crate-name>` to narrow down to a specific dependency. This helps understand which features are enabled and by which crates.

## MSRV (Minimum Supported Rust Version)

-   Vector's MSRV is specified in `Cargo.toml` (`rust-version`).
-   The MSRV can be bumped at any time if required by a dependency or to leverage new Rust language features.
