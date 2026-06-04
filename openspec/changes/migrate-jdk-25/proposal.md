## Why

HDFView currently targets Java 21 across Maven, CI/CD, installers, and developer tooling. JDK 25 is the next long-term-support baseline and brings current JVM, `jpackage`, and toolchain improvements. Moving now keeps builds aligned with supported runtimes, reduces drift before downstream dependencies drop Java 21, and positions releases on a platform the project can standardize on for the next several years.

## What Changes

- Bump Maven compiler `source`, `target`, and `release` from 21 to 25 in the root and module POMs
- Update GitHub Actions workflows to install and use JDK 25 (Temurin) on Linux, macOS, and Windows
- Update developer-facing scripts and docs (`run-hdfview.sh`, `run-hdfview.bat`, `CLAUDE.md`, `CONTRIBUTING.md`, testing guides) to require Java 25+
- Refresh quality-tool configuration and comments (PMD, Checkstyle, SpotBugs) for JDK 25 compatibility; re-evaluate SpotBugs enablement if bytecode support is available
- Verify `jpackage` installer profiles still produce valid app-images and platform packages with JDK 25
- **BREAKING**: Local development and CI require JDK 25; Java 21 is no longer supported for building or running from source

## Capabilities

### New Capabilities

- `build-toolchain`: Defines the required Java version, Maven compiler settings, CI JDK setup, runtime validation in launch scripts, and compatibility expectations for quality tools and `jpackage`

### Modified Capabilities

<!-- No existing openspec/specs capabilities to modify -->

## Impact

- **Build**: Root `pom.xml`, `object/pom.xml`, `hdfview/pom.xml` (compiler plugin, filtered `JAVA_VERSION` in `versions.properties`)
- **CI/CD**: All workflows using `actions/setup-java` with `java-version: '21'` (ci-*, build-*, maven-*, publish-*, test-release-*)
- **Distribution**: `jpackage` profiles in `hdfview/pom.xml` and installer workflows; bundled JRE version in shipped artifacts
- **Developer experience**: Launcher scripts, `scripts/build-dev.sh`, `scripts/test-jpackage-local.sh`, documentation
- **Quality**: PMD/Checkstyle rule descriptors, SpotBugs plugin comments and optional activation
- **Dependencies**: SWT and native HDF bindings must run on JDK 25; any third-party bytecode limits need verification during implementation
