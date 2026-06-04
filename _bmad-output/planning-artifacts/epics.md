---
stepsCompleted: [1, 2, 3]
inputDocuments:
  - planning-artifacts/prds/prd-hdfview-2026-06-04/prd.md
  - planning-artifacts/prds/prd-hdfview-2026-06-04/addendum.md
  - planning-artifacts/prds/prd-hdfview-2026-06-04/.decision-log.md
  - planning-artifacts/architecture.md
  - project-context.md
  - CLAUDE.md
  - docs/Testing-Guide.md
---

# hdfview - Epic Breakdown

## Overview

Epic and story breakdown for **HDFView JDK 25 and HDF5 FFM migration**, decomposing PRD FR-1–FR-13 and Architecture decisions D-1–D-24 into implementable work. Primary user: **HDFView maintainers** (no end-user feature epics).

## Requirements Inventory

### Functional Requirements

FR-1: Build all modules with `maven.compiler.release` **25** (Phase 1; `repository` may remain).

FR-2: Run `mvn test` on JDK 25 without plugin failures; document Surefire JVM args.

FR-3: Launch via project scripts and build **jpackage** app-images with **JDK 25** on Windows x86_64.

FR-4: Phase 1 behavioral parity — HDF5/HDF4 smoke, object `@Tag("unit")`/`fast`, mandatory `@Tag("ui")` (FR-13).

FR-5: Declare `org.hdfgroup` **2.2.0** Maven dependencies with `windows-x86_64` classifiers (Phase 2).

FR-6: Remove **`repository`** module and `jarhdf5` install-file goals for HDF5 (Phase 2).

FR-7: **Big-bang** replace `hdf.hdf5lib` with `javahdf5` / `hdf5-java-ffm` (Phase 2).

FR-8: HDF4 remains **JNI** via `hdf.lib.dir` (Phase 2).

FR-9: Remove `hdf5.lib.dir` and `hdf5.plugin.dir` from build configuration (Phase 2).

FR-10: Phase 2 behavioral parity — object HDF5 tests + mandatory UI suite on Windows.

FR-11: Update maintainer docs (`CLAUDE.md`, `project-context.md`, launchers) per phase.

FR-12: GitHub Actions — Phase 1 all OS on JDK 25; Phase 2 Windows FFM + scoped macOS/Linux jobs.

FR-13: Mandatory `@Tag("ui")` SWTBot tests as phase completion gate (Windows sign-off).

### NonFunctional Requirements

NFR-1: **Parity** — no user-visible regression in open/browse/edit/save (HDF5 + HDF4 on Windows).

NFR-2: **Reliability** — prefer build-time / Maven resolution failure over JVM native crash.

NFR-3: **Maintainability** — single `hdf5.stack.version=2.2.0` BOM property (Phase 2); Maven-native HDF5.

NFR-4: **Testability** — phase gates require object tests + Windows UI tests; CI policy documented.

NFR-5: **Quality gates** — JaCoCo, Checkstyle, PMD remain valid on JDK 25 (pass or documented skip).

NFR-6 (SM-3): No P1/P2 migration regressions post-Phase 2 (or N/A if not externally shipped).

NFR-7 (SM-C1): Minimal diff scope — do not optimize LOC changed.

### Additional Requirements

- **Brownfield** — no greenfield starter; HDFView 3.4.1 codebase retained (Architecture § Foundation).
- **Phased delivery** — Phase 1 complete before Phase 2 starts (D-1).
- **Phase 1:** Keep JNI/`jarhdf5`/`repository`; bump JDK 25; fix `build-dev.sh` JAR names; quote launcher JVM paths (D-13, D-16).
- **Phase 1 CI:** `ci-windows.yml`, `ci-linux.yml`, `ci-macos.yml`, orchestrator → JDK 25; **Windows runs UI tests**; Linux/macOS object tests (D-18, D-19).
- **Phase 1 jpackage** on JDK 25 (D-23).
- **Phase 2 spike** before big-bang: validate `javahdf5` for Float16/BFLOAT16, vlen, refs, plugins (Architecture gap).
- **Phase 2:** BOM `hdf5.stack.version`; `windows-x86_64` profile; delete `repository/`; migrate `object/.../h5/` (D-4–D-10).
- **Phase 2 JVM:** FFM native access + `--enable-native-access=ALL-UNNAMED` for SWT (D-15).
- **Phase 2 CI:** Windows full FFM + UI; macOS/Linux **object-only**, no false-green HDF5 (D-20, D-21).
- **Audit** `build-windows.yml`, `publish-maven-packages.yml` for stale `jarhdf5` references (Architecture gap).
- **Agent patterns:** Phase-tagged PRs; no `hdf.hdf5lib` after Phase 2; update `project-context.md` when build contract changes.

### UX Design Requirements

_None — no UX spec; migration explicitly has no user-visible change._

### FR Coverage Map

| FR | Epic | Notes |
|----|------|-------|
| FR-1 | Epic 1 | JDK 25 POM |
| FR-2 | Epic 1 | Surefire / plugins |
| FR-3 | Epic 1 | Launchers + jpackage |
| FR-4 | Epic 1 | Phase 1 parity + UI gate |
| FR-5 | Epic 2 | org.hdfgroup deps |
| FR-6 | Epic 2 | Remove repository |
| FR-7 | Epic 2 | Big-bang API |
| FR-8 | Epic 2 | HDF4 JNI retained |
| FR-9 | Epic 2 | Drop HDF5 build.properties paths |
| FR-10 | Epic 2 | Phase 2 parity + UI gate |
| FR-11 | Epic 1 & 2 | Docs each phase |
| FR-12 | Epic 1 & 2 | CI per phase |
| FR-13 | Epic 1 & 2 | Mandatory UI tests |

## Epic List

### Epic 1: JDK 25 Platform Readiness

**Maintainer outcome:** HDFView builds, tests, packages, and runs on **JDK 25** with the **existing JNI HDF5 stack**, all primary CI platforms on JDK 25, and **mandatory Windows UI test** sign-off — with **no user-visible behavior change**.

**FRs covered:** FR-1, FR-2, FR-3, FR-4, FR-11 (Phase 1), FR-12 (Phase 1), FR-13 (Phase 1)

**Dependencies:** None (first epic).

---

### Epic 2: HDF5 FFM on Windows (Maven-native)

**Maintainer outcome:** HDF5 uses **`org.hdfgroup` 2.2.0 FFM** on **Windows x86_64**, **`repository`/`jarhdf5` removed**, **`hdf5.lib.dir` dropped**, **HDF4 JNI unchanged**, big-bang API migration complete, Windows CI + UI parity sign-off.

**FRs covered:** FR-5, FR-6, FR-7, FR-8, FR-9, FR-10, FR-11 (Phase 2), FR-12 (Phase 2), FR-13 (Phase 2)

**Dependencies:** Epic 1 must be complete and merged.

---

## Epic 1: JDK 25 Platform Readiness

HDFView builds, tests, packages, and runs on **JDK 25** with existing JNI HDF5; CI on JDK 25; mandatory Windows UI tests; no user-visible change.

### Story 1.1: Set JDK 25 as Maven compiler baseline

As an **HDFView maintainer**,
I want the project to compile with **Java 25** across all modules,
So that we align the toolchain before any binding changes.

**Acceptance Criteria:**

**Given** the root BOM and module POMs  
**When** `maven.compiler.release` is set to **25** and `mvn clean package -DskipTests` runs on Windows x86_64 with JDK 25  
**Then** `repository`, `object`, and `hdfview` compile successfully  
**And** `CLAUDE.md` states JDK **25** as the minimum (FR-1)

---

### Story 1.2: Validate tests and quality plugins on JDK 25

As an **HDFView maintainer**,
I want unit tests and quality plugins to run on JDK 25,
So that CI and local dev use the same JVM rules.

**Acceptance Criteria:**

**Given** Story 1.1 is complete  
**When** `mvn test -pl object` runs with documented Surefire `--add-opens` and `--enable-native-access=jarhdf5`  
**Then** object `@Tag("unit")` and `@Tag("fast")` tests pass  
**And** PMD, Checkstyle, and JaCoCo either pass or have documented skips with follow-up issues (FR-2)

---

### Story 1.3: Fix launchers and development build scripts for JDK 25

As an **HDFView maintainer**,
I want launchers and `build-dev.sh` to work on Windows with JDK 25,
So that local run/debug matches CI.

**Acceptance Criteria:**

**Given** Story 1.1 is complete  
**When** `run-hdfview.bat` launches HDFView with quoted `-Djava.library.path` for paths containing spaces  
**Then** the main window opens and HDF5/HDF4 samples load via existing JNI  
**And** `scripts/build-dev.sh` installs `object-3.4.1.jar` (not SNAPSHOT) to `~/.m2` (FR-3, D-13)

---

### Story 1.4: jpackage app-image on JDK 25

As an **HDFView maintainer**,
I want jpackage to produce an app-image using JDK 25,
So that packaging matches the new runtime.

**Acceptance Criteria:**

**Given** Story 1.3 is complete  
**When** `mvn verify -Pjpackage-app-image -pl object,hdfview -DskipTests` runs on Windows x86_64  
**Then** the app-image build completes without JDK version errors  
**And** the bundled/runtime configuration documents JDK 25 (FR-3, SM-5)

---

### Story 1.5: Upgrade GitHub Actions to JDK 25 (all platforms)

As an **HDFView maintainer**,
I want Windows, macOS, and Linux CI jobs on JDK 25,
So that remote builds match local development.

**Acceptance Criteria:**

**Given** Story 1.2 is complete  
**When** `ci-windows.yml`, `ci-linux.yml`, `ci-macos.yml`, and related orchestrator workflows use JDK **25**  
**Then** compile and **object-module** tests pass on all three OS runners  
**And** `build-windows.yml` / `publish-maven-packages.yml` are audited for stale JDK 21 assumptions only (no Phase 2 FFM yet) (FR-12 Phase 1, D-18)

---

### Story 1.6: Enable mandatory Windows UI tests and update Testing Guide

As an **HDFView maintainer**,
I want Windows CI to run mandatory UI tests and accurate test docs,
So that Phase 1 sign-off matches FR-13.

**Acceptance Criteria:**

**Given** Story 1.5 is complete  
**When** the Windows CI job runs `mvn test -pl hdfview` with `@Tag("ui")` on a display-capable runner  
**Then** the mandatory UI suite passes (or failing tests are fixed or explicitly waived with issue links)  
**And** `docs/Testing-Guide.md` documents JDK **25**, Windows UI CI policy, and that Xvfb is insufficient for SWT (FR-13, FR-4, D-19, D-22)

---

### Story 1.7: Phase 1 parity sign-off and documentation

As an **HDFView maintainer**,
I want a recorded Phase 1 sign-off with updated agent docs,
So that Epic 2 can start from a known-good baseline.

**Acceptance Criteria:**

**Given** Stories 1.1–1.6 are complete  
**When** UJ-1 smoke is executed (HDF5 + HDF4 open/edit/save, jpackage smoke, UI tests green)  
**Then** no new user-visible regressions are observed vs pre-migration baseline  
**And** `project-context.md` reflects JDK 25 and Phase 1 CI policy; Phase 1 section of FR-11 is done (FR-4, FR-11)

---

## Epic 2: HDF5 FFM on Windows (Maven-native)

HDF5 via **org.hdfgroup 2.2.0** FFM on Windows x86_64; `repository` removed; HDF4 JNI retained.

### Story 2.1: Spike javahdf5 API coverage

As an **HDFView maintainer**,
I want a documented API mapping from `hdf.hdf5lib` to `javahdf5`,
So that the big-bang migration has no surprise gaps.

**Acceptance Criteria:**

**Given** `org.hdfgroup` 2.2.0 artifacts are available locally  
**When** a spike covers Float16/BFLOAT16, vlen, compound, refs, and plugin paths used by HDFView  
**Then** a short report lists mapped APIs, gaps, and mitigations  
**And** Epic 2 implementation proceeds only if gaps are accepted or resolved (Architecture spike; blocks 2.4)

---

### Story 2.2: Add org.hdfgroup 2.2.0 BOM and Windows dependencies

As an **HDFView maintainer**,
I want HDF5 FFM artifacts declared in Maven,
So that natives resolve without `repository/lib`.

**Acceptance Criteria:**

**Given** Story 2.1 gap report is accepted  
**When** root BOM defines `hdf5.stack.version=2.2.0` and `object/pom.xml` declares `hdf5-native`, `hdf5-szip-native`, `hdf5-zlib-native`, `hdf5-java-ffm` (`windows-x86_64`), and `javahdf5` per addendum  
**Then** `mvn dependency:resolve -pl object` succeeds on Windows x86_64  
**And** missing classifier fails at resolution with a clear Maven error (FR-5, D-10)

---

### Story 2.3: Remove repository module and jarhdf5 bootstrap

As an **HDFView maintainer**,
I want the `repository` module removed from the build,
So that HDF5 no longer depends on manual jar install.

**Acceptance Criteria:**

**Given** Story 2.2 is complete  
**When** `repository` is removed from root `<modules>` and all `jarhdf5` install-file / copy goals are deleted  
**Then** `mvn clean package -DskipTests` succeeds with modules `object` → `hdfview` only  
**And** docs no longer instruct building `repository` first (FR-6, D-7)

---

### Story 2.4: Big-bang migrate hdf.object.h5 to javahdf5

As an **HDFView maintainer**,
I want all HDF5 code in `object` to use javahdf5/FFM,
So that JNI is eliminated for HDF5.

**Acceptance Criteria:**

**Given** Stories 2.2–2.3 are complete  
**When** `object/src/main/java/hdf/object/h5/` and related tests compile with no `hdf.hdf5lib` imports  
**Then** `mvn test -pl object` passes on Windows x86_64  
**And** grep for `hdf.hdf5lib` under `object/` returns zero matches (FR-7)

---

### Story 2.5: Update hdfview HDF5 references and JVM native access

As an **HDFView maintainer**,
I want the GUI module to compile against the new HDF5 stack,
So that the desktop app uses FFM end-to-end.

**Acceptance Criteria:**

**Given** Story 2.4 is complete  
**When** remaining `hdfview` HDF5 imports are updated and Surefire/launcher JVM args use FFM + SWT native-access rules (`--enable-native-access=ALL-UNNAMED` or documented equivalent)  
**Then** `mvn compile test-compile -pl hdfview` succeeds  
**And** grep for `jarhdf5` and `hdf.hdf5lib` under `hdfview/` returns zero matches (FR-7, D-15)

---

### Story 2.6: Remove HDF5 paths from build.properties and POM enforcers

As an **HDFView maintainer**,
I want HDF5 native paths removed from developer configuration,
So that Maven artifacts are the only HDF5 native source.

**Acceptance Criteria:**

**Given** Story 2.5 is complete  
**When** `build.properties` template omits `hdf5.lib.dir` and `hdf5.plugin.dir` and enforcer/copy plugins no longer require HDF5 paths  
**Then** a fresh Windows dev can build HDF5 without a system HDF5 install  
**And** `hdf.lib.dir` remains documented for HDF4 JNI only (FR-9, FR-8)

---

### Story 2.7: Phase 2 Windows CI with FFM and mandatory UI tests

As an **HDFView maintainer**,
I want Windows CI to build and test with org.hdfgroup FFM,
So that Phase 2 regressions are caught remotely.

**Acceptance Criteria:**

**Given** Story 2.6 is complete  
**When** `ci-windows.yml` (and related Windows build workflows) use FFM dependencies without `repository` bootstrap  
**Then** object tests and mandatory `@Tag("ui")` suite pass on the Windows runner  
**And** no step silently skips HDF5 validation (FR-12 Phase 2, FR-13, D-20)

---

### Story 2.8: Scope macOS and Linux CI for Phase 2

As an **HDFView maintainer**,
I want macOS/Linux CI to stay on JDK 25 with explicit Phase 2 scope,
So that jobs do not false-pass without Windows FFM artifacts.

**Acceptance Criteria:**

**Given** Story 2.7 is complete  
**When** `ci-linux.yml` and `ci-macos.yml` run **object-module** tests (and compile-only or profile-skipped hdfview HDF5 as documented)  
**Then** jobs pass without requiring `windows-x86_64` classifiers  
**And** workflow comments or docs state HDF5 FFM validation is Windows-only for this initiative (FR-12 Phase 2, D-21)

---

### Story 2.9: Phase 2 parity sign-off and documentation

As an **HDFView maintainer**,
I want Phase 2 sign-off recorded with updated maintainer docs,
So that the migration is complete.

**Acceptance Criteria:**

**Given** Stories 2.1–2.8 are complete  
**When** UJ-2 smoke runs (HDF5 + HDF4, full UI suite on Windows)  
**Then** behavior matches Phase 1 baseline for the same sample files  
**And** `CLAUDE.md` and `project-context.md` document org.hdfgroup 2.2.0, no `repository`, HDF4-only `hdf.lib.dir`, and Phase 2 CI policy (FR-10, FR-11, FR-8)

---

### Story-to-FR coverage (validation)

| Story | FRs |
|-------|-----|
| 1.1 | FR-1 |
| 1.2 | FR-2 |
| 1.3, 1.4 | FR-3 |
| 1.5 | FR-12 (P1) |
| 1.6 | FR-13, FR-4 |
| 1.7 | FR-4, FR-11 |
| 2.1 | (spike) |
| 2.2 | FR-5 |
| 2.3 | FR-6 |
| 2.4, 2.5 | FR-7 |
| 2.6 | FR-8, FR-9 |
| 2.7 | FR-12 (P2), FR-13 |
| 2.8 | FR-12 (P2) |
| 2.9 | FR-10, FR-11 |
