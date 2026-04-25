# Contract: Single HDF5 Java binding (post–003 migration)

**Audience**: Maintainers, CI authors, release engineers.  
**Scope**: After **`jarhdf5`** removal; HDF5 access exclusively via **`org.hdfgroup:hdf5-java-ffm`** at **2.2.0** with the **active** platform classifier.  
**Related**: [specs/002-hdf5-maven-deps/contracts/developer-build-resolution.md](../002-hdf5-maven-deps/contracts/developer-build-resolution.md) (resolution visibility); this contract adds **binding uniqueness** and **FFM JVM** rules.

## Preconditions

- JDK **25** on the runner (FFM requirement).
- Resolver can see **`hdf5-java-ffm:2.2.0`** and **`hdf5-native:2.2.0`** for the **runner’s** classifier (local `~/.m2`, GitHub Packages, mirror, or Maven Central).

## MUST hold

| ID | Requirement |
|----|----------------|
| B1 | Effective dependency tree for **`object`** (compile + test) MUST list **`org.hdfgroup:hdf5-java-ffm:2.2.0`** with the **intended** classifier and MUST **NOT** list **`jarhdf5:jarhdf5`**. |
| B2 | Same for **`hdfview`** if it transitively or directly pulls HDF5 Java components. |
| B3 | JVM configs used to run tests (`argLine`, launch scripts) MUST include **`--enable-native-access=ALL-UNNAMED`** (or a documented equivalent) for FFM, and MUST **NOT** reference **`--enable-native-access=jarhdf5`**. |
| B4 | HDF4 **`jarhdf`** MAY remain documented separately; B3 still applies to combined launch command lines. |

## Suggested verification commands

| Step | When | Command | Expected |
|------|------|---------|----------|
| V1 | Local / CI | `mvn -q -DskipTests dependency:tree -pl object` | Contains **`hdf5-java-ffm:jar:…:2.2.0:compile`** (classifier matches OS); **no** `jarhdf5:jarhdf5` |
| V2 | After workflow edits | `rg "jarhdf5" --glob '!**/specs/**' --glob '!**/.git/**'` | Only allowed hits: HDF4 sidecars, historical docs explicitly marked, or **pending** migration TODOs (should trend to **zero** in product paths) |
| V3 | Tests | `mvn -q test -pl object` (with `build.properties` / env paths for natives) | No `IllegalCallerException` from FFM; HDF5 tests pass baseline |

## MUST NOT

- M1: Reintroduce **`jarhdf5`** as a Maven dependency “for compatibility” while **`hdf5-java-ffm`** is present.
- M2: Document that developers must download **`jarhdf5-*.jar`** into `repository/lib` for standard HDF5 builds (superseded by org.hdfgroup artifacts for this repo’s supported flow).
