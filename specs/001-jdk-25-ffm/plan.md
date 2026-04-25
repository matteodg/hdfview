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
**Scale/Scope**: Root `pom.xml` properties + module POM overrides + **all** workflows using
`actions/setup-java` with `java-version: '21'` + scripts/docs that mention Java 21

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle / rule (from `.specify/memory/constitution.md`) | Status | Notes |
|-----------------------------------------------------------|--------|-------|
| I. Build & Tooling Consistency (Maven-only) | **Pass** | Still Maven-only; JDK major bump is in
spec scope. |
| I / Engineering Standards: text still says **Java 21** | **Violation** | Constitution lags the
approved feature. **Mitigation**: amend constitution (MINOR bump) in the same change series as JDK
25 land (see Complexity Tracking). |
| II. Native Library Safety | **Pass** | Baseline bump must not weaken crash guards; no native call
surface change required for this feature. |
| III. Tests where they matter | **Pass** | Run existing suites on JDK 25; add tests only if a
JDK-specific regression is found. |
| IV. Minimal, focused changes | **Pass** | Touch only version pins, enforcer, docs, CI matrices,
constitution alignment. |
| V. Quality gates & documentation | **Pass** | Verify PMD/Checkstyle/JaCoCo still run; bump static
analysis **target JDK** metadata (e.g. PMD `targetJdk`) to **25** alongside compiler `release`. |

**Gate conclusion**: Proceed with implementation **only** if constitution and docs are updated
**together** with the JDK 25 baseline (no “code says 25 / constitution says 21” drift).

### Post-design re-check

Research and contracts lock the enforcement strategy (Maven `release` **25**, Enforcer
`requireJavaVersion`, CI `setup-java` **25**, launcher minimum **25**, constitution **25**). No
additional constitution conflicts identified.

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
├── ci-linux.yml                 # setup-java 21 → 25
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
└── …                            # grep for java-version / Java 21
run-hdfview.sh                   # Java version gate (currently >=21)
run-hdfview.bat
README.md
CLAUDE.md
.specify/memory/constitution.md # amend Java 21 → 25 (sync impact header + version bump)
```

**Structure Decision**: This feature is a **cross-cutting build & CI** change anchored at the root
`pom.xml`, propagated to GitHub Actions under `.github/workflows/`, developer launchers, and
governance docs. Application Java sources under `object/src`, `hdfview/src`, and `src/` require **no
structural move**—only bytecode level change via `release`.

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| Constitution still mandates **Java 21** while the feature mandates **25** | Single source of
truth for agents and reviewers; avoids contradictory guidance | Leaving constitution at 21 would
fail the plan gate and mislead automation |
