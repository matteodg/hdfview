# Feature Specification: Migrate HDF5 Java access from legacy jarhdf5 to FFM binding

**Feature Branch**: `003-migrate-hdf5-ffm`  
**Created**: 2026-04-26  
**Status**: Draft  
**Input**: User description: "replace all usages of the hdf5 library from jarhdf5 to hdf5-java-ffm"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - HDF5 files behave as before for end users (Priority: P1)

An HDFView user opens typical HDF5 files (groups, datasets, attributes, common datatypes), navigates the tree, previews data, and performs supported edits. They notice no loss of previously supported HDF5 capabilities that the application already exposed, and no new blocking errors for workflows that worked before the change.

**Why this priority**: End-user trust and release safety depend on functional parity for core HDF5 viewing and editing paths.

**Independent Test**: Using a fixed set of representative HDF5 sample files (including files already used for regression), a tester records pass/fail for a short checklist of open, browse, subset view, and save operations before and after the migration; after migration, the same checklist outcomes are **no worse** than the baseline.

**Acceptance Scenarios**:

1. **Given** a sample HDF5 file that opened successfully before migration, **When** the user opens it after migration, **Then** the file opens without error and the object tree is complete and navigable.
2. **Given** a dataset the user could preview before migration, **When** they preview the same selection after migration, **Then** values render correctly for the same slice and datatype cases covered by the checklist.
3. **Given** an edit the application already supported on HDF5 content before migration, **When** the user repeats that edit after migration, **Then** the change persists and re-opens consistently.

---

### User Story 2 - Build and packaging no longer rely on the legacy binding (Priority: P1)

A maintainer or CI job produces a standard project build and distributable output. The build’s declared supply chain for HDF5 Java access no longer pulls or names the legacy **jarhdf5** line; it consistently resolves the HDF Group’s **FFM-based** Java binding (**hdf5-java-ffm**) together with the matching native HDF5 artifacts the project already standardizes on.

**Why this priority**: Incorrect or duplicate bindings are a primary source of runtime crashes and ambiguous support; a single, explicit binding line is required for reproducible builds.

**Independent Test**: From a clean resolver state (or documented mirror), dependency and packaging manifests are inspected: there is **no** remaining reference to the legacy jarhdf5 artifact or module identity, and the FFM binding appears wherever HDF5 Java access is required for the application and its modules under test.

**Acceptance Scenarios**:

1. **Given** a clean dependency resolution for the full application build, **When** the maintainer lists resolved HDF5-related Java components, **Then** the legacy jarhdf5 coordinate does not appear and the FFM binding coordinate the project standardizes on does appear for every module that needs HDF5 Java access.
2. **Given** packaged runtime layout rules for the application, **When** the build completes, **Then** the running application does not load two competing Java HDF5 bindings for the same role.

---

### User Story 3 - Contributors follow one documented HDF5 Java path (Priority: P2)

A new contributor reads setup and troubleshooting documentation to enable HDF5 in HDFView. Instructions describe obtaining and aligning **native HDF5** with the **FFM Java binding** only; they do not send the reader to the legacy jarhdf5 path.

**Why this priority**: Documentation drift causes misconfigured environments and false bug reports.

**Independent Test**: A documentation reviewer searches contributor-facing docs for legacy binding names; expected hits are **zero** except where historical context is explicitly labeled as archived or superseded.

**Acceptance Scenarios**:

1. **Given** the primary contributor quickstart for this repository, **When** a developer follows it on a supported platform, **Then** they can run the application with HDF5 enabled without steps that mention jarhdf5 as the active binding.
2. **Given** troubleshooting for “HDF5 failed to load,” **When** the user reads the doc, **Then** remediation steps reference the FFM binding and matching native layout, not the legacy binding.

---

### Edge Cases

- **Partial file features**: Operations that depended on jarhdf5-specific quirks must either work through the FFM binding or fail with a clear, user-visible message (no silent wrong data).
- **Platform matrix**: Platforms still on legacy-only paths today must either be brought onto the FFM path in this delivery or be explicitly scoped out with documented user impact (no contradictory “FFM everywhere” claim where legacy remains).
- **Native mismatch**: Wrong or missing native HDF5 libraries must surface as a diagnosable configuration error, not an unexplained process exit.
- **Automated tests**: Tests that assumed the legacy binding’s internals must be updated so the suite reflects the FFM binding without disabling coverage unless an operation is genuinely unsupported.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The product MUST remove all build-time and runtime dependency declarations that identify the legacy **jarhdf5** Java HDF5 binding as the means of HDF5 access for HDFView and its companion modules in scope of this change.
- **FR-002**: The product MUST use the HDF Group’s **hdf5-java-ffm** (FFM-based) Java binding for all HDF5 Java access paths previously implemented through jarhdf5, including UI flows, object layer behavior, and tests that exercise HDF5.
- **FR-003**: For each automated test scenario that covered HDF5 behavior before migration, the same scenario MUST remain executable after migration and preserve expected assertions, except where a capability is documented as intentionally removed or unsupported with a clear rationale.
- **FR-004**: The product MUST avoid loading two different Java HDF5 bindings for the same functional role in the same process (no duplicate or conflicting binding in the assembled deliverable).
- **FR-005**: Contributor-facing documentation MUST be updated so that setup, build, and troubleshooting instructions for HDF5 Java access refer to the FFM binding and aligned native artifacts, not jarhdf5, except in explicitly marked historical notes.
- **FR-006**: Where HDF5 operations fail due to environment (missing native libraries, version skew), the product MUST present actionable messaging suitable for end users or contributors (what is missing or misaligned), consistent with existing application error-handling patterns.

### Key Entities

- **HDF5 file**: User data container (groups, datasets, attributes, links) accessed through the application.
- **Java HDF5 binding**: The Java-side library that connects application code to native HDF5; in scope, this migrates from the legacy jarhdf5 line to **hdf5-java-ffm**.
- **Native HDF5 libraries**: Platform binaries required by the Java binding; must stay version- and layout-compatible with what the FFM binding expects.
- **Build resolution**: Declared coordinates and classifiers that produce the runtime dependency graph for the application and its tests.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: **100%** of HDF5-focused automated regression scenarios that passed immediately before migration **pass** after migration, or each failure is tracked with an explicit, reviewed waiver tied to a documented capability gap (no silent downgrades).
- **SC-002**: **100%** of in-scope deliverables that previously declared jarhdf5 for HDF5 Java access declare **hdf5-java-ffm** instead, with **zero** remaining jarhdf5 declarations in those scopes (verifiable by an auditable review of declared third-party HDF5 Java components).
- **SC-003**: On a supported developer configuration, a new contributor can complete the documented setup and launch the application with HDF5 enabled in **under 30 minutes** when following only updated instructions (same target as prior contributor quickstart expectations).
- **SC-004**: For a fixed panel of **at least five** representative HDF5 sample files spanning common datatypes and nesting, **100%** complete the open-and-browse checklist in User Story 1 without new blocking defects attributable to the binding swap.

## Assumptions

- **Full migration**: “Replace all usages” means a complete cutover from jarhdf5 to **hdf5-java-ffm** for HDF5 Java access in this repository’s application and the parts of the codebase that ship with it, not a side-by-side transitional compatibility shim unless a shim is explicitly added later under a separate change.
- **Native alignment**: Native HDF5 artifacts and paths remain governed by the same org.hdfgroup versioning and platform-classifier strategy already adopted for this project (e.g., work reflected in existing HDF5 dependency specifications); this feature changes the **Java binding**, not the business goal of official native supply.
- **Naming**: “jarhdf5” and “hdf5-java-ffm” are used in this specification as the recognizable product and artifact names stakeholders already use; planning may translate these into concrete module coordinates and code entry points without changing the intent.
- **Out of scope**: New HDF5 features not present before migration, and HDF4/NetCDF paths, unless they share code paths that must compile against the new binding—only what is required for a clean HDF5 binding swap is in scope.
