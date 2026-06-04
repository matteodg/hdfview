# Story 1.2: Validate tests and quality plugins on JDK 25

Status: ready-for-dev

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

- [ ] Task 1: Run object-module unit/fast tests on JDK 25 (AC: #1)
  - [ ] Run `mvn test -pl object` (build `repository` + `object` first if needed: `mvn install -pl repository,object -DskipTests`)
  - [ ] Confirm Surefire `argLine` in `object/pom.xml` (L188–195) is applied: `--add-opens` x4 + `--enable-native-access=jarhdf5` + `-Djava.library.path=${platform.hdf.lib}`
  - [ ] Record pass/fail counts; classify any failure as pre-existing vs JDK-25-introduced
- [ ] Task 2: Validate PMD on JDK 25 (AC: #2, #3)
  - [ ] Align `pom.xml` `maven-pmd-plugin` `<targetJdk>21</targetJdk>` → `25` (L384)
  - [ ] Run PMD (`mvn pmd:check -pl object,hdfview` or via quality profile / `scripts/validate-quality.sh`)
  - [ ] Note: `<skipPmdError>true</skipPmdError>` and `<typeResolution>false</typeResolution>` already mitigate ASM/Java 25 issues (L398–400) — keep them
- [ ] Task 3: Validate Checkstyle on JDK 25 (AC: #2, #4)
  - [ ] Run Checkstyle (v3.3.1 / checkstyle 10.12.5, L405–413); confirm it parses JDK 25 sources
  - [ ] Update stale "Java 21 compatible" comment on the Checkstyle plugin block (L404) to JDK 25
- [ ] Task 4: Validate JaCoCo on JDK 25 (AC: #2)
  - [ ] Run `mvn test` with JaCoCo agent active; confirm report generation and that thresholds (~60% line / 50% branch) are evaluated, not skipped
  - [ ] Do NOT lower thresholds; if instrumentation fails on JDK 25, document and open a follow-up
- [ ] Task 5: Comment hygiene + documentation (AC: #2, #4)
  - [ ] Fix stale "Java 21" comments only in the Surefire/PMD/Checkstyle blocks you touch
  - [ ] Document any plugin skip + follow-up note in the story Completion Notes
  - [ ] Correct the SpotBugs comment class-file major version if touched (major 69 = Java 25, not 68)

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

Ultimate context engine analysis completed — comprehensive developer guide created. Status: **ready-for-dev**.

## Dev Agent Record

### Agent Model Used

{{agent_model_name_version}}

### Debug Log References

### Completion Notes List

### File List
