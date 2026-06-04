## ADDED Requirements

### Requirement: Minimum Java version is 25

The HDFView project SHALL require JDK 25 or later for compiling, testing, packaging, and running the application from source.

#### Scenario: Developer with JDK 25 builds successfully

- **WHEN** a developer runs `mvn clean compile` with JDK 25 as `JAVA_HOME`
- **THEN** all modules compile without compiler version errors

#### Scenario: Developer with JDK 21 is rejected at launch

- **WHEN** a developer runs `run-hdfview.sh` or `run-hdfview.bat` with only Java 21 on PATH
- **THEN** the launcher reports that Java 25+ is required and exits before starting the application

### Requirement: Maven targets Java 25 bytecode

The root and module Maven builds SHALL set `maven.compiler.source`, `maven.compiler.release`, and compiler plugin `release` to 25 consistently.

#### Scenario: Packaged JAR reports Java 25

- **WHEN** the project is built with `mvn package -DskipTests`
- **THEN** main application classes are compiled for Java 25 (class file major version 69)

#### Scenario: Version metadata reflects Java 25

- **WHEN** `versions.properties` is filtered during the build
- **THEN** `JAVA_VERSION` resolves to `25`

### Requirement: CI uses JDK 25

All GitHub Actions workflows that set up Java for build, test, quality, publish, or installer jobs SHALL use `java-version: '25'` with a supported distribution (Temurin unless platform constraints require otherwise).

#### Scenario: Linux CI job compiles on JDK 25

- **WHEN** the `ci-linux` workflow runs the compile step
- **THEN** the job uses JDK 25 from `actions/setup-java`

#### Scenario: Windows installer workflow uses JDK 25

- **WHEN** the Windows build or test-release workflow packages the application
- **THEN** the job uses JDK 25 for Maven and `jpackage` steps

### Requirement: Quality tools support JDK 25

PMD and Checkstyle SHALL run successfully against Java 25-compiled sources in the standard quality workflow. SpotBugs MAY be enabled only if the configured SpotBugs version supports Java 25 bytecode without failing the build.

#### Scenario: Maven quality workflow completes static analysis

- **WHEN** `maven-quality` runs PMD and Checkstyle on a clean tree built with JDK 25
- **THEN** the analysis phases complete without tool version or bytecode incompatibility failures

#### Scenario: SpotBugs does not break the default build

- **WHEN** SpotBugs remains disabled in default Maven lifecycle executions
- **THEN** a standard `mvn verify` does not fail due to SpotBugs Java 25 bytecode limitations

### Requirement: jpackage installers build with JDK 25

Platform installer profiles SHALL produce app-images and installers using the JDK 25 toolchain when invoked per project documentation.

#### Scenario: App-image profile succeeds locally

- **WHEN** a maintainer runs `mvn verify -Pjpackage-app-image -pl object,hdfview -DskipTests` on a supported platform with JDK 25
- **THEN** the jpackage app-image is created without JDK version errors
