---
title: HDFView JDK 25 and HDF5 FFM Migration
status: final
created: 2026-06-04
updated: 2026-06-04
sections_completed:
  - document_purpose
  - vision
  - target_user
  - glossary
  - features
  - non_goals
  - mvp_scope
  - success_metrics
  - open_questions
  - assumptions_index
rule_count: 13
optimized_for_llm: true
---

# PRD: HDFView JDK 25 and HDF5 FFM Migration

## 0. Document Purpose

This PRD is for **HDFView maintainers** and downstream owners (architecture, epics, implementation). It defines a **two-phase** platform migration:

1. **Phase 1** — JDK **25** only (JNI/`jarhdf5` unchanged).
2. **Phase 2** — HDF5 **FFM** via **`org.hdfgroup` 2.2.0** on **Windows x86_64**; remove the **`repository`** module; **HDF4 stays JNI**.

Maven coordinates and dependency XML are in `addendum.md`. Decisions are in `.decision-log.md`. FRs use stable IDs **FR-1** through **FR-13** for epics and stories.

## 1. Vision

HDFView depends on JNI (`jarhdf5` / `hdf.hdf5lib`) and a custom **`repository`** module to wire HDF5 natives — fragile on Windows and misaligned with modern Java.

This initiative moves the codebase in two phases: **JDK 25** first (no HDF API change), then **HDF5 via FFM** using official **`org.hdfgroup` 2.2.0** Maven artifacts on **Windows x86_64**, eliminating HDF5 JNI and **`repository`**. **HDF4 remains JNI.** Linux/macOS FFM classifiers are out of scope for Phase 2.

**Drivers:** **maintainability** and **JDK alignment**. **No user-visible change** is acceptable for both phases.

**Success for maintainers:** predictable builds, fewer HDF5 native footguns, alignment with HDF Group Java **2.2.0**, and no regression in open/edit/save behavior on Windows.

## 2. Target User

### 2.1 Jobs To Be Done

- **When** I change Java or HDF5 dependencies, **I want** a Maven-native HDF5 story (no `jarhdf5`, no `repository` bootstrap), **so** I avoid path and classifier failures.
- **When** CI or a new machine builds HDFView, **I want** JDK **25** and documented `org.hdfgroup` coordinates (Phase 2, Windows), **so** builds are reproducible.
- **When** we ship, **I want** behavioral parity on Windows, **so** platform work does not become user regressions.
- **When** I debug HDF5 integration, **I want** FFM aligned with `javahdf5` / `hdf5-java-ffm`, **so** maintenance matches upstream.
- **When** I run GitHub Actions, **I want** the same JDK and dependency rules as local dev, **so** CI matches maintainer failures.

### 2.2 Non-Users (this initiative)

- Scientists expecting new HDFView features or UI changes
- Teams expecting **Linux/macOS FFM** in Phase 2 (deferred)
- Teams pursuing **HDF4-on-FFM** (out of scope)

### 2.3 Key User Journeys

- **UJ-1. Alex validates Phase 1 (JDK 25) on Windows**
  - **Context:** Maintainer, Windows x86_64; HDF4 via `build.properties` JNI; `jarhdf5` / `repository` unchanged.
  - **Path:** Toolchain → `mvn package` → jpackage (JDK 25) → launch → manual smoke → object tests + mandatory `@Tag("ui")` (FR-13).
  - **Climax:** No regression vs baseline; launch works with quoted `Program Files` JVM args.
  - **Resolution:** Phase 1 mergeable.

- **UJ-2. Alex validates Phase 2 (HDF5 FFM) on Windows**
  - **Context:** `repository` removed; `org.hdfgroup` 2.2.0 on classpath.
  - **Path:** FFM deps → drop `jarhdf5` → rebuild → UJ-1 smoke + mandatory UI tests on Windows.
  - **Climax:** HDF5 via FFM; HDF4 JNI; build fails fast if natives/classifiers missing.
  - **Resolution:** Phase 2 mergeable on Windows x86_64.

## 3. Glossary

- **Phase 1** — JDK 25 only; HDF5 still JNI; `repository` may remain.
- **Phase 2** — HDF5 FFM; `org.hdfgroup` 2.2.0; remove `repository`; Windows x86_64; HDF4 JNI.
- **FFM** — Foreign Function & Memory API for HDF5 (no JNI for HDF5 in Phase 2).
- **JNI** — Required for **HDF4** through Phase 2.
- **`repository` module** — Bootstraps `jarhdf5`; **removed in Phase 2**.
- **`jarhdf5`** — Legacy HDF5 JNI binding; **removed in Phase 2**.
- **`org.hdfgroup` artifacts** — See `addendum.md`.
- **Parity** — Same user-visible open/browse/edit/save for HDF5 and HDF4 on Windows.
- **Maintainer** — Primary PRD audience.

## 4. Features

### 4.1 Phase 1 — JDK 25 toolchain alignment

**Description:** JDK **25** across build, test, launch, jpackage, and CI; HDF5 JNI unchanged. No intended user-visible change. Realizes **UJ-1**.

#### FR-1: Java 25 as project baseline

Maintainers build all modules (`repository`, `object`, `hdfview`) with `maven.compiler.release` **25**.

**Consequences:**
- `mvn clean package -DskipTests` succeeds on Windows x86_64 with JDK 25.
- Docs state JDK 25 minimum.

**Out of Scope:** HDF API or dependency changes.

#### FR-2: Test and quality plugins on JDK 25

Maintainers run `mvn test` without Java-25 plugin failures.

**Consequences:**
- Surefire JVM args documented (`--add-opens`, `--enable-native-access=jarhdf5` or successor).
- PMD/Checkstyle/JaCoCo pass or have documented skips.

#### FR-3: Launch and package on JDK 25

Maintainers launch via `run-hdfview.bat` / `.sh` and build **jpackage** app-images with JDK **25**.

**Consequences:**
- JAR launch opens main window on Windows x86_64.
- HDF4 JNI paths via `build.properties`.
- `mvn verify -Pjpackage-app-image` (or equivalent) succeeds on Windows x86_64.

#### FR-4: Behavioral parity (Phase 1)

UJ-1 smoke: HDF5/HDF4 open, edit, save without new errors.

**Consequences:**
- `@Tag("unit")` / `@Tag("fast")` pass on CI and locally.
- **Mandatory** `@Tag("ui")` pass for sign-off (FR-13).
- No new user-facing errors in smoke scenarios.

---

### 4.2 Phase 2 — HDF5 FFM on Windows x86_64

**Description:** HDF5 FFM via **`org.hdfgroup` 2.2.0**; remove **`repository`** and **`jarhdf5`**; HDF4 JNI. Windows **x86_64** only. Realizes **UJ-2**.

#### FR-5: Maven-native HDF5 stack

Declare `org.hdfgroup` dependencies per `addendum.md` (`windows-x86_64` classifiers).

**Consequences:**
- Build resolves from Maven; no `repository/lib` copy.
- Missing classifier fails at resolution with clear Maven error.

#### FR-6: Remove `repository` module

Build without **`repository`** in root `pom.xml` or `jarhdf5` install-file goals for HDF5.

**Consequences:**
- Modules: `object` → `hdfview`.
- Docs no longer require building `repository` first.

#### FR-7: HDF5 FFM bindings (big-bang)

Replace **`hdf.hdf5lib`** with **`javahdf5` / `hdf5-java-ffm`** in one cutover (no JNI adapter for HDF5).

**Consequences:**
- No `jarhdf5` / `hdf.hdf5lib` for HDF5.
- HDF5 smoke on Windows x86_64 passes.

**Out of Scope:** HDF4 FFM; non-Windows classifiers; dual-stack JNI+FFM.

#### FR-8: HDF4 remains JNI

HDF4 open/edit via JNI + `hdf.lib.dir` on Windows.

**Consequences:**
- UJ-2 HDF4 smoke passes.
- Docs cover HDF4-only native paths.

#### FR-9: Remove HDF5 paths from build configuration

Drop **`hdf5.lib.dir`** and **`hdf5.plugin.dir`** from `build.properties` template; HDF5 via Maven only.

**Consequences:**
- Template/docs omit HDF5 path properties.
- Enforcer/copy steps for `hdf5.lib.dir` removed or HDF4-only.

#### FR-10: Behavioral parity (Phase 2)

Sign-off when UJ-2 smoke, object HDF5 tests, and **mandatory UI tests** match Phase 1.

**Consequences:**
- No new crashes on common datatypes (compound, vlen, refs, Float16/BFLOAT16 per existing coverage).
- Full `@Tag("ui")` suite (or documented subset) green on Windows before merge.

**Feature NFRs:** Build-time failure preferred over JVM crash; single **2.2.0** version property for HDF5 Java stack.

---

### 4.3 Cross-cutting — Documentation and CI

#### FR-11: Phase-aligned documentation

Update `CLAUDE.md`, `project-context.md`, launchers per phase.

**Consequences:**
- Phase 1: JDK 25 only.
- Phase 2: `org.hdfgroup` coords; no `repository`.

#### FR-12: GitHub Actions alignment (all platforms)

**Phase 1:** Windows, macOS, Linux → **JDK 25**. **Phase 2:** Windows → FFM **2.2.0**; macOS/Linux → JDK 25 with **documented** test scope (no silent skip of HDF5).

**Consequences:**
- Phase 1: all OS jobs on JDK 25; object tests + UI tests per FR-13 where display is available.
- Phase 2: Windows FFM + mandatory UI; macOS/Linux scoped in stories.

#### FR-13: Mandatory UI tests (both phases)

`@Tag("ui")` SWTBot tests are **required** for phase completion.

**Consequences:**
- DoD includes UI test results.
- Runbook for UI tests per OS.
- Failed UI tests block merge.

## 5. Non-Goals (Explicit)

- User-visible features or UX redesign
- HDF4 FFM
- Linux/macOS `org.hdfgroup` classifiers in Phase 2
- Performance campaigns unless regressions found
- Unrelated `object` API rewrites
- Dropping HDF4 support

## 6. MVP Scope

### 6.1 In Scope

**Phase 1:** JDK 25 (POMs, launchers, jpackage, docs); CI all OS on JDK 25; FR-1–4, FR-11–13; JNI/`repository` unchanged.

**Phase 2:** `org.hdfgroup` 2.2.0 Windows x86_64; remove `repository`/`jarhdf5`; HDF4 JNI; FR-5–10, FR-11–13 on Windows.

### 6.2 Out of Scope for MVP

| Item | Reason |
|------|--------|
| Linux/macOS FFM classifiers | Windows-first |
| HDF4 FFM | Future initiative |
| Marketing release notes | No user-visible change |
| macOS/Linux FFM in Phase 2 CI | Scoped jobs per FR-12 |

## 7. Success Metrics

**Primary**
- **SM-1:** Phase 1 — FR-1–4, FR-12, FR-13 met.
- **SM-2:** Phase 2 — FR-5–10 met on Windows x86_64.

**Secondary**
- **SM-3:** No P1/P2 migration regressions (30 days post-Phase 2, or N/A if not externally shipped).
- **SM-4:** Onboarding docs omit `repository` / manual `jarhdf5` after Phase 2.
- **SM-5:** Phase 1 jpackage on JDK 25 succeeds.
- **SM-6:** Mandatory UI suite green per phase.

**Counter-metrics**
- **SM-C1:** Do not optimize LOC changed.
- **SM-C2:** Do not require JDK features beyond 25.

## 8. Open Questions

None for PRD sign-off. **Deferred to architecture/epics:** UI test list per phase; Linux CI display for SWTBot; macOS/Linux CI behavior after Phase 2 FFM.

## 9. Assumptions Index

| Assumption | Confirm in |
|------------|------------|
| `org.hdfgroup` 2.2.0 covers all HDF5 paths HDFView uses | Architecture / Phase 2 spike |
| Phase 2 macOS/Linux CI remain meaningful without Windows FFM classifiers | Architecture / CI stories |

---

## Usage Guidelines

**For AI agents:** Read this PRD before architecture or implementation. Follow FR IDs. Prefer restrictive interpretation. Technical XML in `addendum.md` only.

**For humans:** Update when stack or CI policy changes. Review assumptions after Phase 2 spike.

**Downstream:** `bmad-create-architecture` → `bmad-create-epics-and-stories` (Phase 1 epic before Phase 2).

Last Updated: 2026-06-04
