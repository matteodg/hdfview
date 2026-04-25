# Implementation Plan: JDK 25 baseline for FFM-ready builds

**Branch**: `001-jdk-25-ffm` | **Date**: 2026-04-26 | **Spec**:
[spec.md](./spec.md)

**Input**: Feature specification from `/specs/001-jdk-25-ffm/spec.md`

**Note**: This file is produced by `/speckit-plan`. Phase 2 task breakdown is `/speckit-tasks`
(`tasks.md` not created here).

## Summary

Raise the HDFView repository **minimum Java baseline from 21 to 25** so the codebase can rely on
**Java 25 language/runtime features**, including the **Foreign Function & Memory (FFM)** API, for
future native-interop work. This delivery updates **Maven compiler settings**, adds **explicit JDK
enforcement** (Maven and/or launch scripts), aligns **GitHub Actions** JDK selection, refreshes
**contributor documentation** (`README.md`, `CLAUDE.md`, launcher scripts), and **amends the project
constitution** so governance matches the new baseline. **No JNI→FFM migration** is in scope for
this feature (baseline only), per `spec.md` assumptions.

## Technical Context

**Language/Version**: Java **25** (minimum for compile, test, and documented local runs)  
**Primary Dependencies**: Maven 3.x; Eclipse **SWT** / NatTable; HDF4/HDF5 **native** libraries via
existing JNI/JARHDF layers; SLF4J; JUnit 5 (+ Vintage where used)  
**Storage**: N/A (desktop app + local HDF files)  
**Testing**: `mvn test` / Surefire; tags (`unit`, `integration`, `ui`); SWTBot where applicable  
**Target Platform**: Linux, Windows, macOS (developer + CI); SWT platform artifacts remain
OS-specific  
**Project Type**: Multi-module **desktop** Java application (`repository/`, `object/`, `hdfview/`)  
**Performance Goals**: No regression in typical open/browse flows; avoid extra UI-thread work  
**Constraints**: JVM flags for native access (`--enable-native-access=jarhdf5`, `--add-opens` …)
must remain valid; jpackage/installer pipelines must keep using a **JDK that matches** the declared
baseline where they compile bytecode  
**Scale/Scope**: Root `pom.xml` properties + module POM overrides + GitHub Actions `setup-java`
jobs + scripts/docs that still reference older Java baselines (cleanup via `tasks.md` **T027**).

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle / rule (from `.specify/memory/constitution.md`) | Status | Notes |
|-----------------------------------------------------------|--------|-------|
| I. Build & Tooling Consistency (Maven-only, **Java 25**) | **Pass** | Constitution **v1.1.0+**
requires Java **25**; matches this feature’s **FR-001**. |
| II. Native Library Safety | **Pass** | Baseline bump must not weaken crash guards; no native call
surface change required for this feature. |
| III. Tests where they matter | **Pass** | Run existing suites on JDK 25; add tests only if a
JDK-specific regression is found. |
| IV. Minimal, focused changes | **Pass** | Touch only version pins, enforcer, docs, CI matrices,
constitution alignment. |
| V. Quality gates & documentation | **Pass** | Verify PMD/Checkstyle/JaCoCo still run; PMD
`targetJdk` and compiler `release` raised to **25** in implementation. |

**Gate conclusion**: Constitution, Maven/CI, and contributor docs MUST stay aligned on **Java 25**
for merge. Historical note: pre-**v1.1.0** constitution text referenced Java 21; that gate is
**closed** once `.specify/memory/constitution.md` is at **1.1.0** or newer.

### Post-design re-check

Enforcement strategy: Maven `release` **25**, Enforcer `requireJavaVersion` **`[25,)`**, CI
`setup-java` **25**, launcher minimum **25**, constitution **Java 25** (**v1.1.0**). Re-scan for
stale “Java 21” strings during polish (**T027**).

## Project Structure

### Documentation (this feature)

```text
specs/001-jdk-25-ffm/
├── plan.md              # This file
├── research.md          # Phase 0
├── data-model.md        # Phase 1
├── quickstart.md        # Phase 1
├── contracts/           # Phase 1 (developer environment contract)
├── spec.md
└── checklists/
    └── requirements.md
```

### Source Code (repository root)

```text
pom.xml                          # maven.compiler.*, pluginManagement compiler config, PMD targetJdk
object/pom.xml                   # compiler release overrides if any
hdfview/pom.xml                  # compiler release overrides if any
.github/workflows/
├── ci-linux.yml                 # setup-java 25
├── ci-macos.yml
├── ci-windows.yml
├── maven-build.yml              # if present / shared patterns
├── maven-ci-orchestrator.yml
├── maven-quality.yml
├── maven-security.yml
├── build-linux.yml
├── build-macos.yml
├── build-windows.yml
├── publish-maven-packages.yml
├── release.yml                  # verify any embedded JDK setup
└── …                            # grep for stale java-version / Java baseline strings
run-hdfview.sh                   # Java version gate (>=25)
run-hdfview.bat
README.md
CLAUDE.md
.specify/memory/constitution.md # Java 25 baseline (v1.1.0+)
```

**Structure Decision**: This feature is a **cross-cutting build & CI** change anchored at the root
`pom.xml`, propagated to GitHub Actions under `.github/workflows/`, developer launchers, and
governance docs. Application Java sources under `object/src`, `hdfview/src`, and `src/` require **no
structural move**—only bytecode level change via `release`.

## Complexity Tracking

> **Historical**: Constitution **v1.0.0** still referenced Java **21** while this feature targets
**25**. Resolved by amending `.specify/memory/constitution.md` to **v1.1.0** (Java **25**) in the same
delivery series as the build/CI/doc updates.
