# Validation And Refactor Gates

Use this sequence for every slice.

## Phase 1: Feature Validation

Run the repo-owned automated checks immediately after feature implementation reaches the planned acceptance bar.

Typical checks may include:

- `cargo fmt --check`
- `cargo clippy --all-targets --all-features -- -D warnings`
- `cargo test`
- repo-specific integration or UI checks

Use the repo's configured validation commands when they exist. Do not invent lighter checks if the repo already defines the expected validation path.

## Phase 2: Mandatory Rust Refactor

After feature validation passes, perform a behavior-preserving refactor across all modules in every touched Rust repo.

Focus on:

- clearer type and module boundaries
- simpler control flow and error handling
- removal of obvious duplication
- Rust-idiomatic ownership and borrowing patterns
- trait, enum, and newtype usage where already justified by the design
- public API documentation comments for new or changed public items

Do not:

- widen the slice scope to untouched repos
- change externally visible behavior without moving that work back into feature scope
- rewrite pre-existing tests without approval

## Test Editing Rule

- Freely update tests created during the current slice.
- Do not edit tests that existed before the slice without explicit approval.
- If a pre-slice test blocks the refactor, stop and ask before changing it.

## Phase 3: Post-Refactor Validation

Run the automated checks again after refactoring. Treat this as a separate required gate, not a rerun only when convenient.

Close the coding portion of the slice only when both automated phases pass or the user explicitly accepts a documented exception.
