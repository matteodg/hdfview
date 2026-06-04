# Deferred Work

## Deferred from: code review of 1-1-set-jdk-25-as-maven-compiler-baseline (2026-06-04)

- CI workflows still pin `java-version: '21'` across 9 `.github/workflows/*.yml` files; JDK 21 cannot compile `--release 25`, so CI will fail after merge until bumped. → Story 1.5.
- PMD `<targetJdk>21</targetJdk>` in root `pom.xml:384` not aligned with compiler release 25. → Story 1.2.
- maven-javadoc-plugin `source/target` 25 (`pom.xml:296-299`) not exercised by `mvn package`; javadoc goal needs JDK 25 tool. → Story 1.2.
- Surefire/CI `--enable-native-access=jarhdf5` / `--add-opens` argLine (object/hdfview POMs, maven-quality.yml) vs JDK version tension. → Story 1.2.
- Launcher scripts enforce Java ≥21 (`run-hdfview.sh:98-101`, `run-hdfview.bat:80-91`), allowing 21–24 to pass then fail compile. → Story 1.3.
- Docs still list Java 21 (`docs/Testing-Guide.md:22`, `docs/guides/Cross-Platform-Build-Quick-Reference.md:6`). → Story 1.7.
- `_bmad-output/project-context.md:28,87` still documents Java 21 / `maven.compiler.release=21`. → Story 1.7.
- Quality ruleset metadata says "Java 21" (`pmd-rules.xml:7`, `checkstyle-rules.xml:9`). → Story 1.2.
- Stale comments in root `pom.xml` reference "Java 21" and an incorrect class-file major version (68 = Java 24, not 25) in the SpotBugs block. → Story 1.2.
