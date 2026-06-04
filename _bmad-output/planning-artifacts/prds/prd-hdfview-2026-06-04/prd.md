---
title: HDFView JDK 25 and HDF5 FFM Migration
status: draft
created: 2026-06-04
updated: 2026-06-04
sections_draft: ['0','1','2','3','4','5','6','7','8','9']
---

# PRD: HDFView JDK 25 and HDF5 FFM Migration

## 0. Document Purpose

This PRD is for **HDFView maintainers** and downstream workflow owners (architecture, epics, implementation). It defines a **two-phase** platform migration: JDK 25, then HDF5 FFM on Windows x86_64. Technical coordinates and dependency XML live in `addendum.md`; decisions are tracked in `.decision-log.md`.

## 1. Vision

HDFView today depends on JNI (`jarhdf5` / `hdf.hdf5lib`) and a custom `repository` module to wire native HDF5 libraries — fragile on Windows (paths, classifiers, native access) and misaligned with current Java platform direction.

This initiative moves the codebase in **two phases**: first **JDK 25** with no HDF API change, then **HDF5 via FFM** using official **`org.hdfgroup` 2.2.0** Maven artifacts on **Windows x86_64**, eliminating JNI for HDF5 and the `repository` module. **HDF4 remains on JNI.** Linux and macOS platform classifiers are out of scope for Phase 2.

**Primary drivers:** **maintainability** of the native-binding and build story, and **JDK alignment** (toolchain, CI, packaging). **No user-visible change** is an acceptable and intended outcome for both phases: scientists should open, inspect, and edit HDF files as they do today.

**Success for maintainers:** predictable builds, fewer JNI footguns for HDF5, and a dependency model that matches HDF Group’s Java 2.2.0 line — without regressing HDFView behavior on Windows.

## 2. Target User

### 2.1 Jobs To Be Done

- **When** I change Java or HDF5 dependencies, **I want** a Maven-native HDF5 story (no hand-installed `jarhdf5`, no `repository` bootstrap), **so** I spend less time on path, classifier, and native-access failures.
- **When** CI or a new dev machine builds HDFView, **I want** JDK 25 and (after Phase 2) `org.hdfgroup` coordinates documented for Windows x86_64, **so** builds are reproducible.
- **When** we ship a release, **I want** behavioral parity with pre-migration HDFView on Windows, **so** platform work does not surface as user regressions.
- **When** I debug HDF5 native integration, **I want** FFM instead of JNI, **so** behavior aligns with upstream `javahdf5` / `hdf5-java-ffm` and is easier to maintain.
- **When** I run GitHub Actions or local release builds, **I want** the same JDK and dependency rules as dev, **so** CI failures match what maintainers see locally.

### 2.2 Non-Users (this initiative)

- Scientists expecting new features or UI changes in HDFView
- Linux or macOS maintainers (platform classifiers deferred)
- Teams pursuing HDF4-on-FFM (out of scope)

### 2.3 Key User Journeys

- **UJ-1. Alex validates Phase 1 (JDK 25) on Windows**
  - **Persona + context:** Alex, HDFView maintainer, Windows x86_64 dev machine; HDF4 still via `build.properties` JNI paths.
  - **Entry state:** Branch on Phase 1; JDK 25 installed; existing `jarhdf5` / `repository` flow unchanged.
  - **Path:** Bump toolchain → `mvn package` → jpackage smoke (JDK 25) → launch HDFView → manual smoke → run object tests + **mandatory** `@Tag("ui")` suite (FR-13).
  - **Climax:** No functional regression vs pre-migration; app launches without `java.library.path` breakage.
  - **Resolution:** Phase 1 mergeable; Phase 2 work can start.
  - **Edge case:** Launcher must tolerate `Program Files` paths (quoted JVM args).

- **UJ-2. Alex validates Phase 2 (HDF5 FFM) on Windows**
  - **Persona + context:** Same maintainer; `repository` module removed; `org.hdfgroup` 2.2.0 on classpath.
  - **Entry state:** Phase 1 complete; Windows `windows-x86_64` classifiers declared in POM.
  - **Path:** Remove `repository` → add FFM dependencies → drop `jarhdf5` for HDF5 → rebuild → repeat UJ-1 smoke + **mandatory UI tests** on Windows.
  - **Climax:** HDF5 operations use FFM; HDF4 still JNI; build fails fast if classifier/native artifacts missing.
  - **Resolution:** Phase 2 mergeable for Windows x86_64; Linux/macOS follow-up tracked separately.

## 3. Glossary

- **Phase 1** — JDK 25 migration only; HDF5 still JNI (`jarhdf5` / `hdf.hdf5lib`); `repository` module may remain.
- **Phase 2** — HDF5 FFM migration; `org.hdfgroup` 2.2.0 stack; remove `repository`; Windows x86_64 only; HDF4 still JNI.
- **FFM (Foreign Function & Memory API)** — Java platform API for calling native libraries without JNI glue for HDF5.
- **JNI** — Java Native Interface; remains required for **HDF4** through Phase 2.
- **`repository` module** — Maven module that bootstraps `jarhdf5` into the local build; **removed in Phase 2**.
- **`jarhdf5`** — Legacy JNI Java binding for HDF5; **removed in Phase 2** for HDF5 code paths.
- **`org.hdfgroup` artifacts** — `hdf5-native`, `hdf5-szip-native`, `hdf5-zlib-native`, `hdf5-java-ffm`, `javahdf5` (see `addendum.md`).
- **Parity** — Pre-migration user-visible behavior: open, browse, edit, save HDF5 and HDF4 files on Windows without new defects.
- **Maintainer** — Engineer changing build, dependencies, or native bindings (primary PRD audience).

## 4. Features

### 4.1 Phase 1 — JDK 25 toolchain alignment

**Description:** Raise the Java platform to **JDK 25** across build, test, launch, and packaging while keeping the existing HDF5 JNI stack and module layout. Delivers maintainability and JDK alignment with **no intended user-visible change**. Realizes UJ-1.

**Functional Requirements:**

#### FR-1: Java 25 as project baseline

Maintainers can build all modules (`repository`, `object`, `hdfview`) with `maven.compiler.release` (or equivalent) set to **25**.

**Consequences (testable):**
- `mvn clean package -DskipTests` succeeds on Windows x86_64 with JDK 25.
- README / `CLAUDE.md` / launcher docs state JDK 25 minimum.

**Out of Scope:** HDF API or dependency coordinate changes.

#### FR-2: Test and quality plugins on JDK 25

Maintainers can run `mvn test` for `object` and agreed `hdfview` subsets without plugin failures attributable to Java 25.

**Consequences (testable):**
- Surefire runs with existing JVM args (`--add-opens`, `--enable-native-access=jarhdf5` or successor) documented if still required.
- PMD/Checkstyle/JaCoCo either pass or have documented skips with follow-up tickets.

#### FR-3: Launch and package on JDK 25

Maintainers can start HDFView via project launchers (`run-hdfview.bat` / `.sh`), produce the same JAR layout as today, and build **jpackage** app-images/installers using a **JDK 25** runtime.

**Consequences (testable):**
- Direct JAR launch opens the main window on Windows x86_64.
- `java.library.path` / `PATH` for HDF4 JNI remains configurable via `build.properties`.
- `mvn verify -Pjpackage-app-image` (or documented equivalent) completes on Windows x86_64 with JDK 25 bundled/configured per jpackage profiles.

#### FR-4: Behavioral parity (Phase 1)

Maintainers can execute UJ-1 smoke: HDF5 and HDF4 sample files open, edit, save without new errors vs pre-Phase-1 baseline.

**Consequences (testable):**
- Object-module `@Tag("unit")` and `@Tag("fast")` tests pass on CI and maintainer machines.
- **Mandatory:** `hdfview` `@Tag("ui")` tests pass for Phase 1 sign-off (maintainer machine with display; CI per FR-13).
- No new user-facing dialogs or data corruption in smoke scenarios.

---

### 4.2 Phase 2 — HDF5 FFM on Windows x86_64

**Description:** Replace HDF5 JNI with **`org.hdfgroup` 2.2.0** FFM artifacts; remove the **`repository`** module and **`jarhdf5`** dependency; keep **HDF4 on JNI**. Windows **x86_64** classifiers only. **No intended user-visible change** for HDF5 workflows scientists use today. Realizes UJ-2.

**Functional Requirements:**

#### FR-5: Maven-native HDF5 stack

Maintainers declare `org.hdfgroup` dependencies (`hdf5-native`, `hdf5-szip-native`, `hdf5-zlib-native`, `hdf5-java-ffm` with `windows-x86_64` classifier, `javahdf5` 2.2.0) in the BOM/module POMs per `addendum.md`.

**Consequences (testable):**
- Clean build resolves artifacts from Maven (local or remote); no manual copy into `repository/lib`.
- Missing classifier fails at dependency resolution with a clear Maven error.

#### FR-6: Remove `repository` module

Maintainers can build the project with **`repository` removed** from the root `pom.xml` modules list and no remaining references to `jarhdf5` install-file goals for HDF5.

**Consequences (testable):**
- Root `pom.xml` module order is `object` → `hdfview` (or equivalent without `repository`).
- Documentation no longer instructs building `repository` first for HDF5.

#### FR-7: HDF5 code paths use FFM bindings (big-bang)

Maintainers replace **`hdf.hdf5lib`** usage with **`javahdf5` / `hdf5-java-ffm`** in a **big-bang** cutover (no long-lived JNI adapter layer for HDF5).

**Consequences (testable):**
- No compile-time dependency on `jarhdf5` or `hdf.hdf5lib` for HDF5.
- `object` and `hdfview` HDF5 call sites compile against `org.hdfgroup` 2.2.0 APIs only.
- HDF5 read/write smoke in UJ-2 succeeds on Windows x86_64.

**Out of Scope:** Rewriting HDF4 bindings; non-Windows classifiers; incremental dual-stack JNI+FFM for HDF5.

#### FR-8: HDF4 remains JNI

Maintainers can still open and edit HDF4 files using existing JNI + `build.properties` native paths for HDF4 on Windows.

**Consequences (testable):**
- UJ-2 HDF4 smoke passes after Phase 2.
- `hdf.lib.dir` (or successor) documented for HDF4-only natives.

#### FR-9: Remove HDF5 paths from build configuration

Maintainers no longer set `hdf5.lib.dir` or `hdf5.plugin.dir` in `build.properties` (or template); HDF5 natives and plugins resolve via `org.hdfgroup` Maven artifacts only.

**Consequences (testable):**
- `build.properties` template and docs omit `hdf5.lib.dir` / `hdf5.plugin.dir`.
- Fresh Windows dev setup builds and runs HDF5 without a system HDF5 installer path for HDF5 (HDF4 `hdf.lib.dir` may still be required).
- Maven enforcer / copy steps that referenced `hdf5.lib.dir` are removed or gated to HDF4-only.

#### FR-10: Behavioral parity (Phase 2)

Maintainers can sign off Phase 2 when UJ-2 smoke, object-module HDF5 tests, and **mandatory UI tests** match Phase 1 behavior for the same sample files.

**Consequences (testable):**
- No new crashes on common datatypes (including compound, vlen, refs; Float16/BFLOAT16 per existing test coverage).
- **Mandatory:** full `@Tag("ui")` suite (or agreed Phase 2 subset documented in stories) passes on Windows x86_64 before merge.
- Regression list documented in epic/story acceptance criteria.

**Feature-specific NFRs:**
- **Reliability:** Prefer build-time failure over JVM crash on missing natives.
- **Maintainability:** Single version property for HDF5 Java stack (`2.2.0`) aligned across modules.

---

### 4.3 Cross-cutting — Documentation and CI

**Description:** Keep maintainer docs and CI consistent with each phase. Supports all JTBD.

#### FR-11: Phase-aligned documentation

Maintainers can follow updated build/run instructions per phase in `CLAUDE.md`, `project-context.md`, and launcher scripts.

**Consequences (testable):**
- Phase 1 docs reference JDK 25 only.
- Phase 2 docs reference `org.hdfgroup` coordinates and removal of `repository`.

#### FR-12: GitHub Actions alignment (all platforms)

Maintainers update **Windows, macOS, and Linux** GitHub Actions workflows to **JDK 25** in Phase 1 (existing jobs updated, not replaced). Phase 2 adds **Windows x86_64 FFM** validation; non-Windows jobs stay on JDK 25 with a documented test scope until Linux/macOS FFM classifiers exist.

**Consequences (testable):**
- **Phase 1:** Windows, macOS, and Linux CI jobs use **JDK 25** and pass compile + object-module tests (+ UI tests per FR-13 where runners support display).
- **Phase 2:** **Windows** CI uses **`org.hdfgroup` 2.2.0** FFM dependencies (no `repository` / `jarhdf5` bootstrap) and passes object + **mandatory UI** tests.
- **Phase 2:** macOS/Linux CI remain on JDK 25; jobs either pass with documented exclusions for Windows-only FFM artifacts or are explicitly scoped in stories (no silent green builds that skip HDF5).

#### FR-13: Mandatory UI tests (both phases)

Maintainers treat **`@Tag("ui")`** SWTBot tests as a **required** gate for phase completion, not optional smoke.

**Consequences (testable):**
- Phase 1 and Phase 2 definition-of-done lists include UI test execution and results.
- CI or maintainer runbook documents how UI tests run per OS (e.g. display on Windows maintainer box; headless/display service on Linux CI if used).
- Failed UI tests block phase merge same as unit test failures.

## 5. Non-Goals (Explicit)

- User-visible feature work (new menus, formats, UX redesign)
- HDF4 FFM or removal of HDF4 JNI
- Linux or macOS `org.hdfgroup` classifiers in this PRD
- Performance tuning or benchmark campaigns unless regressions are found
- Rewriting the entire `object` module API surface unrelated to HDF5 binding swap
- Dropping HDF4 support

## 6. MVP Scope

Scope is **two phased MVPs**, not a single cutover.

### 6.1 In Scope

**Phase 1 (must ship before Phase 2 starts)**
- JDK 25 in POMs, launchers, **jpackage**, and maintainer docs
- CI (Windows, macOS, Linux) on JDK 25 per FR-12
- Parity per FR-4 / FR-13 / UJ-1
- JNI/`jarhdf5`/`repository` unchanged

**Phase 2 (depends on Phase 1)**
- `org.hdfgroup` 2.2.0 FFM stack, Windows x86_64 only
- Remove `repository` module and HDF5 JNI (`jarhdf5`)
- HDF4 JNI retained
- Parity per FR-10 / FR-13 / UJ-2 (mandatory UI tests on Windows)

### 6.2 Out of Scope for MVP

| Item | Reason |
|------|--------|
| Linux/macOS FFM classifiers | Deferred; Windows-first |
| HDF4 FFM | Separate future initiative |
| User-facing release notes beyond “maintenance” | No user-visible change goal |
| Linux/macOS FFM in Phase 2 CI | Windows-only FFM artifacts; non-Windows CI scoped per FR-12 |

## 7. Success Metrics

**Primary**
- **SM-1:** Phase 1 complete — FR-1–FR-4, FR-12 (all OS JDK 25), FR-13 consequences met. Validates FR-1–FR-4, FR-12, FR-13.
- **SM-2:** Phase 2 complete — 100% of FR-5–FR-10 consequences met on Windows x86_64. Validates FR-5–FR-10.

**Secondary**
- **SM-3:** Zero P1/P2 user-reported regressions attributable to migration in the first 30 days post-Phase-2 merge (or “no reports” if not shipped to external users). Validates FR-4, FR-10.
- **SM-4:** New Windows dev onboarding no longer documents `repository` or manual `jarhdf5` install after Phase 2. Validates FR-6, FR-9.
- **SM-5:** Phase 1 jpackage app-image build succeeds on JDK 25. Validates FR-3.
- **SM-6:** Mandatory UI test suite green for phase sign-off. Validates FR-13, FR-4, FR-10.

**Counter-metrics (do not optimize)**
- **SM-C1:** Lines of code changed — prefer minimal diffs; do not expand scope for refactor credit.
- **SM-C2:** JDK feature adoption beyond 25 — not required for success.

## 8. Open Questions

_None — resolved in `.decision-log.md`. Architecture/epics should detail: exact UI test list per phase, Linux CI display strategy for SWTBot, and macOS/Linux job behavior after Phase 2 FFM (Windows-only artifacts)._

## 9. Assumptions Index

- `[ASSUMPTION: org.hdfgroup 2.2.0 APIs support all HDF5 code paths HDFView uses today.]` — confirm during architecture / Phase 2 spike
- `[ASSUMPTION: Phase 2 macOS/Linux CI can be kept meaningful without Windows FFM classifiers (scoped tests or temporary job config).]` — confirm in architecture / CI stories
