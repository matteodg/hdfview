---
baseline_commit: 9e1ff17bd877012af4f605846e58cdcbcae15b02
---

# Story 1.2: Validate tests and quality plugins on JDK 25

Status: review

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As an **HDFView maintainer**,
I want unit tests and quality plugins (PMD, Checkstyle, JaCoCo) to run on **JDK 25**,
so that CI and local development use the same JVM rules after the compiler baseline moved to 25 (Story 1.1).

## Acceptance Criteria

1. **Given** Story 1.1 is complete, **when** `mvn test -pl object` runs on Windows x86_64 with JDK 25 using the documented Surefire `--add-opens` and `--enable-native-access=jarhdf5` args, **then** object `@Tag("unit")` and `@Tag("fast")` tests pass (or pre-existing failures are documented, not newly introduced by JDK 25). (FR-2, D-14)
2. **Given** the quality plugins run on JDK 25, **then** **PMD**, **Checkstyle**, and **JaCoCo** either pass or have a **documented skip with a follow-up issue/note** — no silent disabling and no lowering of JaCoCo thresholds. (FR-2, anti-pattern: do not lower coverage to green a migration)
3. **Given** PMD `targetJdk` was left at `21` by Story 1.1, **then** it is aligned to **25** (or explicitly documented why it must stay) so PMD analyzes at the same language level as the compiler. (D-2 consistency; carried over from Story 1.1 review)
4. **Given** Surefire/native-access JVM args are validated, **then** any stale "Java 21" comments in the touched Surefire/quality plugin blocks are corrected to reflect JDK 25 (comment hygiene only where lines are touched).
5. **Given** this is Phase 1 Story 1.2, **then** no HDF5 binding changes, no `org.hdfgroup` 2.2.0 dependencies, no CI workflow edits (Story 1.5), and no launcher/jpackage edits (Stories 1.3/1.4).

## Tasks / Subtasks

- [x] Task 1: Run object-module unit/fast tests on JDK 25 (AC: #1)
  - [x] Run `mvn test -pl object` (built `repository` + `object` first)
  - [x] Confirm Surefire `argLine` in `object/pom.xml` applied: `--add-opens` x4 + `--enable-native-access=jarhdf5` + `-Djava.library.path`
  - [x] Record results: **163 tests, 0 failures, 0 errors** after two pre-existing env/config fixes (see Completion Notes)
- [x] Task 2: Validate PMD on JDK 25 (AC: #2, #3)
  - [x] Align `pom.xml` `maven-pmd-plugin` `<targetJdk>21</targetJdk>` → `25`
  - [x] Bump pmd-core/pmd-java `7.7.0` → `7.17.0` (Java 25 support added in PMD 7.16.0; 7.7.0 rejected targetJdk 25)
  - [x] PMD runs clean on JDK 25, BUILD SUCCESS (violations are non-failing warnings; `failOnViolation=false`)
- [x] Task 3: Validate Checkstyle on JDK 25 (AC: #2, #4)
  - [x] Checkstyle (v3.3.1 / checkstyle 10.12.5) parses JDK 25 sources — warnings only, no failure
  - [x] Updated stale "Java 21 compatible" comment on the Checkstyle plugin block to JDK 25
- [x] Task 4: Validate JaCoCo on JDK 25 (AC: #2)
  - [x] JaCoCo plugin (0.8.12) executes without JDK-version errors; `verify` is green
  - [x] **Documented skip:** object-module coverage is not collected because the module's literal Surefire `<argLine>` overrides JaCoCo's agent argLine (pre-existing). Did NOT lower thresholds. Follow-up logged.
- [x] Task 5: Comment hygiene + documentation (AC: #2, #4)
  - [x] Fixed stale "Java 21" comments in PMD/Checkstyle/SpotBugs blocks; SpotBugs class-file major corrected to 69 (Java 25)
  - [x] Cleaned 3 stale PMD ruleset excludes surfaced by the 7.17 bump (`DataflowAnomalyAnalysis`, `ExcessiveClassLength` removed; `JUnitTestsShouldIncludeAssert` → `UnitTestShouldIncludeAssert`)
  - [x] Re-asserted PMD/Checkstyle "JDK 25 compatible" in CLAUDE.md (Story 1.1 had softened it pending this validation)

## Dev Notes

### Story scope guardrails (Phase 1)

- **In scope:** Test execution validation + quality-plugin (PMD/Checkstyle/JaCoCo) validation on JDK 25; PMD `targetJdk` alignment; comment hygiene in touched plugin blocks.
- **Out of scope:** CI workflow JDK bump (Story 1.5), launchers/`build-dev.sh` (1.3), jpackage (1.4), full `project-context.md` / Testing-Guide refresh (1.7), HDF5 FFM (Epic 2).
- **Do NOT** lower JaCoCo thresholds, disable quality gates silently, or remove test `@Tag`s to make CI green. [Source: architecture.md enforcement rule 5; project-context.md testing rules]

### Carried over from Story 1.1 code review (deferred → now in scope here)

These were logged in `deferred-work.md` for Story 1.2:
- PMD `<targetJdk>21</targetJdk>` not bumped [pom.xml:384] → **Task 2**
- Quality ruleset metadata says "Java 21" [pmd-rules.xml:7, checkstyle-rules.xml:9] → optional comment hygiene
- Stale SpotBugs/quality comments in root POM (incl. wrong class-file major 68) [pom.xml:357,366,404,524-530,...] → **Task 5** (only where touched; SpotBugs executions stay disabled)
- Surefire `--enable-native-access`/`--add-opens` argLine validation → **Task 1**

### Current state (key files)

| File | Location | Current | Action |
|------|----------|---------|--------|
| `object/pom.xml` | Surefire `argLine` L184–197 | `--add-opens` x4 + `--enable-native-access=jarhdf5` | Validate runs on JDK 25; no change unless failing |
| `pom.xml` | `maven-pmd-plugin` L366–402 | `<targetJdk>21</targetJdk>`, `skipPmdError=true`, `typeResolution=false` | Bump targetJdk → 25; keep ASM mitigations |
| `pom.xml` | `maven-checkstyle-plugin` L404–413 | v3.3.1 / checkstyle 10.12.5, "Java 21" comment | Validate; fix comment |
| `pom.xml` | JaCoCo block | ~60%/50% targets | Validate report + thresholds |
| `pom.xml` | SpotBugs L355–364 | executions disabled, "Java 21" comments | Leave disabled; fix comment only if touched |

**Preserve:** native-access flags, `skipPmdError`, `typeResolution=false`, PMD excludes for `hdf/hdf5lib`/`hdf4lib`/tests, JaCoCo thresholds, non-modular classpath build.

### Architecture compliance

- [Source: `architecture.md`] D-14: Phase 1 Surefire keeps `--add-opens` + `--enable-native-access=jarhdf5` (document updates)
- [Source: `architecture.md`] D-2: compiler release 25 — PMD `targetJdk` should match
- [Source: `architecture.md`] Phase 1 sequence: POM JDK 25 → CI → Surefire/JVM → launchers/jpackage; this story is the Surefire/quality validation step
- [Source: `architecture.md`] Enforcement: "Do not lower JaCoCo thresholds to green migration PRs"

### Technical requirements

- Build JVM must be **JDK 25** (Temurin 25.0.3 verified in Story 1.1).
- `object` tests need native libs on path via `build.properties` `platform.hdf.lib`; ensure HDF5 2.1.1 / HDF4 4.3.1 native dirs resolve (a missing `…\HDF5\2.1.1\bin\plugin` warning is non-fatal, observed in Story 1.1).
- `--enable-native-access=jarhdf5` is a JDK 22+ flag and is correct under JDK 25; without it, native-access warnings/errors may surface.
- If a quality plugin genuinely cannot run on JDK 25, **skip with a documented follow-up** rather than failing the story — but capture it explicitly (AC #2).

### Testing requirements

- Primary gate: `mvn test -pl object` green for `@Tag("unit")` + `@Tag("fast")`. [Source: project-context.md testing rules]
- UI tests (`hdfview/src/test/java/uitest/`) require a real display and are **Story 1.6** — do not attempt to make them a gate here.
- Run single test if isolating failures: `mvn test -pl object -Dtest=TestClassName`.
- JUnit 5 primary; JUnit 4 vintage still present.

### Library / framework versions (unchanged)

- PMD plugin 3.25.0 / pmd-core+pmd-java 7.7.0; Checkstyle plugin 3.3.1 / checkstyle 10.12.5; SpotBugs 4.8.6 (disabled)
- JUnit 5.10.0 (+ vintage), SLF4J 2.0.17, SWT 3.126.0, BOM 3.4.1
- `hdf5.version` 2.0.0 / `jarhdf5` — **do not change**

### Previous story intelligence (Story 1.1)

- `mvn clean package` first failed because `HDFView.exe` under `hdfview/target/dist` was locked — close any running HDFView before clean.
- Story 1.1 softened CLAUDE.md PMD/Checkstyle wording (removed unverified "JDK 25 compatible"); this story is where that compatibility is actually validated and can be re-asserted if it passes.
- PMD already carries a comment "Skip auxclasspath to avoid ASM errors with Java 25 dependencies" (pom.xml:398) — Java 25 was partially anticipated.

### Project context reference

- [Source: `project-context.md`] Surefire JVM args list (4 `--add-opens` + `--enable-native-access=jarhdf5`)
- [Source: `project-context.md`] Run quality via `scripts/validate-quality.sh` before claiming done
- [Source: `project-context.md`] Object tests under `object/src/test/java/object/`, tag unit/fast

### References

- [Source: `epics.md` — Story 1.2]
- [Source: `prd.md` — FR-2]
- [Source: `architecture.md` — D-2, D-14, Phase 1 sequence, enforcement rules]
- [Source: `deferred-work.md` — items deferred from Story 1.1 to Story 1.2]
- [Source: `pom.xml` — PMD/Checkstyle/SpotBugs/JaCoCo blocks; `object/pom.xml` — Surefire argLine]

## Story completion status

Implementation complete — ready for code review.

## Dev Agent Record

### Agent Model Used

Composer (dev-story)

### Debug Log References

- Object tests run 1: **29 errors**, all `NoClassDefFound Could not initialize class hdf.hdf5lib.H5 / hdf.object.h5.H5File`. Root cause: `build.properties` `platform.hdf.lib` was **empty**, so `-Djava.library.path` was blank and native HDF5 DLLs were not found. (Pre-existing env config, not JDK 25.)
- Object tests run 2 (after setting `platform.hdf.lib`): forked VM crash — `Error: Could not find or load main class Files.HDF_Group...`. Root cause: unquoted `-Djava.library.path=${platform.hdf.lib}` with spaces in `C:/Program Files/...`. (Latent Surefire bug; would also fail on JDK 21.)
- Object tests run 3 (after quoting the argLine path): **163 tests, 0 failures, 0 errors, BUILD SUCCESS.**
- PMD run 1: `Unsupported targetJdk value '25'` with pmd 7.7.0. Fixed by bumping pmd-core/pmd-java to 7.17.0 (Java 25 support since PMD 7.16.0).
- JaCoCo: `verify` green but no `.exec`/report produced for object — agent argLine overridden by module Surefire config (documented skip).

### Completion Notes List

- **AC #1 PASS:** `mvn test -pl object` → 163/163 pass on Temurin 25.0.3 with documented Surefire `--add-opens` + `--enable-native-access=jarhdf5`.
- **AC #2 PASS:** PMD + Checkstyle pass (warnings only, non-failing); JaCoCo runs without JDK-25 error with a **documented skip + follow-up** (no thresholds lowered).
- **AC #3 PASS:** PMD `targetJdk` aligned to 25; required pmd-core/pmd-java bump 7.7.0 → 7.17.0.
- **AC #4 PASS:** No HDF5 binding, `org.hdfgroup`, CI, launcher, or jpackage changes.
- **Two pre-existing fixes were required to run the tests at all:** (1) populate local `build.properties` `platform.hdf.lib`; (2) quote `-Djava.library.path` in the object Surefire argLine. Both are unrelated to JDK 25 but blocked AC #1.

#### Follow-ups created (not in this story)

- **JaCoCo coverage not collected for `object`:** the module's literal Surefire `<argLine>` overrides JaCoCo's agent `argLine`, so no coverage is measured (and JaCoCo 0.8.12 predates JDK 25 class-file v69). Wiring `@{argLine}` + bumping JaCoCo ≥0.8.13 should be a dedicated story; doing it here risks tripping the 60%/50% gate. → logged in `deferred-work.md`.
- **`hdfview/pom.xml` Surefire argLine has the same unquoted `-Djava.library.path`** latent crash; left for Story 1.6 (UI tests). → logged in `deferred-work.md`.

### File List

- `pom.xml` — PMD `targetJdk` 21→25; pmd-core/pmd-java 7.7.0→7.17.0; PMD/Checkstyle/SpotBugs comment hygiene (incl. class-file major 68→69)
- `object/pom.xml` — quoted `-Djava.library.path` in Surefire argLine
- `pmd-rules.xml` — removed/renamed 3 stale excludes for PMD 7; description "Java 21"→"JDK 25"
- `CLAUDE.md` — re-asserted PMD/Checkstyle "JDK 25 compatible"
- `build.properties` — set local `platform.hdf.lib` (LOCAL/machine-specific; not for commit per project-context.md)

## Change Log

- 2026-06-04: Story 1.2 — Validated object tests (163/163) and PMD/Checkstyle/JaCoCo on JDK 25. PMD `targetJdk`=25 (pmd 7.17.0); quoted Surefire library path; JaCoCo object-coverage documented as follow-up.
