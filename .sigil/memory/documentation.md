# Vector's Documentation Guidelines

Documentation is a first-class feature in the Vector project. All contributors are responsible for maintaining and updating documentation in conjunction with code changes.

## Responsibilities

### Contributors

When making code changes, contributors *MUST* update the following documentation:

-   **Reference Documentation**: Changes in the [`/website/cue`](../website/cue) directory, typically for configuration options.
-   **Existing Guides**: Updates to relevant guides in the [`/website/content`](../website/content) directory.
-   **Release Highlights**: If the change is significant, add an entry to [`/website/content/en/highlights`](../website/content/en/highlights) for future release notes.

Contributors are *NOT* typically responsible for:

-   Writing entirely new guides (unless assigned).
-   Writing blog posts (unless assigned).

### User Experience Committee

-   Responsible for the overall user experience and documentation quality.
-   Reviews and approves proposed user experience changes and user-facing documentation in pull requests.
-   Maintains and evolves guides to reflect new Vector features and best practices.

## Reference Documentation

Vector's [reference documentation](https://vector.dev/docs/reference) is a comprehensive index of all Vector features, especially configuration options. It is data-driven, generated from structured data defined in the [`/website/cue`](../website/cue) directory using [CUE](https://cuelang.org).

### Installing CUE

-   **Version**: Vector currently uses CUE `v0.7.0`. Using a different version may cause validation errors.
-   **Installation**: Install CUE from package managers or source. (Tracking issue for improved developer experience: [#15909](https://github.com/vectordotdev/vector/issues/15909)).

### Generating from Source Code

Much of the reference documentation is automatically compiled from Rust source code (e.g., doc comments).

-   **Command**: Run `make generate-docs` to regenerate this content.

### Adding Documentation for New Components

When adding a new source, sink, or transform:

1.  **Generate Base Documentation**: Run `make generate-component-docs`.
    -   This creates an auto-generated CUE file in `website/cue/reference/components/{sources,sinks,transforms}/generated/<component_name>.cue`.
    -   Doc comments in Rust code are used to populate this file.
2.  **Create Manual CUE File**: Create a new file at `website/cue/reference/components/{sources,sinks,transforms}/<component_name>.cue`.
    -   This file adds metadata like title, description, examples, feature classifications, and how-it-works sections.
    -   Refer to existing CUE files (e.g., `website/cue/reference/components/transforms/remap.cue`) for guidance.
3.  **Format CUE Files**: Run `./scripts/cue.sh fmt`.
4.  **Create Markdown Documentation File**: Create a new file at `website/content/en/docs/reference/configuration/{sources,sinks,transforms}/<component_name>.md`.
5.  **Verify Documentation**: Run `make check-generated-docs`.
6.  **Preview**: Use `cd website && make serve` to view how the documentation renders on the Vector website.

### Formatting CUE

-   **Tool**: CUE's autoformatter.
-   **Command**: Run `./scripts/cue.sh fmt` from the Vector root.
-   **CI**: CI checks ensure CUE sources are properly formatted.

### Validating CUE

-   **Command**: Run `CI=true make check-docs` from the Vector root.
-   **Purpose**: Ensures CUE sources conform to various schemas.
-   **Tips**: Make small, incremental changes and validate frequently. Use `watchexec
