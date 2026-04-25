# Feature Specification: JDK 25 baseline for FFM-ready builds

**Feature Branch**: `001-jdk-25-ffm`  
**Created**: 2026-04-26  
**Status**: Draft  
**Input**: User description: "I'd like to first use the JDK 25 as it has FFM API"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Maintainer adopts JDK 25 for the build (Priority: P1)

A release or build maintainer updates the project so that compiling, testing, and packaging HDFView
use **JDK 25** as the supported minimum Java baseline. The goal is to unlock the **Foreign Function
and Memory (FFM)** API for safer, more maintainable native interoperability work in follow-on
changes, without changing end-user visible behavior in this feature alone.

**Why this priority**: Without a JDK 25 baseline, no downstream FFM-based native work can land
reliably; the build must be truthful about the Java version the team intends to use first.

**Independent Test**: On a clean machine with JDK 25 and required native HDF dependencies installed,
follow published setup steps and confirm the project completes the same primary build and test
entry points that existed before this change (no new silent skips).

**Acceptance Scenarios**:

1. **Given** JDK 25 is the only JDK on the PATH, **When** the maintainer runs the documented build
   command, **Then** the build completes successfully using JDK 25.
2. **Given** a contributor uses JDK 24 or older, **When** they run the build, **Then** they receive
   a clear, actionable message that JDK 25 is required (instead of obscure compiler errors).

---

### User Story 2 - CI reflects the new baseline (Priority: P2)

Continuous integration runs on JDK 25 so regressions in compatibility with the new baseline are
caught automatically.

**Why this priority**: CI is the enforcement mechanism for the baseline; local-only updates drift.

**Independent Test**: Inspect CI configuration and a green CI run (or dry-run matrix) showing JDK
25 used for the primary build/test job set.

**Acceptance Scenarios**:

1. **Given** the default CI pipeline for the repository, **When** it runs for a normal change,
   **Then** the Java jobs use JDK 25 (or a documented equivalent pinned distribution) for compile
   and test.
2. **Given** a change that breaks JDK 25 compilation, **When** CI runs, **Then** the failure
   surfaces in the Java build job with logs sufficient to diagnose the issue.

---

### User Story 3 - Contributors can install and verify JDK 25 (Priority: P3)

A new contributor can install JDK 25, verify their environment, and match the project’s
documented expectations before spending time on code changes.

**Why this priority**: Reduces support churn and avoids “works on my machine” due to JDK mismatch.

**Independent Test**: A contributor follows only the docs updated in this feature and successfully
prints or records the same major Java version the project requires.

**Acceptance Scenarios**:

1. **Given** the contributor documentation, **When** they follow the JDK section, **Then** they
   can confirm they are on the required major version with one command.
2. **Given** Windows, Linux, and macOS are supported developer platforms for this repo, **When** a
   contributor reads the docs, **Then** they see platform-appropriate pointers (or explicitly
   scoped limitations) for installing JDK 25.

### Edge Cases

- Mixed JDK installs on one machine (PATH/JAVA_HOME points to wrong JDK).
- Contributors who must stay on an older JDK for unrelated work: docs should state the project
  baseline is JDK 25 and older JDKs are unsupported for this repository.
- Native HDF libraries and JVM flags: baseline JDK change must not remove required runtime flags
  without an explicit follow-up decision.
- Packaging/installers (jpackage): verify the JDK used for packaging aligns with the new baseline
  where applicable.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The project MUST declare **JDK 25** as the minimum supported Java version for
  development, compile, and automated test execution.
- **FR-002**: Build configuration MUST fail fast (or otherwise clearly signal) when the active JDK
  is below 25.
- **FR-003**: Primary CI workflows MUST run compile/test using JDK 25 (or a documented pinned
  equivalent that implements the same Java SE 25 language/runtime level required by the project).
- **FR-004**: Contributor-facing documentation MUST be updated so a new developer can install JDK
  25 and validate their environment consistently.
- **FR-005**: The change set MUST preserve existing automated quality gates at the same level of
  coverage as before (no new broad suppression of tests or checks without a recorded exception and
  owner follow-up).

### Key Entities

- **Java baseline policy**: The single declared minimum JDK major version and how it is enforced
  (build + CI + docs).
- **Build entrypoints**: The documented commands contributors and CI use to compile and test.
- **CI job definitions**: The workflows or jobs that select the JDK for build and test.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: On the reference CI configuration, **100%** of Java compile-and-test jobs that ran
  before the baseline change still run after it, with **JDK 25** as the selected runtime for those
  jobs.
- **SC-002**: **100%** of sampled setup paths in updated contributor documentation report the same
  required major Java version when the verification command is run (no contradictory instructions
  across README, build docs, and agent guidance where they overlap).
- **SC-003**: A maintainer can complete the documented “verify environment” steps in **10 minutes or
  less** on a machine that already has HDF native prerequisites installed (time excludes first-time
  HDF library installation).

## Assumptions

- “First use JDK 25” means **raising the minimum baseline** to 25 for this codebase, not merely
  adding an optional secondary toolchain while keeping 21 as the default.
- **FFM API availability** is the motivation for choosing 25; this specification does **not**
  require migrating existing JNI/native glue to FFM in the same delivery—only establishing the
  JDK baseline that makes FFM work possible next.
- End-user installers and platform packaging continue to follow existing project conventions; only
  documentation or configuration updates required for consistency are in scope unless a concrete
  breakage is discovered during implementation planning.
