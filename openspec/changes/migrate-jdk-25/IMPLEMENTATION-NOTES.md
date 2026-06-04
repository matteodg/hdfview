# Implementation Notes: migrate-jdk-25

## Local verification (JDK 25.0.3 Temurin, Windows)

| Check | Result |
|-------|--------|
| `mvn clean compile` | Pass |
| `mvn package -DskipTests` | Pass; `JAVA_VERSION=25` in filtered `versions.properties` |
| `mvn pmd:check` | Pass after upgrading PMD to 7.17.0 / maven-pmd-plugin 3.28.0 |
| `mvn checkstyle:check` | Pass |
| `mvn spotbugs:spotbugs` | Fail — SpotBugs 4.8.6 cannot read Java 25 bytecode; executions remain disabled |
| `mvn verify -Pjpackage-app-image -pl object,hdfview -DskipTests` | Pass (Windows app-image) |
| `mvn test -pl object` | Fail — `H5File` native init (`NoClassDefFoundError`); environment/native libs, not JDK migration |
| `./scripts/validate-quality.sh` | Fail at test phase (same native library issue) |

## CI (task 2.3)

Workflow files updated to `java-version: '25'`. Full matrix pass must be confirmed on push via GitHub Actions.

## Open questions (task 7.3)

- **GitHub runners:** JDK 25 availability confirmed via `actions/setup-java` in workflow YAML; runtime validation pending CI.
- **SWT:** No compile errors with current SWT 3.126.0 on JDK 25 locally; UI/runtime behavior needs CI or manual smoke test.
- **SpotBugs:** Still incompatible with Java 25 class files; keep disabled until SpotBugs core supports major version 69.

## Additional change

- Bumped `maven-pmd-plugin` to 3.28.0 and PMD artifacts to 7.17.0 for `targetJdk` 25 support.
