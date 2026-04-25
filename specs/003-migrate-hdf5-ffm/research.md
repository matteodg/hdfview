# Phase 0 — Research: jarhdf5 → hdf5-java-ffm

## 1. Java API surface after migration

**Decision**: Treat **`hdf.hdf5lib`** (`H5`, `HDF5Constants`, `HDFNativeData`, `exceptions.HDF5Exception`, etc.) as **unchanged** for migration planning.

**Rationale**: HDF Group documents that **JNI (`hdf5-java-jni`) and FFM (`hdf5-java-ffm`) use identical package and class names and method signatures**; migration is primarily artifact and JVM configuration. See [Using HDF5 Maven Artifacts — Migrating Between JNI and FFM](https://support.hdfgroup.org/documentation/hdf5/latest/_c_b__maven_artifacts.html).

**Alternatives considered**:

- **Assume full API rewrite** — rejected without evidence; would explode scope vs spec.
- **Spike-read `hdf5-java-ffm` JAR** — optional validation during implementation; not blocking for plan.

## 2. JVM native access for FFM

**Decision**: Standardize on **`--enable-native-access=ALL-UNNAMED`** for processes that load **`hdf5-java-ffm`**, per HDF Group FFM section of the same document.

**Rationale**: Replaces **`--enable-native-access=jarhdf5`**, which targeted the legacy automatic module name for the **`jarhdf5`** JAR; FFM guidance explicitly recommends **`ALL-UNNAMED`** for unrestricted callers.

**Alternatives considered**:

- **Named module only** — only if a future modular layout exposes a stable module name for the FFM JAR; current build is non-modular (`forceLegacyJavacApi`).

## 3. HDF4 (`jarhdf`) vs HDF5 flags

**Decision**: Keep **HDF4** on existing **`jarhdf`** native-access wiring where present; **add** FFM policy for HDF5 without assuming one flag replaces both.

**Rationale**: Launch scripts use **`--enable-native-access=ALL-UNNAMED`** for HDF5 FFM and retain
**`--enable-native-access=jarhdf`** for HDF4 JNI where applicable.

**Alternatives considered**:

- **`ALL-UNNAMED` only** — acceptable if it subsumes both HDF4 and HDF5 native access needs on all platforms; verify on Linux CI before deleting `jarhdf`-specific flags.

## 4. Linux / macOS classifiers and CI

**Decision**: Before removing **`jarhdf5`** from default **`object`** dependencies, **extend** root `dependencyManagement` and **`object/pom.xml`** OS-activated profiles so that **each CI runner OS/arch** that compiles `object` resolves **`hdf5-java-ffm:2.2.0`** with the **correct correspondent classifier** (and matching **`hdf5-native`**), mirroring the **002** “no foreign classifier” rule.

**Rationale**: Today only **Windows** profiles attach `hdf5-java-ffm`; **Linux CI** still relies on **`jarhdf5`** installed from `repository/lib`. Removing `jarhdf5` without Linux profiles would break CI.

**Alternatives considered**:

- **Windows-only migration** — violates spec (“all usages”) and leaves Linux on legacy line.

## 5. Local `repository/lib` and `install:install-file`

**Decision**: **Remove or bypass** workflow and `repository/pom.xml` steps that install **`jarhdf5`** into `~/.m2` once **`hdf5-java-ffm`** resolves from the configured repository (local cache, mirror, or Central).

**Rationale**: Spec SC-002 / FR-001 require **zero** remaining `jarhdf5` coordinates in scope; optional retention of a **physical** `jarhdf5-*.jar` on disk without Maven coordinates is confusing and should be avoided unless another tool requires it.

**Alternatives considered**:

- **Keep install-file for backwards compat** — rejected; contradicts “single binding” supply chain story.

## 6. User-visible strings and tests mentioning “jarhdf5”

**Decision**: Update **error messages and assertions** that name **`jarhdf5 2.0.0`** to refer to **HDF5 Java binding** or **FFM** where still accurate; re-validate **VLEN / complex** limitations against **hdf5-java-ffm** behavior (may differ from 2.0.0 JNI).

**Rationale**: Avoid misleading diagnostics after the legacy JAR is gone.

**Alternatives considered**:

- **Leave strings unchanged** — risks false diagnostics for support.

## 7. Relationship to specs/002-hdf5-maven-deps

**Decision**: **003** supersedes **002** research note “do not remove jarhdf5 in same PR”; **003** explicitly requires removal. **002** contracts for resolution visibility remain valid patterns; add **003** contract for **single binding** verification.

**Alternatives considered**:

- **Amend 002 artifacts in place** — unnecessary; 003 is the governing spec for removal.

---

## 8. Implementation inventory (speckit-implement, 2026-04-26)

Git-tracked paths that referenced **`jarhdf5`** or **`--enable-native-access=jarhdf5`** before migration
(product code, workflows, docs, and feature specs):

- `.github/workflows/build-linux.yml`, `build-macos.yml`, `build-windows.yml`
- `.github/workflows/ci-linux.yml`, `ci-macos.yml`, `ci-windows.yml`
- `.github/workflows/maven-quality.yml`, `maven-security.yml`, `publish-maven-packages.yml`
- `CLAUDE.md`, `README.md`
- `hdfview/pom.xml`, `object/pom.xml`, `repository/pom.xml`
- `object/src/main/java/hdf/object/h5/H5ScalarDS.java`
- `object/src/main/java/module-info.java.disabled`, `object/src/test/java/module-info.java.disabled`
- `object/src/test/java/object/TestComplexDatatype.java`
- `run-hdfview.bat`, `run-hdfview.sh`
- `specs/001-jdk-25-ffm/plan.md`, `specs/002-hdf5-maven-deps/*`, `specs/003-migrate-hdf5-ffm/*`

## 9. Verified packaging — `dependency:get` (2026-04-26)

| Artifact | Classifier | Resolver | Result |
|------------|------------|----------|--------|
| `org.hdfgroup:hdf5-java-ffm:2.2.0:jar` | `linux-x86_64` | Maven Central (`repo.maven.apache.org`) | **FAIL** — artifact not found (expected until Central publication). |
| `org.hdfgroup:hdf5-native:2.2.0:jar` | `linux-x86_64` | Maven Central | **FAIL** — same. |

**Mitigation merged in code**: Root `pom.xml` adds **`hdfgroup-github-packages`** repository
(`https://maven.pkg.github.com/HDFGroup/hdf5`); CI and developers authenticate per **§0** in
`quickstart.md`. Windows/Linux/mac profiles on `object` attach **`hdf5-java-ffm`** + **`hdf5-native`**
at **2.2.0** for the active platform classifier.
