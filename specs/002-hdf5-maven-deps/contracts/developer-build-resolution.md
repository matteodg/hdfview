# Contract: Developer dependency resolution (HDF5 2.2.0 org.hdfgroup line)

**Audience**: Contributors and CI maintainers on **Windows x86_64** (and reviewers verifying
non-Windows CI stays green).  
**Scope**: Builds after the **002-hdf5-maven-deps** feature wires Maven coordinates per `spec.md`.

## Preconditions

- Repository checkout on branch **`002-hdf5-maven-deps`** or later merged state.
- **Windows x86_64**: Resolver can see **`org.hdfgroup:hdf5-native:2.2.0`** and
  **`org.hdfgroup:hdf5-java-ffm:2.2.0:windows-x86_64`** (local `~/.m2`, corporate mirror, or Maven
  Central once published).
- **Linux/macOS**: No requirement for these artifacts to resolve for default CI jobs (profile
  inactive).

## Required commands (MUST succeed)

| Step | Platform | Command | Expected signal |
|------|----------|---------|-------------------|
| R1 | Windows x86_64 | `mvn -q -DskipTests dependency:get -DgroupId=org.hdfgroup -DartifactId=hdf5-native -Dversion=2.2.0` (add `-Dpackaging=…` if not default `jar`) | Resolves **2.2.0** without substitution |
| R2 | Windows x86_64 | `mvn -q -DskipTests dependency:get -DgroupId=org.hdfgroup -DartifactId=hdf5-java-ffm -Dversion=2.2.0 -Dpackaging=jar -Dclassifier=windows-x86_64` | Classifier **`windows-x86_64`** resolves at **2.2.0** |
| R3 | Any | `mvn -q -DskipTests validate` on Linux CI | Does **not** download Windows-only classifiers |

## MUST NOT

- M1: Declared coordinates for this feature MUST NOT use open version ranges on **`2.2.0`**.
- M2: Documentation MUST NOT state that **all** OS/arch combinations consume **`hdf5-java-ffm`**
  **`windows-x86_64`** (violates **FR-005**).

## Notes

- Exact `dependency:get` incantations MAY be adjusted to match HDF Group packaging (`type` / packaging)
  once verified against the published POMs—update this contract in the same PR if commands change.
