# Implementation Plan: HDF5 native and Java FFM via published artifacts

**Branch**: `002-hdf5-maven-deps` | **Date**: 2026-04-26 | **Spec**:
[spec.md](./spec.md)

**Input**: Feature specification from `/specs/002-hdf5-maven-deps/spec.md`

**Note**: This file is produced by `/speckit-plan`. Executable task breakdown lives in
[`tasks.md`](./tasks.md) (maintained by `/speckit-tasks` and implementation).

## Summary

Adopt the HDF Group **Maven coordinates** for **HDF5 2.2.0** native packaging and **Java FFM**
bindings: `org.hdfgroup:hdf5-native:2.2.0` and **`org.hdfgroup:hdf5-java-ffm:2.2.0`** with a
**platform-specific classifier** per OS/arch (reference: **`windows-x86_64`** on 64-bit Windows).
**Other platforms are assumed to have correspondent classifiers** for the same artifact and
version. The delivery wires these into the **multi-module Maven build** (properties,
`dependencyManagement`, and/or module dependencies with **OS-scoped activation** so only the active
platform’s classifier is resolved), updates
**contributor documentation** for pre–Maven Central resolution (local `~/.m2` or mirror) versus
post-central default resolution, and defines how **native binaries** from the artifact line reach
the existing **package/copy** path without silent version drift. **Replacing** legacy
`jarhdf5` / `hdf5.lib.dir` JNI workflows entirely is **not** required by the spec if the spec’s
FRs are met by declarations plus clear scope in docs; any **code-level FFM migration** is a
separate slice unless explicitly merged into the same task list.

## Technical Context

**Language/Version**: Java **25** (repository baseline per constitution)  
**Primary Dependencies**: Maven 3.x; existing **`jarhdf5`** / **`jarhdf`** JARs at
`${hdf5.version}` (**2.0.0** today); new **`org.hdfgroup:hdf5-native:2.2.0`** and
**`org.hdfgroup:hdf5-java-ffm:2.2.0`** + **per-platform classifiers** (e.g. **`windows-x86_64`**)  
**Storage**: N/A (build-time dependency resolution + optional unpack into `target/` for packaging)  
**Testing**: `mvn test` / Surefire; `mvn dependency:tree` (or `-q validate`) for resolution checks on
Windows x86_64  
**Target Platform**: **Per-platform classifiers** on `hdf5-java-ffm` (Windows x86_64 is the
reference); platforms without a wired profile keep current `build.properties` / `hdf5.lib.dir`
behavior until the same Maven pattern is enabled for their classifier  
**Project Type**: Multi-module desktop Java app (`repository/`, `object/`, `hdfview/`)  
**Performance Goals**: No regression in dependency resolution time beyond one-time unpack/copy of
native payloads where introduced  
**Constraints**: Constitution **II** (native safety)—avoid loading **two incompatible HDF5 native
stacks** on Windows; `--enable-native-access` / module-native policy must stay coherent with
whatever module loads FFM vs JNI  
**Scale/Scope**: Root `pom.xml` properties + **one primary consumer module** (expected **`object/`**
for Java API classpath; **`hdfview/`** for packaged natives if Windows bundle changes) +
`README.md` / `CLAUDE.md` / `build.properties` docs touchpoints

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle / rule (from `.specify/memory/constitution.md`) | Status | Notes |
|-----------------------------------------------------------|--------|-------|
| I. Build & Tooling Consistency (Maven-only, Java 25) | **Pass** | New deps are Maven-first-party; stay within existing modules. |
| II. Native Library Safety | **Pass** | Plan MUST avoid ambiguous `PATH` / duplicate `hdf5.dll` when both `hdf5.lib.dir` and artifact unpack supply natives—pick one precedence for Windows x86_64 and document it. |
| III. Tests where they matter | **Pass** | Prefer automated check that dependency graph includes exact GAV (script or Maven enforcer rule) where feasible; no UI test requirement. |
| IV. Minimal, focused changes | **Pass** | Add coordinates + scoped wiring + docs; defer broad JNI→FFM refactors unless tasks explicitly include them. |
| V. Quality gates & documentation | **Pass** | Document pre-central mirror/local install; OWASP/dependency plugins already project-wide—record any new suppressions only with justification. |

**Gate conclusion**: Proceed. Re-check after design: confirm **inactive OS profiles** do not
resolve **foreign** `hdf5-java-ffm` classifiers on CI; confirm **FR-003** (no silent fallback) via
pinned versions and optional Enforcer `requireUpperBoundDeps` / banned duplicates analysis.

### Post-design re-check

- **OS gating**: Use Maven `activation` per target platform (e.g. Windows + amd64/x86_64 for
  **`windows-x86_64`**, and **correspondent** `os` / `arch` tuples when additional classifiers are
  wired) so CI jobs only resolve the classifier for the OS they run on.
- **Duplicate natives**: `research.md` records the chosen precedence (artifact unpack vs
  `hdf5.lib.dir`) for Windows x86_64.

## Project Structure

### Documentation (this feature)

```text
specs/002-hdf5-maven-deps/
├── plan.md              # This file
├── research.md          # Phase 0
├── data-model.md        # Phase 1
├── quickstart.md        # Phase 1
├── contracts/           # Phase 1 (build resolution contract)
├── spec.md
└── checklists/
    └── requirements.md
```

### Source Code (repository root)

```text
pom.xml                          # Properties for org.hdfgroup HDF5 2.2.0 GAVs; dependencyManagement
object/pom.xml                   # Scoped dependencies + test/compile classpath; profile activation
hdfview/pom.xml                  # If Windows packaging must copy natives from unpacked artifact
repository/pom.xml               # Only if local jarhdf5 install pipeline must align with new coords
README.md                        # Windows x86_64: pre-central vs central resolution
CLAUDE.md                        # Same, shorter pointer for agents
build.properties                 # Comment only if clarifying interaction with hdf5.lib.dir
```

**Structure Decision**: This feature is a **supply-chain and build wiring** change: versioned
**org.hdfgroup** artifacts enter the Maven graph under **OS-scoped profiles** (Windows x86_64
first), with documentation for resolver visibility and the **per-platform classifier** assumption.
Legacy **`jarhdf5`** at **2.0.0** may remain until a follow-on removes it;
the spec is satisfied by explicit **2.2.0** declarations and clear scope notes (**FR-005**).

## Complexity Tracking

> No constitution violations requiring justification tables. Optional complexity: **two HDF5 Java
> API lines** (JNI jarhdf5 vs FFM jar) may coexist briefly—document in `research.md` and cap scope in
> `tasks.md` when generated.
