# Vector's Rust Code Style and Conventions

This document outlines the Rust coding conventions and patterns enforced within the Vector project. Adhering to these guidelines ensures consistency, readability, and maintainability across the codebase.

## Formatting

-   **Tool**: `rustfmt` is used exclusively and comprehensively.
-   **Execution**: Run `make fmt` to format code. CI checks will fail if code is not formatted correctly.
-   **Configuration**: Vector uses its own `.rustfmt.toml` file, which is automatically picked up by `cargo fmt`.
-   **Macros**: `rustfmt` may sometimes struggle with code inside macros; manual adjustment might be necessary.

## Import Statements (`use`)

-   **Placement**: All `use` statements must be at the top of the file/module or at the top of `mod tests`.
-   **Ordering**: Group imports in the following order:
    1.  `std` (standard library)
    2.  External crates
    3.  Internal (`crate::`) modules
-   **Automation**: `rustfmt` automatically organizes imports according to these rules.

## Logging Style

Always use the [Tracing crate](https://tracing.rs/tracing/)'s key/value style for log events.

-   **Capitalization**: Messages must be capitalized and end with a period.
-   **Error Naming**: Always use `error` (not `e` or `err`) for error values to enrich logs.
-   **Display vs. Debug**: Prefer `Display` (`%error`) over `Debug` (`?error`) for formatting errors.
-   **Structured Logging**: Key/value pairs provide structured logging, which is crucial for observability.

**Correct Example:**

```rust
warn!(message =
