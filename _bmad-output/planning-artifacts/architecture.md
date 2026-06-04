---
stepsCompleted: [1, 2, 3, 4]
inputDocuments:
  - planning-artifacts/prds/prd-hdfview-2026-06-04/prd.md
  - planning-artifacts/prds/prd-hdfview-2026-06-04/addendum.md
  - planning-artifacts/prds/prd-hdfview-2026-06-04/.decision-log.md
  - project-context.md
  - CLAUDE.md
  - docs/Testing-Guide.md
  - docs/guides/Cross-Platform-Build-Quick-Reference.md
workflowType: architecture
project_name: hdfview
user_name: Matteo
date: 2026-06-04
---

# Architecture Decision Document

_This document builds collaboratively through step-by-step discovery. Sections are appended as we work through each architectural decision together._

## Project Context Analysis

### Requirements Overview

**Functional Requirements (13 FRs, two phases):**

| Phase | FRs | Architectural meaning |
|-------|-----|------------------------|
| **Phase 1 — JDK 25** | FR-1–4, FR-11–13 | Toolchain uplift only: root BOM, Surefire/JaCoCo/PMD/Checkstyle, `run-hdfview.*`, **jpackage** (JDK 25 runtime), GitHub Actions on **Windows/macOS/Linux** with JDK 25. HDF5 remains **JNI** (`jarhdf5`, `hdf.hdf5lib`, `repository` module). |
| **Phase 2 — HDF5 FFM** | FR-5–10, FR-11–13 | **Binding swap:** `org.hdfgroup` **2.2.0** Maven stack (`hdf5-native`, szip/zlib natives, `hdf5-java-ffm`, `javahdf5`) with **`windows-x86_64`** classifier. **Remove `repository` module.** **Big-bang** replacement of `hdf.hdf5lib` in `object`/`hdfview` (~40+ Java files touch HDF5 APIs). **HDF4 stays JNI** via `hdf.lib.dir` only. Drop `hdf5.lib.dir` / `hdf5.plugin.dir`. |

**Non-Functional Requirements (from PRD + brownfield):**

- **Parity / reliability:** No user-visible regression; prefer **build-time** failure over JVM native crashes (FR-10 NFRs).
- **Maintainability:** Single HDF5 Java version property (**2.2.0**); Maven-resolved natives; align with HDF Group artifacts.
- **Testability:** Mandatory **`@Tag("ui")`** SWTBot suite both phases (FR-13); object `@Tag("unit")`/`fast` on all OS CI in Phase 1.
- **CI divergence:** Phase 2 FFM artifacts are **Windows-only** — macOS/Linux jobs need an **explicit scoped strategy** (PRD assumption; architecture must decide).
- **Platform:** Desktop **SWT** (NatTable, JFace); default POM SWT profile is **Linux GTK** — Windows dev/CI is primary for Phase 2 validation.
- **Quality gates:** JaCoCo (~60%/50%), Checkstyle, PMD — must remain valid on JDK 25.

**Scale & Complexity:**

- **Primary domain:** Brownfield **Java desktop** (Maven multi-module: `repository` → `object` → `hdfview`).
- **Complexity level:** **High** for Phase 2 (dual native stacks: FFM HDF5 + JNI HDF4; big-bang API migration; CI matrix tension).
- **Estimated architectural components:** Build/BOM, native binding layer (`object` h5 package), HDF4 JNI bridge, GUI (`hdfview`), test/CI matrix, packaging (jpackage), launcher scripts, docs/agent context.

### Technical Constraints & Dependencies

**Current baseline (brownfield):**

- Java **21** in POM; local runs on **Java 25**; HDF5 **2.1.1** via `build.properties`; `hdf5.version` **2.0.0** / `jarhdf5` in `repository/lib`.
- Module system **disabled** (classpath build).
- `Testing-Guide.md` still documents Java 21 and UI tests **disabled in CI** — **conflicts with PRD FR-12/FR-13**; must be updated in Phase 1.

**Phase 2 constraints:**

- **`org.hdfgroup` 2.2.0** coordinates in `addendum.md` (user has artifacts in local Maven repo).
- **No** HDF4 FFM, **no** Linux/macOS FFM classifiers in this initiative.
- **Big-bang** — no long-lived JNI↔FFM adapter for HDF5.
- JVM args will change: `--enable-native-access=jarhdf5` likely replaced by FFM/JDK 25 native-access rules for SWT and HDF5.

**PRD assumptions to resolve in later architecture steps:**

1. `org.hdfgroup` 2.2.0 API covers all HDF5 code paths (spike on Float16/BFLOAT16, vlen, refs, plugins).
2. macOS/Linux CI remain meaningful after Phase 2 without Windows FFM classifiers.

### Cross-Cutting Concerns Identified

1. **Dual native binding model (Phase 2):** HDF5 FFM + HDF4 JNI in one JVM — classpath, `java.library.path`, and launcher quoting (`Program Files`).
2. **Module graph simplification:** Removing `repository` changes bootstrap order and all workflows that `install-file` `jarhdf5`.
3. **API migration surface:** Concentrated in `hdf.object.h5.*` and scattered `hdfview` references; tests in `object/src/test` heavily use `hdf.hdf5lib`.
4. **CI/CD matrix:** `ci-windows.yml`, `ci-linux.yml`, `ci-macos.yml`, `build-*.yml`, `maven-ci-orchestrator.yml` — JDK 25 bump touches many; Phase 2 Windows-only FFM vs PRD “all OS JDK 25”.
5. **UI test infrastructure:** SWT requires real display; PRD mandates UI tests in CI “where display is available” — architecture must define per-OS approach (Windows primary; Linux Xvfb **not** sufficient per existing Testing Guide).
6. **Packaging:** jpackage profiles and bundled JRE must track JDK 25 in Phase 1.
7. **Documentation drift:** `project-context.md`, `CLAUDE.md`, `build-dev.sh` (SNAPSHOT vs 3.4.1), `Testing-Guide.md`, `run-hdfview.bat` path quoting.
8. **Version skew:** POM `hdf5.version` 2.0.0 vs local 2.1.1 vs target 2.2.0 — resolved in Phase 2 only.

## Foundation: Brownfield Stack (Step 3)

**No greenfield starter.** HDFView **3.4.1** remains the application; this initiative changes platform and HDF5 binding only.

| Layer | Decision (retain unless noted) |
|-------|--------------------------------|
| **App structure** | Maven modules: Phase 1 `repository` → `object` → `hdfview`; Phase 2 `object` → `hdfview` only |
| **UI** | Eclipse **SWT 3.126.0** + NatTable + JFace; `hdf.view.*` factories unchanged |
| **Data** | `hdf.object.*` format packages; HDF5 logic in `hdf.object.h5` |
| **Build** | Maven BOM, `build.properties`, non-modular classpath |
| **Phase 2 dependency “starter”** | `org.hdfgroup` **2.2.0** BOM-managed artifacts (see `addendum.md`) |

**Technical preferences (from PRD + project context):** Minimal diffs; no JPMS without explicit effort; preserve module boundaries and test tags; maintainability over new JDK features.

## Core Architectural Decisions

### Decision Priority Analysis

**Critical (block implementation):** D-1 phased delivery · D-2 JDK 25 · D-4–D-9 FFM stack & big-bang · D-17–D-21 CI/UI gates

**Important:** D-10 BOM · D-14–D-16 JVM/launchers · D-22–D-24 docs/packaging

**Deferred:** Linux/macOS FFM classifiers · HDF4 FFM · JPMS

### Delivery & versioning

| ID | Decision | Version | Rationale |
|----|----------|---------|-----------|
| D-1 | Two-phase: JDK 25 then HDF5 FFM | — | Isolate toolchain vs binding risk |
| D-2 | `maven.compiler.release=25` | JDK 25 | PRD FR-1; FFM alignment |
| D-3 | HDF5 stack **2.2.0** in Phase 2 only | org.hdfgroup 2.2.0 | Phase 1 keeps jarhdf5 |

### Native binding

| ID | Decision | Rationale |
|----|----------|-----------|
| D-4 | HDF5 via `hdf5-java-ffm` + `javahdf5` | Maintainability; PRD |
| D-5 | HDF4 JNI + `hdf.lib.dir` only | PRD FR-8 |
| D-6 | Big-bang `hdf.hdf5lib` → javahdf5; no adapter | PRD FR-7 |
| D-7 | Remove `repository` + `jarhdf5` (Phase 2) | PRD FR-6 |
| D-8 | Drop `hdf5.lib.dir` / `hdf5.plugin.dir` (Phase 2) | PRD FR-9 |
| D-9 | Native classifier `windows-x86_64` (Phase 2) | PRD scope |

**Pre-Phase-2 spike:** Float16/BFLOAT16, vlen, refs, plugins on javahdf5 API.

### Build & BOM

| ID | Decision | Rationale |
|----|----------|-----------|
| D-10 | BOM property `hdf5.stack.version=2.2.0` | Single version line |
| D-11 | Phase 1: keep repository + jarhdf5 | FR-1 scope |
| D-12 | Non-modular classpath | SWT / existing build |
| D-13 | Phase 1: fix `build-dev.sh` JAR names, quoted launcher paths | Known Windows issues |

### JVM & runtime

| ID | Decision | Rationale |
|----|----------|-----------|
| D-14 | Phase 1 Surefire: `--add-opens` + `--enable-native-access=jarhdf5` (document updates) | FR-2 |
| D-15 | Phase 2: FFM native access + SWT via `--enable-native-access=ALL-UNNAMED` (or documented scope) | Java 25 + SWT |
| D-16 | Launchers quote paths; Phase 2 `java.library.path` for HDF4 only | Program Files / FR-8 |

### Testing & CI

| ID | Decision | Rationale |
|----|----------|-----------|
| D-17 | Mandatory `@Tag("ui")` on **Windows** for both phases | FR-13 |
| D-18 | Phase 1: ci-windows, ci-linux, ci-macos → JDK 25 | FR-12 |
| D-19 | Phase 1: **Windows CI runs UI tests**; Linux/macOS object tests + maintainer UI | Testing Guide vs PRD |
| D-20 | Phase 2: Windows CI full FFM + object + UI | FR-12 |
| D-21 | Phase 2: macOS/Linux CI JDK 25, **object-only** (or profile); no false-green HDF5 | Windows-only FFM |
| D-22 | Update Testing-Guide.md in Phase 1 | Doc alignment |

### Packaging & docs

| ID | Decision | Rationale |
|----|----------|-----------|
| D-23 | jpackage JDK 25 runtime (Phase 1) | FR-3 |
| D-24 | Update CLAUDE.md + project-context.md each phase | FR-11 |

### Decision Impact Analysis

**Phase 1 sequence:** POM JDK 25 → CI JDK 25 → Surefire/JVM → launchers/jpackage → Testing Guide → Windows UI CI → parity (UJ-1).

**Phase 2 sequence:** API spike → BOM/deps → remove repository → big-bang `hdf.object.h5` → hdfview imports → build.properties HDF5 removal → Windows CI FFM → UI + parity (UJ-2).

**Dependencies:** D-7/D-8 require D-4–D-6 complete; D-21 depends on D-9; D-17 blocks both phase merges.
