# Contract: Developer dependency resolution (HDF5 2.2.0 org.hdfgroup line)

**Audience**: Contributors and CI maintainers (per **active** OS profile) and reviewers verifying
that **inactive** jobs do not pull foreign `hdf5-java-ffm` classifiers.  
**Scope**: Builds after the **002-hdf5-maven-deps** feature wires Maven coordinates per `spec.md`.
**Assumption**: HDF Group publishes **correspondent `hdf5-java-ffm` classifiers** for other platforms
at **2.2.0**; this contract’s R2 row uses **`windows-x86_64`** as the reference example.

## Preconditions

- Repository checkout on branch **`002-hdf5-maven-deps`** or later merged state.
- **Windows x86_64**: Resolver can see **`org.hdfgroup:hdf5-native:2.2.0`** and
  **`org.hdfgroup:hdf5-java-ffm:2.2.0:windows-x86_64`** (local `~/.m2`, corporate mirror, or Maven
  Central once published).
- **Linux/macOS (default CI)**: No requirement for **another** platform’s `hdf5-java-ffm` classifier
  to resolve when that platform’s profile is inactive.

## Required commands (MUST succeed)

| Step | Platform | Command | Expected signal |
|------|----------|---------|-------------------|
| R1 | Windows x86_64 | `mvn -q -DskipTests dependency:get -DgroupId=org.hdfgroup -DartifactId=hdf5-native -Dversion=2.2.0` (add `-Dpackaging=…` if not default `jar`) | Resolves **2.2.0** without substitution |
| R2 | Windows x86_64 | `mvn -q -DskipTests dependency:get -DgroupId=org.hdfgroup -DartifactId=hdf5-java-ffm -Dversion=2.2.0 -Dpackaging=jar -Dclassifier=windows-x86_64` | Classifier **`windows-x86_64`** resolves at **2.2.0** (substitute **correspondent** `-Dclassifier=` on other OS) |
| R3 | Any | `mvn -q -DskipTests validate` on Linux CI | Does **not** download **inactive** foreign `hdf5-java-ffm` classifiers |

## MUST NOT

- M1: Declared coordinates for this feature MUST NOT use open version ranges on **`2.2.0`**.
- M2: Documentation MUST NOT state that **all** OS/arch combinations consume the **same**
  **`hdf5-java-ffm`** classifier string (violates **FR-005**); each target uses its **correspondent**
  classifier.

## Notes

- Exact `dependency:get` incantations MAY be adjusted to match HDF Group packaging (`type` / packaging)
  once verified against the published POMs—update this contract in the same PR if commands change.
