## 1. Maven and build configuration

- [x] 1.1 Set `maven.compiler.source` and `maven.compiler.release` to `25` in root `pom.xml`
- [x] 1.2 Update `maven-compiler-plugin` `source`, `target`, and `release` to `25` in root `pluginManagement` and any duplicate configurations
- [x] 1.3 Update compiler `release` to `25` in `object/pom.xml` and `hdfview/pom.xml`
- [x] 1.4 Replace Java 21 references in root `pom.xml` comments (PMD, Checkstyle, SpotBugs) with Java 25
- [x] 1.5 Run `mvn clean compile` locally with JDK 25 and fix any compile-time deprecations or API removals

## 2. CI/CD workflows

- [x] 2.1 Update all `actions/setup-java` steps from `java-version: '21'` to `'25'` (ci-linux, ci-windows, ci-macos, build-*, maven-*, publish-maven-packages, test-release-* as applicable)
- [x] 2.2 Rename workflow step labels from "Set up JDK 21" to "Set up JDK 25"
- [ ] 2.3 Confirm a full CI matrix run passes on Linux, Windows, and macOS

## 3. Developer scripts and launchers

- [x] 3.1 Update `run-hdfview.sh` and `run-hdfview.bat` to require Java 25+ (messages and version checks)
- [x] 3.2 Update `scripts/build-dev.sh` and related scripts if they mention Java 21
- [x] 3.3 Update `scripts/test-jpackage-local.sh` comments or docs if they reference Java 21

## 4. Quality tooling

- [x] 4.1 Update `pmd-rules.xml` and `checkstyle-rules.xml` descriptions for Java 25
- [x] 4.2 Run `mvn pmd:check` and `mvn checkstyle:check` (or maven-quality workflow) on JDK 25; bump plugin versions only if needed
- [x] 4.3 Run `mvn spotbugs:spotbugs` manually; enable default executions only if Java 25 bytecode is supported; otherwise refresh disable comments

## 5. Installers and distribution

- [x] 5.1 Run `mvn verify -Pjpackage-app-image -pl object,hdfview -DskipTests` on a supported platform with JDK 25
- [ ] 5.2 Smoke-test one platform installer profile (deb, rpm, dmg, or msi) per design validation plan
- [x] 5.3 Verify filtered `JAVA_VERSION` in `hdfview/src/main/resources/versions.properties` reports 25 after build

## 6. Documentation

- [x] 6.1 Update `CLAUDE.md`, `CONTRIBUTING.md`, and `docs/Testing-Guide.md` to state Java 25 minimum
- [x] 6.2 Search repo for remaining "Java 21" / `java-version: '21'` strings and update or justify exceptions
- [x] 6.3 Note **BREAKING** JDK requirement in changelog or release notes if applicable for the target release

## 7. Verification

- [ ] 7.1 Run `mvn test` on JDK 25 for `object` and fast `hdfview` unit tests
- [ ] 7.2 Run `./scripts/validate-quality.sh` if used before merge
- [x] 7.3 Document any open questions from design (SWT compatibility, runner availability) with CI evidence in the PR
