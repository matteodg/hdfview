---
baseline_commit: f44d06a9f86eaef1b6ff0058cd24c8a5ee6f3bd1
---

# Story 1.1: Set JDK 25 as Maven compiler baseline

Status: review

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As an **HDFView maintainer**,
I want the project to compile with **Java 25** across all modules,
so that we align the toolchain before any HDF5 FFM or CI changes.

## Acceptance Criteria

1. **Given** the root BOM and module POMs, **when** `maven.compiler.release` (and aligned compiler plugin settings) are **25**, **then** `mvn clean package -DskipTests` succeeds on Windows x86_64 with **JDK 25** installed.
2. **Given** the build completes, **then** `repository`, `object`, and `hdfview` modules all compile without release/target mismatch errors.
3. **Given** documentation is updated, **then** `CLAUDE.md` states **JDK 25** as the minimum Java version for building HDFView (FR-1).
4. **Given** this is Phase 1 Story 1.1, **then** no changes to HDF5 bindings (`jarhdf5`, `hdf.hdf5lib`), `org.hdfgroup` dependencies, or GitHub Actions JDK versions (deferred to Stories 1.5+).

## Tasks / Subtasks

- [x] Task 1: Centralize Java 25 in root `pom.xml` properties (AC: #1)
  - [x] Set `maven.compiler.source` and `maven.compiler.release` to `25`
  - [x] Update `pluginManagement` `maven-compiler-plugin` `<source>`, `<target>`, `<release>` to `25`
  - [x] Update `maven-javadoc-plugin` `<source>` / `<target>` to `25` in pluginManagement
- [x] Task 2: Align module-level compiler overrides (AC: #2)
  - [x] `object/pom.xml` — `maven-compiler-plugin` configuration (lines ~121–123)
  - [x] `hdfview/pom.xml` — `maven-compiler-plugin` configuration (lines ~789–791)
  - [x] Confirm `repository/pom.xml` inherits from parent (no stale `21` override)
- [x] Task 3: Verify compile on JDK 25 (AC: #1, #2)
  - [x] Run `mvn clean package -DskipTests` from repo root with JDK 25 on PATH
  - [x] Confirm `repository` → `object` → `hdfview` build order succeeds
- [x] Task 4: Update minimum Java in `CLAUDE.md` (AC: #3)
  - [x] Replace Java 21 references in "Key Build Configuration" / Java Version bullets with JDK 25
  - [x] Optionally align `CONTRIBUTING.md` prerequisites (not required by AC but recommended)

## Dev Notes

### Story scope guardrails (Phase 1)

- **In scope:** Compiler release level only — POM properties and `maven-compiler-plugin` / javadoc source levels.
- **Out of scope for this story:** Surefire JVM args (Story 1.2), launchers / `build-dev.sh` (1.3), jpackage (1.4), CI workflows (1.5), UI tests (1.6), full `project-context.md` refresh (1.7).
- **Do not** bump `hdf5.version`, add `org.hdfgroup` artifacts, or remove `repository` module.

### Current state (files to modify)

| File | Current | Target |
|------|---------|--------|
| `pom.xml` properties L18–19 | `21` | `25` |
| `pom.xml` pluginManagement compiler L122–124 | `21` | `25` |
| `pom.xml` pluginManagement javadoc L298–299 | `21` | `25` |
| `object/pom.xml` compiler L121–123 | `21` | `25` |
| `hdfview/pom.xml` compiler L789–791 | `21` | `25` |
| `CLAUDE.md` | Java 21 minimum | JDK 25 minimum |

**Preserve:** `forceLegacyJavacApi`, `maven.compiler.testCompilerArgument=-classpath`, non-modular classpath build, module order `repository` → `object` → `hdfview`.

### Architecture compliance

- [Source: `_bmad-output/planning-artifacts/architecture.md`] D-2: `maven.compiler.release=25`
- [Source: `_bmad-output/planning-artifacts/architecture.md`] D-11/D-12: Phase 1 keeps `repository` + `jarhdf5` unchanged
- [Source: `_bmad-output/planning-artifacts/architecture.md`] Phase 1 sequence — this story is first

### Technical requirements

- Use **JDK 25** for the verification build (you already run 25 locally per brownfield notes).
- Root command: `mvn clean package -DskipTests -Dpmd.skip=true -Dcheckstyle.skip=true` if quality plugins block on first compile (document in completion notes; Story 1.2 owns full plugin validation).
- Native libraries still come from `build.properties` — unchanged in this story.
- **SpotBugs** is known incompatible with newer bytecode in comments; do not enable SpotBugs executions as part of this story.

### Testing requirements

- **Verification:** compile-only gate — `mvn clean package -DskipTests` success is sufficient for story completion.
- **Do not** require full `mvn test` or UI tests in this story (FR-4 / FR-13 are Story 1.2+).
- If compile fails on quality plugin bytecode checks, fix only if required for `package` phase; otherwise note for Story 1.2.

### Library / framework versions (unchanged)

- Maven BOM `hdfview-bom` **3.4.1**
- SWT **3.126.0**, SLF4J **2.0.17**, JUnit **5.10.0**
- `hdf5.version` **2.0.0** / `jarhdf5` — **do not change**

### Project context reference

- [Source: `_bmad-output/project-context.md`] Build order: `repository` first
- [Source: `_bmad-output/project-context.md`] Java currently documented as 21 — update via CLAUDE.md in this story only
- [Source: `CLAUDE.md`] Authoritative dev guide for Java version statement

### Latest technical notes (JDK 25)

- JDK **25** is the target LTS alignment per PRD/architecture (released 2025).
- Maven Compiler Plugin **3.14.0** supports `--release 25` when JDK 25 is on the build JVM.
- SWT/native access warnings at **runtime** are expected until Story 1.2/1.3 adjust JVM flags — not blockers for **compile**.

### Project structure notes

- `repository/` has no local compiler override — inherits parent `25` after root change.
- `object/` uses custom `-cp` compiler args for `jarhdf5` on classpath — **do not** remove; only bump release numbers.
- Output JARs remain `libs/object-3.4.1.jar` and `libs/hdfview-3.4.1.jar` after package.

### Git intelligence summary

Recent commits (`57899e0d` … `37bb0946`) are BMAD planning artifacts only — **no JDK 25 POM edits yet**. This story is the first code change in the migration; follow existing POM comment style (Java 21 → 25 in plugin comments only where you touch lines).

### References

- [Source: `_bmad-output/planning-artifacts/epics.md` — Story 1.1]
- [Source: `_bmad-output/planning-artifacts/prds/prd-hdfview-2026-06-04/prd.md` — FR-1]
- [Source: `_bmad-output/planning-artifacts/architecture.md` — D-2, Phase 1 sequence]
- [Source: `pom.xml` — properties and pluginManagement]
- [Source: `object/pom.xml`, `hdfview/pom.xml` — module compiler overrides]

## Story completion status

Implementation complete — ready for code review.

## Dev Agent Record

### Agent Model Used

Composer (dev-story)

### Debug Log References

- First `mvn clean package` failed: `HDFView.exe` locked under `hdfview/target/dist` (clean could not delete). Removed `hdfview/target/dist` and re-ran; build succeeded.
- Ant copy warning: `HDF5\2.1.1\bin\plugin` path missing (pre-existing `build.properties` layout; non-blocking).

### Completion Notes List

- Set `maven.compiler.source` / `maven.compiler.release` and compiler + javadoc plugin levels to **25** in root, `object`, and `hdfview` POMs.
- Verified `repository/pom.xml` has no local Java 21 override (inherits parent).
- `mvn clean package -DskipTests` succeeded on Windows x86_64 with **OpenJDK Temurin 25.0.3**.
- Updated `CLAUDE.md` and `CONTRIBUTING.md` minimum Java to JDK 25.
- No HDF5 binding, CI, or dependency changes (AC #4).

### File List

- `pom.xml`
- `object/pom.xml`
- `hdfview/pom.xml`
- `CLAUDE.md`
- `CONTRIBUTING.md`

## Change Log

- 2026-06-04: Story 1.1 — JDK 25 Maven compiler baseline (POMs + docs); compile verified with `mvn clean package -DskipTests`.
