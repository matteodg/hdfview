---
stepsCompleted: [1, 2, 3, 4, 5, 6, 7, 8]
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
lastStep: 8
status: complete
completedAt: 2026-06-04
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

## Implementation Patterns & Consistency Rules

### Critical conflict points

Agents could diverge on: mixing Phase 1/2 in one PR; FFM in `hdfview` vs `object`; leftover `hdf.hdf5lib`; reintroducing `repository` or `hdf5.lib.dir`; skipping Windows UI tests; false-green macOS/Linux CI after Phase 2.

### Naming patterns

| Area | Rule |
|------|------|
| HDF5 API (Phase 2) | `javahdf5` / org.hdfgroup types in `hdf.object.h5` only — **no** new `hdf.hdf5lib.*` |
| Packages | `hdf.object.h5` (HDF5), `hdf.object.h4` (HDF4), `hdf.view.*` (SWT UI) |
| BOM | Single `hdf5.stack.version=2.2.0`; no scattered version literals |
| Tests | `uitest/` + `@Tag("ui")`; `object/` + `@Tag("unit")` / `@Tag("fast")` |

### Structure patterns

| Area | Rule |
|------|------|
| Phase 1 PRs | POMs, CI workflows, launchers, Testing Guide, Surefire JVM args — **no** h5 API migration |
| Phase 2 PRs | BOM → delete `repository/` → migrate `object` h5 → fix `hdfview` imports → `build.properties` template |
| Dependencies | `org.hdfgroup` in BOM + `object/pom.xml`; `windows-x86_64` via profile/property |

### Process patterns

| Area | Rule |
|------|------|
| Native errors | Fail at resolve/build when possible; UI uses existing dialogs — no silent native catch |
| SWT | UI thread only (`syncExec` / `asyncExec`) |
| Scope | Minimal diff; preserve copyright headers; Checkstyle/Google style |
| Phase gate | `mvn test -pl object` + Windows `mvn test -pl hdfview` (UI) + UJ-1/UJ-2 smoke |

### Enforcement (agents MUST)

1. Read `architecture.md` + PRD FR for the story.
2. Tag PR **Phase 1** or **Phase 2** in description.
3. Phase 2: zero `hdf.hdf5lib` / `jarhdf5` references before merge.
4. Update `project-context.md` when JVM args or `build.properties` contract changes.
5. Do not lower JaCoCo thresholds to green migration PRs.

### Anti-patterns

- Temporary JNI adapter for HDF5 in Phase 2
- `@Disabled` UI tests without explicit FR-13 waiver
- `install-file` jarhdf5 after repository removal
- Linux Xvfb for SWT UI tests (documented as insufficient)

## Project Structure & Boundaries

### FR → structure mapping

| FRs | Primary locations |
|-----|-------------------|
| FR-1, D-2 | `pom.xml`, `object/pom.xml`, `hdfview/pom.xml` |
| FR-2, D-14 | Surefire/JVM in `object/pom.xml`, `hdfview/pom.xml` |
| FR-3, D-23 | `hdfview/pom.xml` (jpackage), `run-hdfview.bat`, `run-hdfview.sh` |
| FR-4, FR-13 | `object/src/test/`, `hdfview/src/test/java/uitest/` |
| FR-5–FR-10 | `object/src/main/java/hdf/object/h5/`, `object/pom.xml`, root BOM |
| FR-6, D-7 | Remove `repository/`; root `pom.xml` `<modules>` |
| FR-8 | `object/.../h4/`, `build.properties` (`hdf.lib.dir`) |
| FR-9 | `build.properties` template, enforcer/copy in `pom.xml` |
| FR-11–FR-12 | `CLAUDE.md`, `project-context.md`, `.github/workflows/ci-*.yml` |

### Phase 2 migration hotspots

- `object/src/main/java/hdf/object/h5/` — core big-bang
- `object/src/test/java/object/`, `object/src/test/java/misc/`
- `hdfview/src/main/java/hdf/view/` — few direct HDF5 imports
- `hdfview/src/test/java/uitest/` — mandatory UI gate (Windows)

### Migration-relevant tree

```
hdfview/
├── pom.xml
├── build.properties              # Ph2: hdf.lib.dir only
├── CLAUDE.md
├── run-hdfview.bat / .sh
├── scripts/build-dev.sh
├── repository/                   # REMOVED Phase 2
├── object/
│   ├── pom.xml                   # org.hdfgroup deps (Ph2)
│   └── src/main/java/hdf/object/h5/
├── hdfview/
│   ├── pom.xml
│   ├── src/main/java/hdf/view/
│   └── src/test/java/uitest/
├── .github/workflows/
│   ├── ci-windows.yml
│   ├── ci-linux.yml
│   └── ci-macos.yml
└── docs/Testing-Guide.md
```

### Boundaries

| Boundary | Rule |
|----------|------|
| HDF5 | `object` only → javahdf5/FFM; `hdfview` uses `hdf.object.*` |
| HDF4 | `object.h4` + JNI via `hdf.lib.dir` |
| UI | SWT in `hdfview` only |
| Build | Phase 2: no `repository` install-file |

**Data flow:** User → `hdf.view.HDFView` (SWT) → `hdf.object.*` → FFM (HDF5) / JNI (HDF4) → natives

## Architecture Validation Results

### Coherence validation

**Decision compatibility:** Phased JDK-then-FFM ordering avoids mixing concerns. Dual natives (FFM HDF5 + JNI HDF4) is explicit. Windows-only classifiers align with Phase 2 CI split (D-20 vs D-21).

**Pattern consistency:** Phase-tagged PRs, package boundaries, and enforcement rules match D-1–D-24.

**Structure alignment:** FR mapping points to concrete paths; `repository` removal is reflected in tree.

### Requirements coverage

| FR | Architectural support |
|----|------------------------|
| FR-1–4 | D-2, D-11, D-13, D-14, D-17–D-19, D-23 |
| FR-5–10 | D-4–D-9, D-6, D-10, D-20 |
| FR-11–13 | D-17, D-22, D-24, patterns § |

**NFRs:** Parity via UJ-1/UJ-2 + UI tests; reliability via build-time fail-fast; maintainability via BOM + Maven natives.

### Implementation readiness

**Ready for Phase 1** with high confidence. **Phase 2** requires pre-merge **API spike** (assumption in PRD §9) before big-bang.

### Gap analysis

| Priority | Gap | Mitigation |
|----------|-----|------------|
| Important | Phase 2 javahdf5 API spike not executed | Story 0 / spike before FR-7 merge |
| Important | `build-windows.yml`, `publish-maven-packages.yml` may still reference `jarhdf5` | Audit in Phase 1 CI PR |
| Minor | `project-context.md` still describes Java 21 + repository-first | D-24 end of Phase 1 |
| Minor | Exact UI test class list per phase | Epics/stories |

### Architecture completeness checklist

**Requirements analysis:** [x] context [x] scale [x] constraints [x] cross-cutting

**Architectural decisions:** [x] critical decisions [x] stack [x] integration [ ] performance (explicitly out of scope per PRD)

**Implementation patterns:** [x] naming [x] structure [x] communication N/A desktop [x] process

**Project structure:** [x] directories [x] boundaries [x] integration [x] FR mapping

### Readiness assessment

**Overall status:** **READY WITH MINOR GAPS**

**Confidence:** High for Phase 1; Medium for Phase 2 until spike completes

**Strengths:** Clear two-phase split; explicit CI strategy; brownfield boundaries preserved

**Future enhancement:** Linux/macOS FFM classifiers; HDF4 FFM; JPMS

### Implementation handoff

**Agent guidelines:** Follow `architecture.md` + PRD FR IDs; Phase 1 before Phase 2; grep gates for `hdf.hdf5lib` in Phase 2.

**First implementation priority:** Phase 1 Epic — bump `maven.compiler.release` to 25, update `ci-*.yml` JDK, fix launchers and `build-dev.sh`, run Windows UI tests, jpackage smoke.
