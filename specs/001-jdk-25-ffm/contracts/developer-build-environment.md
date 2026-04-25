# Contract: Developer build environment (JDK 25)

**Audience**: Contributors and CI maintainers  
**Scope**: Local and CI builds of HDFView after the JDK 25 baseline feature.

## Preconditions

- Repository checkout at `001-jdk-25-ffm` or later merged state.
- `build.properties` provides valid HDF native paths (existing project rules).

## Required commands (MUST succeed)

| Step | Command | Expected signal |
|------|---------|-----------------|
| R1 | `java -version` | Major version **25** |
| R2 | `mvn -version` | Uses JDK **25** for “Java version:” line |
| R3 | `mvn -q -DskipTests validate` (or project’s documented first-build command) | Completes without
downgrading JDK in logs |

## MUST NOT

- M1: CI workflows that compile or test HDFView MUST NOT pin `java-version: '21'` as the **only**
Java for those jobs after this feature.
- M2: Documentation MUST NOT instruct contributors to install **Java 21** as the minimum for this
branch after the feature merges.

## Notes

- This contract is **developer-facing**; end-user installer JDK bundling follows existing packaging
docs and must be updated in lockstep if installers embed a JDK.
