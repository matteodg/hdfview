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

<!-- Stories appended in step 3 -->
