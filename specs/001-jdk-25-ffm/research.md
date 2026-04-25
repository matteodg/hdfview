# Phase 0 — Research: JDK 25 baseline

## 1. JDK distribution in CI and locally

**Decision**: Use **Eclipse Temurin 25** (via `actions/setup-java@v4`, `distribution: temurin`) as
the default CI JDK, matching common HDFView contributor setups.

**Rationale**: Temurin is widely mirrored on GitHub Actions, pin-able, and aligns with prior
implicit OpenJDK-style usage.

**Alternatives considered**:

- **Oracle OpenJDK 25** — acceptable if Actions matrix supports it; Temurin chosen for consistency
  with existing community defaults.
- **Microsoft build of OpenJDK (Windows-only)** — rejected for primary matrix (splits behavior by
  OS).

**Follow-up**: On first CI run after bump, confirm `setup-java` accepts `java-version: '25'`. If a
workflow uses a self-hosted runner without JDK 25, document the required image update.

---

## 2. Maven enforcement strategy

**Decision**: Combine **compiler `release` 25** (root properties + `maven-compiler-plugin`
pluginManagement) with **`requireJavaVersion` ≥ 25** in `maven-enforcer-plugin` (new or merged
execution in root `pom.xml`).

**Rationale**: Compiler flags alone still allow older JDKs to fail with confusing errors; Enforcer
fails fast with a clear message (matches **FR-002**).

**Alternatives considered**:

- **Maven Toolchains only** — stronger for multi-JDK machines but adds `toolchains.xml` burden for
  every contributor; defer unless team requests it.
- **Wrapper script only** — insufficient because CI and IDEs invoke Maven directly.

---

## 3. Static analysis plugins vs Java 25 bytecode

**Decision**: Raise **PMD** `targetJdk` to **25** alongside `maven.compiler.release`. Re-validate
**Checkstyle** and **JaCoCo** on JDK 25 in CI. Treat **SpotBugs** as already “best effort” (plugin
comments note bytecode limits); re-run once on JDK 25 and record outcome—**do not** silently delete
quality phases.

**Rationale**: Constitution requires keeping quality gates; bytecode level changes can surface plugin
gaps.

**Alternatives considered**:

- **Skip static analysis on JDK 25** — rejected unless a plugin hard-fails with no upgrade path and
  an explicit exception is recorded per constitution.

---

## 4. Launcher scripts (`run-hdfview.*`)

**Decision**: Change minimum check from **Java 21+** to **Java 25+** (parse major version robustly
for `java -version` output variants).

**Rationale**: Contributors often bypass Maven enforcer when launching JARs directly; scripts are
part of the documented developer path.

**Alternatives considered**:

- **Keep scripts at 21** — violates spec acceptance scenario for older JDKs and contradicts FR-001.

---

## 5. FFM scope

**Decision**: **No FFM-based native calls** in this feature; JDK 25 is purely baseline + readiness.

**Rationale**: Spec explicitly scopes FFM to follow-on work; reduces risk while still unlocking API
availability.

**Alternatives considered**:

- **Pilot FFM wrapper** — out of scope; would mix native safety review with a mechanical JDK bump.

---

## 6. Constitution amendment

**Decision**: Patch `.specify/memory/constitution.md` **Principle I** and **Engineering Standards**
lines that state Java 21 → **Java 25**, bump constitution **PATCH** (wording alignment) or **MINOR**
if principles materially change—recommend **MINOR** because the runtime requirement changes.

**Rationale**: Plan constitution gate requires documentation alignment.
