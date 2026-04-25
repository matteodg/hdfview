# Implementation Plan: Migrate HDF5 from jarhdf5 to hdf5-java-ffm

**Branch**: `003-migrate-hdf5-ffm` | **Date**: 2026-04-26 | **Spec**: [spec.md](./spec.md)  
**Input**: Feature specification from `/specs/003-migrate-hdf5-ffm/spec.md`

## Summary

Cut over HDFView’s HDF5 Java stack from the legacy **`jarhdf5:jarhdf5`** Maven line to **`org.hdfgroup:hdf5-java-ffm`** (version **2.2.0**, per-platform **classifier**), with matching **`hdf5-native`** where already modeled. Per [HDF Group Maven documentation](https://support.hdfgroup.org/documentation/hdf5/latest/_c_b__maven_artifacts.html), JNI and FFM artifacts expose the **same** `hdf.hdf5lib` API (`H5`, `HDF5Constants`, etc.), so **large-scale Java rewrites are not expected**; work concentrates on **dependency graph**, **JVM native-access flags**, **CI/local install pipelines**, and **verifying no duplicate HDF5 Java bindings** (constitution + spec FR-004).

## Technical Context

**Language/Version**: Java **25** (repository standard)  
**Primary Dependencies**: Maven multi-module (`repository/`, `object/`, `hdfview/`); remove **`jarhdf5:jarhdf5`**; consume **`org.hdfgroup:hdf5-java-ffm`** + **`org.hdfgroup:hdf5-native`** at **2.2.0** with **active-platform classifiers** (pattern established in **002-hdf5-maven-deps**). HDF4 remains on **`jarhdf:jarhdf`** (unchanged by this feature).  
**Storage**: N/A (desktop app + local HDF files)  
**Testing**: JUnit 5 / Surefire (`object/`, `hdfview/`); JVM `argLine` in POMs and CI must align with FFM native-access policy  
**Target Platform**: Windows x86_64 / amd64 (profiles exist); **Linux/macOS CI and developers** require **additional** `hdf5-java-ffm` + `hdf5-native` classifier profiles and matching `dependencyManagement` rows once artifacts are confirmed (see `research.md`)  
**Project Type**: Desktop (SWT) + core **`object`** library  
**Performance Goals**: No regression in common open/browse flows (constitution IV/V)  
**Constraints**: **Single** HDF5 Java binding on the resolved graph (HDF doc: only one of JNI vs FFM); FFM requires **`--enable-native-access=ALL-UNNAMED`** (or equivalent scoped flag); native library safety and clear errors on misconfiguration (constitution II)  
**Scale/Scope**: ~**40+** Java compilation units import `hdf.hdf5lib.*` (grep-based inventory); all POMs, run scripts, and **10+** workflow locations reference `jarhdf5` install or JVM flags

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-checked after Phase 1 design.*

| Principle | Assessment |
|-----------|--------------|
| **I. Build & Tooling (Maven, Java 25, modules)** | PASS — migration stays on Maven 25; extend profiles/`dependencyManagement` rather than new build systems. |
| **II. Native Library Safety** | PASS — enforce one binding; validate missing-native and plugin paths; avoid JVM fatal exits where errors can be surfaced. |
| **III. Tests** | PASS — keep `object/` HDF5 tests runnable; update Surefire/CI `argLine` for FFM native access; do not rely on disabled UI tests for binding proof. |
| **IV. Minimal, Focused** | PASS — prefer coordinate + flag + pipeline changes over speculative refactors; Java edits only where API/behavior deltas are proven. |
| **V. Quality & Docs** | PASS — update `CLAUDE.md`, `README.md`, contributor quickstarts, and workflow docs when `jarhdf5` steps are removed. |

**Post-design note**: Official docs assert API identity between JNI and FFM; **implementation still MUST** run full `object` test suite and spot-check UI on at least one platform after flag and dependency changes.

## Project Structure

### Documentation (this feature)

```text
specs/003-migrate-hdf5-ffm/
├── plan.md              # This file
├── research.md          # Phase 0
├── data-model.md        # Phase 1
├── quickstart.md        # Phase 1
├── contracts/           # Phase 1
└── tasks.md             # /speckit-tasks (not created here)
```

### Source Code (repository root)

```text
object/pom.xml                 # Remove jarhdf5 dependency; Surefire argLine; OS profiles for hdf5-java-ffm
hdfview/pom.xml                # Surefire / exec argLine native access
pom.xml                        # dependencyManagement: add non-Windows classifiers when ready
repository/pom.xml             # install-file execution for jarhdf5 — remove or gate off for HDF5-only path
object/src/main/java/hdf/object/h5/*.java   # Mostly unchanged if API parity holds; message strings may reference "jarhdf5"
object/src/test/java/**/*.java
hdfview/src/main/java/**/*.java
run-hdfview.sh / run-hdfview.bat
CLAUDE.md / README.md
.github/workflows/*.yml        # jarhdf5 install steps; maven-quality argLine; build-* install paths
```

**Structure Decision**: Changes are **cross-cutting** (Maven + scripts + CI + JVM flags) with **`object`** as the primary compile consumer of `hdf.hdf5lib`; **`hdfview`** inherits tests and runtime packaging.

## Complexity Tracking

No constitution violations requiring a waiver table; no additional frameworks or modules introduced beyond extending existing HDF5 org.hdfgroup profiles.
