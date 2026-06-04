## Context

HDFView is a multi-module Maven project (`repository`, `object`, `hdfview`) standardized on Java 21 since the Phase 1 Maven migration. Java version is centralized in the root `pom.xml` (`maven.compiler.source` / `release` = 21) with duplicate compiler plugin blocks in `object/pom.xml` and `hdfview/pom.xml`. CI, installer workflows, launcher scripts, and documentation all reference Java 21. Quality tooling (PMD 7.x, Checkstyle 10.12+) was chosen for Java 21 compatibility; SpotBugs remains disabled in default lifecycle due to Java 21 bytecode gaps.

The project uses a non-modular classpath build (`forceLegacyJavacApi`, module-info disabled) for SWT compatibility. Native HDF libraries and `jpackage` profiles depend on the JDK used at build time.

## Goals / Non-Goals

**Goals:**

- Single authoritative Java version: **25** across Maven properties, module POMs, filtered version metadata, CI, and developer scripts
- Green `mvn clean compile`, `mvn test` (where environment allows), and quality workflows on JDK 25
- Validated `jpackage` app-image and installer paths on at least one platform per OS family
- Updated contributor documentation reflecting the new minimum

**Non-Goals:**

- Re-enabling the Java module system or refactoring package structure for JPMS
- Upgrading unrelated dependencies (SWT, HDF natives, JUnit) unless required for JDK 25 compatibility
- Changing supported HDF file formats or application features
- Mandating JDK 25 for end users of pre-built installers beyond what `jpackage` bundles (runtime follows installer build JDK)

## Decisions

### 1. Target JDK 25 with Temurin in CI

**Choice:** Set `java-version: '25'` and `distribution: 'temurin'` in all `actions/setup-java` steps, matching the current Temurin + explicit version pattern.

**Rationale:** Consistency with existing workflows; Temurin is widely available on GitHub-hosted runners.

**Alternatives:** Oracle JDK distribution (licensing/availability varies on runners); matrix build 21+25 during transition (adds CI cost; proposal treats 21 drop as breaking).

### 2. Centralize version in root POM properties

**Choice:** Update root `maven.compiler.source` and `maven.compiler.release` to `25`; align `pluginManagement` compiler `source`/`target`/`release`; update module overrides in `object/pom.xml` and `hdfview/pom.xml` to `25` only where they hardcode `21`.

**Rationale:** Matches current pattern; avoids drift between modules.

**Alternatives:** Maven toolchain plugin only (does not replace documenting minimum JDK for scripts); parent-only release with child inheritance (some children already duplicate config—still need a pass).

### 3. Launcher minimum version check: 25

**Choice:** Update `run-hdfview.sh` / `run-hdfview.bat` version parsing to require major version >= 25 (same pattern as existing Java 21 check).

**Rationale:** Fail fast for developers before opaque Maven errors.

### 4. SpotBugs: verify, do not block migration

**Choice:** After bump, run `mvn spotbugs:spotbugs` manually; enable lifecycle executions only if SpotBugs supports class file version 69. Otherwise leave disabled and update comments from "Java 21" to "Java 25".

**Rationale:** Historical blocker on Java 21; migration must not regress default builds.

**Alternatives:** Bump SpotBugs plugin/core preemptively without verification (risk of false positives or new failures).

### 5. Quality plugins: comment and config sweep, not major upgrades

**Choice:** Update PMD/Checkstyle XML descriptions and POM comments; run `maven-quality` workflow. Upgrade plugin versions only if JDK 25 causes failures.

**Rationale:** PMD 7 and Checkstyle 10.12+ already target modern Java; minimize churn.

### 6. jpackage: same profiles, JDK 25 runtime

**Choice:** No structural change to jpackage profiles; validate `jpackage-app-image` and one installer profile per platform after JDK bump.

**Rationale:** `jpackage` is tied to the building JDK; version bump is the primary change.

## Risks / Trade-offs

| Risk | Mitigation |
|------|------------|
| SWT or native HDF JARs fail on JDK 25 | Run full compile + smoke `run-hdfview` / UI smoke tests on Linux, macOS, Windows CI |
| GitHub runners lack JDK 25 early | Confirm `setup-java` supports 25; pin workflow only after a green trial run |
| SpotBugs still incompatible | Keep executions disabled; document in design/tasks |
| Contributor machines still on 21 | **BREAKING** called out in CONTRIBUTING; clear launcher error messages |
| Installer size/behavior changes with bundled JRE 25 | Compare app-image smoke tests in test-release workflows |
| `--enable-native-access` / JVM args for tests | Re-run object and hdfview unit tests; adjust surefire `argLine` only if JDK 25 requires it |

## Migration Plan

1. **Local:** Install JDK 25, update `JAVA_HOME`, run `./scripts/build-dev.sh` and targeted tests.
2. **Codebase:** Bump POMs and scripts in one PR; update all workflow `java-version` entries in the same change to avoid mixed CI.
3. **CI validation:** Trigger `ci-linux`, `ci-windows`, `ci-macos`, and `maven-quality`; optionally installer workflows on a maintainer branch.
4. **Docs:** Update `CLAUDE.md`, `CONTRIBUTING.md`, `docs/Testing-Guide.md`, and POM/XML comments referencing Java 21.
5. **Rollback:** Revert the single PR; no data migration. Released installers built on 21 remain valid until rebuilt.

## Open Questions

- Is JDK 25 available on all required GitHub runner images at merge time? (Verify on first CI run.)
- Does Eclipse SWT `3.126.0` for `gtk.linux.x86_64` (and macOS/Windows variants in CI) require a SWT version bump for JDK 25?
- Should `repository/` or published Maven artifacts document a new `maven.compiler.release` in consumer POMs?
