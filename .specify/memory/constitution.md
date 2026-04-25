<!--
Sync Impact Report
- Version change: N/A → 1.0.0
- Modified principles: N/A (template initialization)
- Added sections: Engineering Standards, Development Workflow
- Removed sections: None
- Templates requiring updates:
  - ✅ d:\c\hdfview\.specify\templates\tasks-template.md (align test guidance with constitution)
  - ✅ d:\c\hdfview\.specify\templates\plan-template.md (no change needed)
  - ✅ d:\c\hdfview\.specify\templates\spec-template.md (no change needed)
  - ✅ d:\c\hdfview\.specify\templates\constitution-template.md (no change needed)
- Follow-up TODOs (deferred placeholders):
  - TODO(RATIFICATION_DATE): original adoption date not known; fill once established by maintainers.
-->

# HDFView Constitution

## Core Principles

### I. Build & Tooling Consistency (Maven-only, Java 21)
All changes MUST keep the repository buildable with Maven and Java 21.

- Build and packaging MUST use Maven (Ant is not supported).
- Changes MUST respect the existing multi-module structure (`repository/`, `object/`, `hdfview/`).
- Platform-specific behavior (SWT, native libs, packaging) MUST be considered; avoid assumptions that
  only hold on one OS.

### II. Native Library Safety (no JVM-crash regressions)
HDF4/HDF5 native interactions are high-risk. Any change that touches native calls, datatype handling,
or library loading MUST prioritize crash safety and graceful degradation.

- Prefer defensive checks and clear error paths over “best effort” calls into native code.
- Avoid introducing behavior that can crash the JVM (SIGSEGV, fatal errors) on invalid/unsupported
  operations.
- If platform-specific library paths or plugins are involved, ensure failures produce actionable
  messages and do not corrupt user data.

### III. Tests Where They Matter (pragmatic, CI-friendly)
Changes to **non-UI logic** (e.g., `object/` module, data handling, file format behavior) MUST be
covered by automated tests when feasible.

- Unit/integration tests SHOULD be added for regressions and new behavior in non-UI code.
- UI tests MAY be added but MUST account for real-display constraints; do not block delivery on UI
  automation that cannot run in CI.
- Test additions MUST be stable and not require proprietary environments to run.

### IV. Minimal, Focused Changes (avoid over-engineering)
Prefer small, reviewable diffs that match established patterns in the codebase.

- Refactors MUST be justified by a specific problem (bug, maintenance pain, correctness, safety).
- Avoid introducing new frameworks or architectural layers without clear benefit.
- Keep public behavior stable unless the change is intentional and documented.

### V. Quality Gates & Documentation for Change
Changes MUST maintain the project’s quality and be understandable to maintainers.

- Do not introduce linter/formatting/style violations.
- Update docs when behavior, packaging, build steps, or developer workflows change.
- For user-impacting changes, add a clear test plan (how to validate locally).

## Engineering Standards

- **Language/Runtime**: Java 21.
- **Build System**: Maven (multi-module).
- **UI**: SWT/NatTable (platform differences are expected; validate on target platforms when
  applicable).
- **Security & Supply Chain**: Dependency additions MUST be justified; avoid unnecessary new
  transitive dependencies.
- **Performance**: Avoid regressions in common file-open/browse flows; do not add expensive work on
  UI thread without justification.

## Development Workflow

- Prefer repo-provided scripts when applicable (e.g., `scripts/build-dev.sh`,
  `scripts/clean-all.sh`, `run-hdfview.{sh,bat}`).
- Keep commits reviewable; each change SHOULD have a clear purpose and rollback path.
- When touching native/library loading, validate both “happy path” and “missing library/plugin”
  scenarios.

## Governance

- This constitution supersedes template guidance when they conflict.
- Amendments MUST:
  - Update the **Sync Impact Report** at the top of this file
  - Increment version using semantic versioning:
    - **MAJOR**: backwards-incompatible governance/principle removals or redefinitions
    - **MINOR**: new principle/section or material expansion of requirements
    - **PATCH**: clarifications, wording fixes, non-semantic refinements
  - Update `LAST_AMENDED_DATE` (YYYY-MM-DD)
- Reviews SHOULD explicitly check compliance for changes that touch:
  - Native library interactions
  - Build/packaging
  - User data integrity
  - Cross-platform SWT behavior

Guidance references: `CLAUDE.md` (developer workflow and build/test notes).

**Version**: 1.0.0 | **Ratified**: TODO(RATIFICATION_DATE) | **Last Amended**: 2026-04-26
