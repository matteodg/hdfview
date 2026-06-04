# Deferred Work

## Deferred from: code review of 1-2-validate-tests-and-quality-plugins-on-jdk-25 (2026-06-04)

- Quoted `-Djava.library.path` in object Surefire argLine validated on Windows only; verify on Linux/macOS once CI moves to JDK 25. → Story 1.5.
- `maven-quality.yml` uses an unquoted CLI `-Djava.library.path=${HDF5LIB_PATH}/lib`; latent space-handling parallel to the Surefire fix. → Story 1.5.
- `platform.hdf.lib` is not validated by the maven-enforcer rules (only `hdf5.lib.dir` is); an empty value silently breaks native test loading. Consider an enforcer rule. → follow-up.
- No tracked `build.properties` template (e.g. `build.properties.example`) for fresh clones since the real file is gitignored. → Story 1.7 / docs.

## Deferred from: dev of 1-2-validate-tests-and-quality-plugins-on-jdk-25 (2026-06-04)

- **JaCoCo object-module coverage not collected:** the `object` Surefire `<argLine>` is set literally and overrides JaCoCo's agent `argLine`, so no `.exec`/report is produced. Fix needs `@{argLine}` wiring + JaCoCo bump ≥0.8.13 (0.8.12 predates JDK 25 class-file major 69), and may surface a 60%/50% threshold failure — warrants a dedicated story. Do NOT lower thresholds.
- **`hdfview/pom.xml` Surefire argLine** has the same unquoted `-Djava.library.path=${platform.hdf.lib}` that crashes the forked VM when the path contains spaces. Apply the same quoting fix during Story 1.6 (UI tests).

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
