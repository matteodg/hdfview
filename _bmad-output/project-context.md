---
project_name: 'hdfview'
user_name: 'Matteo'
date: '2026-06-04'
sections_completed:
  - technology_stack
  - language_rules
  - framework_rules
  - testing_rules
  - quality_rules
  - workflow_rules
  - anti_patterns
status: complete
rule_count: 48
optimized_for_llm: true
---

# Project Context for AI Agents

_This file contains critical rules and patterns that AI agents must follow when implementing code in this project. Focus on unobvious details that agents might otherwise miss._

---

## Technology Stack & Versions

| Component | Version / note |
|-----------|----------------|
| Java | 21 (`maven.compiler.release=21`) |
| Maven BOM | `hdfview-bom` **3.4.1** |
| Modules (build order) | `repository` → `object` → `hdfview` |
| HDF4 native | **4.3.1** (via `build.properties`) |
| HDF5 native | **2.1.1** (local `build.properties` paths; plugins dir recommended) |
| SLF4J | **2.0.17** |
| JUnit 5 | **5.10.0** (+ JUnit 4 vintage during migration) |
| SWT | **3.126.0** (platform artifact, default `gtk.linux.x86_64`) |
| NatTable | **2.5.0** |
| JFace | **3.34.0** |
| Main class | `hdf.view.HDFView` |
| Module system | **Disabled** — classpath build, not JPMS |
| Config file | `build.properties` (not committed with secrets; required locally) |

Also read `CLAUDE.md` and `docs/` for extended build/run guidance.

---

## Critical Implementation Rules

### Language-Specific Rules (Java)

- **Packages:** Production code uses `hdf.view.*` (GUI) and `hdf.object.*` (data model) — not `org.hdfgroup.*` in source trees despite Maven `groupId`.
- **Module boundary:** Put HDF format/data logic in `object/`; SWT/UI only in `hdfview/`. `hdfview` depends on `object`; never the reverse.
- **Build `repository` first** — local Maven repo population; builds fail if order is wrong.
- **Resources:** Icons/images may live under `src/main/java` as well as `src/main/resources` (Maven resources plugin copies both).
- **Logging:** Use SLF4J APIs; do not introduce alternate logging frameworks without discussion.
- **Error handling:** Prefer existing HDFView patterns (dialogs, `Tools.showError`, object-layer exceptions) over new global handlers.
- **Native calls:** Treat JNI/HDF5 calls as crash-prone — validate inputs, avoid speculative native experiments in UI thread.

### Framework-Specific Rules (SWT / HDF object layer)

- **UI thread:** All SWT widget create/update on the display thread; use `Display.syncExec` / `asyncExec` when crossing threads.
- **Factories:** Views use existing factory/producer patterns (`DataViewFactoryProducer`, `ImageViewFactory`, etc.) — extend, do not duplicate parallel hierarchies.
- **Format packages:** HDF4 → `hdf.object.h4`, HDF5 → `hdf.object.h5`, NetCDF/FITS in their subpackages — keep format-specific code there.
- **NatTable:** Reuse existing table/view infrastructure in `hdf.view.TableView` rather than raw SWT tables for dataset editing.
- **SWT platform:** Default POM targets Linux GTK; changing SWT platform requires POM/profile changes — do not assume Windows/macOS widgets match Linux without testing.
- **Float8 / Float16 / BFLOAT16:** Supported with application workarounds; native HDF5 gaps exist — follow existing guards; do not remove crash protection.

### Testing Rules

- **Framework:** JUnit 5 primary; JUnit 4 tests still run via vintage engine — migrate new tests to Jupiter.
- **Tags:** Use `@Tag("unit")`, `@Tag("fast")`, `@Tag("integration")`, `@Tag("ui")` consistently; run subsets with `-Dgroups=...`.
- **Object tests:** Under `object/src/test/java/object/` (and related); tag `unit` + `fast` where appropriate.
- **UI tests:** Under `hdfview/src/test/java/uitest/`; extend `AbstractWindowTest`; require real display (fail/skip in headless CI).
- **JVM args for tests** (Surefire — required for native/module access):
  ```
  --add-opens java.base/java.lang=ALL-UNNAMED
  --add-opens java.base/java.time=ALL-UNNAMED
  --add-opens java.base/java.time.format=ALL-UNNAMED
  --add-opens java.base/java.util=ALL-UNNAMED
  --enable-native-access=jarhdf5
  ```
- **Coverage:** JaCoCo targets ~60% line / 50% branch project-wide; object module stricter (~70% on key packages). Avoid lowering thresholds without cause.
- **HDF data changes:** Test multiple datatypes (compound, vlen, refs, Float16) when touching read/write paths.
- **Run single test:** `mvn test -pl hdfview -Dtest=TestClassName` or `-pl object` as appropriate.

### Code Quality & Style Rules

- **Checkstyle:** Google-based rules in `checkstyle-rules.xml` (10.12+, Java 21); severity warning; max file length 2500 lines for Java.
- **PMD:** `pmd-rules.xml` — run via Maven quality profile / `scripts/validate-quality.sh` before claiming work is done.
- **Scope of change:** Minimal diff focused on the task; match surrounding naming, imports, and comment density.
- **Comments:** Only for non-obvious HDF/native/UI behavior — avoid narrating obvious code.
- **File headers:** Preserve existing HDF Group copyright/license headers on new files.
- **No Ant:** Maven-only build — do not reintroduce Ant scripts.

### Development Workflow Rules

- **Local config:** Copy/configure `build.properties` with `hdf.lib.dir`, `hdf5.lib.dir`, `hdf5.plugin.dir`, `platform.hdf.lib` (Windows: semicolon-separated `bin` paths in `platform.hdf.lib` when empty runtime path fails).
- **Quick build:** `./scripts/build-dev.sh` (tests/quality optional flags).
- **Deep clean:** `./scripts/clean-all.sh` when classloading or stale artifact issues appear.
- **Run app:** `./run-hdfview.sh` or `run-hdfview.bat` (validates env, can build if needed).
- **Debug logging:** `./scripts/set-debug-logging.sh` for targeted SLF4J configs (e.g. float16).
- **Commits:** Only when user explicitly requests; never skip hooks or force-push `main`.
- **Installers:** jpackage profiles (`jpackage-app-image`, platform installer profiles) — signing only on canonical CI.

### Critical Don't-Miss Rules

- **Do not** call HDF native APIs from wrong thread or without existing lifecycle (open/close file) patterns.
- **Do not** add dependencies that pull a second SWT version — exclusions in POM are intentional.
- **Do not** enable Java modules (`module-info.java`) without a dedicated migration effort.
- **HDF5 native vs Java binding:** `build.properties` points at installed natives (e.g. **2.1.1**); root `pom.xml` `hdf5.version` and `repository/lib/jarhdf5-${hdf5.version}.jar` must match those natives or JNI will misbehave.
- **Do not** assume native HDF5 behavior matches application Float16/BFLOAT16 workarounds — verify against your installed version; see `CLAUDE.md` known issues.
- **Do not** add broad try/catch that swallows native failures — prefer fail-safe UI messaging.
- **Do not** commit `build.properties` secrets or machine-specific absolute paths meant for local use only.
- **Do not** create commits or PRs unless the user asks.
- **UI tests in CI:** Often disabled/no display — do not “fix” by removing tags; use unit/object tests for CI-safe coverage.
- **Windows exec plugins:** Some shell-based Maven exec steps skip on Windows (`maven.exec.skip`) — use documented launchers instead.

---

## Usage Guidelines

**For AI Agents:**

- Read this file (and `CLAUDE.md`) before implementing any code.
- Follow ALL rules above; when in doubt, prefer the more restrictive option.
- Update this file when new non-obvious patterns emerge.

**For Humans:**

- Keep lean — agent-focused unobvious rules only.
- Update when stack or native library versions change.
- Review periodically; remove rules that become universal knowledge.

Last Updated: 2026-06-04
