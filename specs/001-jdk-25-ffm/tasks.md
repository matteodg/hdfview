---
description: "Task list for JDK 25 baseline (FFM readiness)"
---

# Tasks: JDK 25 baseline for FFM-ready builds

**Input**: Design documents from `/specs/001-jdk-25-ffm/`  
**Prerequisites**: [plan.md](./plan.md), [spec.md](./spec.md), [research.md](./research.md),
[data-model.md](./data-model.md), [contracts/](./contracts/), [quickstart.md](./quickstart.md)

**Tests**: Not explicitly requested in the feature spec; validation is via **Maven compile/test**
and CI parity (see T011, T030).

**Organization**: Phases follow user-story priorities after shared setup and a constitution gate.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no ordering dependency within the same phase)
- **[Story]**: `US1`, `US2`, `US3` map to priorities in `spec.md`

## Path Conventions

- **Root POM**: `pom.xml`
- **Modules**: `object/pom.xml`, `hdfview/pom.xml`
- **CI**: `.github/workflows/*.yml`
- **Docs & launchers**: `CLAUDE.md`, `CONTRIBUTING.md`, `docs/`, `run-hdfview.sh`, `run-hdfview.bat`
- **Governance**: `.specify/memory/constitution.md`

---

## Phase 1: Setup (inventory)

**Purpose**: Confirm full blast radius before edits.

- [x] T001 [P] List every `java-version: '21'` hit under `.github/workflows/` (record file paths; no
  edits yet).
- [x] T002 [P] List every contributor-facing `Java 21` / `JDK 21` / `Java 21+` string in
  `CLAUDE.md`, `CONTRIBUTING.md`, `docs/`, `run-hdfview.sh`, and `run-hdfview.bat` (record paths; no
  edits yet).

---

## Phase 2: Foundational (constitution gate)

**Purpose**: Satisfy plan **Constitution Check** — governance MUST match the new JDK **before** or
**with** the same merge series as toolchain edits.

**⚠️ Gate**: Before merging POM/CI that require JDK **25**, confirm `.specify/memory/constitution.md`
requires **Java 25** (**≥1.1.0**) so governance and toolchain stay aligned.

- [x] T003 Verify Java **25** baseline language, Sync Impact Report, and semver (**≥1.1.0**) in
  `.specify/memory/constitution.md` (Principle I + Engineering Standards; align with governance
  rules in that file). If already amended on this branch, re-check diff and **Last Amended** date only.

---

## Phase 3: User Story 1 — Maintainer adopts JDK 25 for the build (Priority: P1)

**Goal**: Maven + Enforcer + module POMs compile with `release` **25**; builds fail fast below 25.

**Independent Test**: On JDK **25**, `mvn clean compile -DskipTests` from repo root succeeds; on
JDK **24**, Enforcer emits an explicit **Java 25 required** failure (per **FR-002**).

### Implementation for User Story 1

- [x] T004 [US1] Set `maven.compiler.source` and `maven.compiler.release` to **25** in root `pom.xml`
  `<properties>`.
- [x] T005 [US1] Set `maven-compiler-plugin` `<source>`, `<target>`, and `<release>` to **25** in root
  `pom.xml` `<pluginManagement><plugins>` entry for `maven-compiler-plugin`.
- [x] T006 [US1] Set `maven-javadoc-plugin` `<source>` and `<target>` to **25** in root `pom.xml`.
- [x] T007 [US1] Set PMD `targetJdk` to **25** in root `pom.xml` `maven-pmd-plugin` `<configuration>`.
- [x] T008 [US1] Set compiler `<source>`, `<target>`, and `<release>` to **25** in `object/pom.xml` for
  the `maven-compiler-plugin` configuration.
- [x] T009 [US1] Set compiler `<source>`, `<target>`, and `<release>` to **25** in `hdfview/pom.xml` for
  the `maven-compiler-plugin` configuration.
- [x] T010 [US1] Add `requireJavaVersion` with version **`[25,)`** (or **`25`** per Enforcer docs) to
  `maven-enforcer-plugin` in root `pom.xml`, with a custom `<message>` (e.g. **Java 25 or newer is
  required to build HDFView.**); do **not** use **`1.25`** as the version string. Preserve existing
  HDF-related enforcer executions.
- [ ] T011 [US1] From repo root on JDK **25**, run `mvn clean compile -DskipTests` and fix any
  compilation issues until green.

**Checkpoint**: Maintainer story satisfied for local Maven compile path.

---

## Phase 4: User Story 2 — CI reflects the new baseline (Priority: P2)

**Goal**: All primary GitHub Actions jobs that compile or test Java use **JDK 25** (Temurin per
`research.md` unless a workflow already forces a different distribution for good reason).

**Independent Test**: Grep shows no `java-version: '21'` under `.github/workflows/`; CI logs for a
push show `setup-java` selecting **25**.

### Implementation for User Story 2

- [x] T012 [P] [US2] Update `actions/setup-java` step names and `java-version` to **25** in
  `.github/workflows/ci-linux.yml`.
- [x] T013 [P] [US2] Same for `.github/workflows/ci-macos.yml`.
- [x] T014 [P] [US2] Same for `.github/workflows/ci-windows.yml`.
- [x] T015 [P] [US2] Same for `.github/workflows/maven-quality.yml`.
- [x] T016 [P] [US2] Same for **all** `setup-java` steps in `.github/workflows/maven-security.yml`.
- [x] T017 [P] [US2] Same for `.github/workflows/build-linux.yml`.
- [x] T018 [P] [US2] Same for `.github/workflows/build-macos.yml`.
- [x] T019 [P] [US2] Same for **both** JDK setup jobs in `.github/workflows/build-windows.yml`.
- [x] T020 [P] [US2] Same for `.github/workflows/publish-maven-packages.yml`.

**Checkpoint**: CI configuration matches **FR-003**.

---

## Phase 5: User Story 3 — Contributors install and verify JDK 25 (Priority: P3)

**Goal**: Documentation and launch scripts agree on **Java 25** minimum; quickstart remains accurate.

**Independent Test**: A contributor following `CLAUDE.md` / `CONTRIBUTING.md` / `quickstart.md` and
`run-hdfview.sh --validate` (or Windows equivalent) sees consistent **25** messaging.

### Implementation for User Story 3

- [x] T021 [P] [US3] Update Java version and jpackage wording in `CLAUDE.md` to **Java 25** (build,
  plugins, installers sections).
- [x] T022 [P] [US3] Update Java prerequisite line in `CONTRIBUTING.md` to **Java 25** minimum.
- [x] T023 [P] [US3] Update prerequisites in `docs/Testing-Guide.md` to **Java 25**.
- [x] T024 [P] [US3] Update JDK bullet in `docs/guides/Cross-Platform-Build-Quick-Reference.md` to
  **JDK 25**.
- [x] T025 [US3] Update header comment, version check, and user-facing messages in `run-hdfview.sh` to
  require **Java 25+** (not 21).
- [x] T026 [US3] Update comments and messages in `run-hdfview.bat` to require **Java 25+** (not 21).

**Checkpoint**: Contributor-facing surfaces match **FR-004** and `contracts/developer-build-environment.md`.

---

## Phase 6: Polish & cross-cutting concerns

**Purpose**: Stragglers, packaging alignment, and broader validation.

- [x] T027 [P] Repo-wide search for `java-version: '21'`, `Java 21`, `JDK 21`, and `Java 21+`; fix any
  remaining **minimum** requirement strings (including `README.md`, `CLAUDE.md`, and misleading
  comments in `pom.xml`).
- [x] T028 Re-read `specs/001-jdk-25-ffm/quickstart.md` against final `pom.xml` Enforcer behavior and
  update if commands or failure messages changed.
- [x] T029 Audit `hdfview/pom.xml` and root `pom.xml` for **jpackage** / release installer profiles;
  align any hard-coded JDK **21** toolchains with **25** or document a scoped exception in
  `specs/001-jdk-25-ffm/plan.md`.
- [ ] T030 Run `mvn test` from repo root on JDK **25** (or CI-parity command); if a quality plugin
  cannot parse bytecode, file a targeted upgrade/suppression **with rationale** per project
  constitution (no silent wide skips).

---

## Dependencies & execution order

### Phase dependencies

- **Setup (Phase 1)** → no dependencies.
- **Foundational (Phase 2)** → depends on Phase 1 inventory (recommended) but may start once scope
  is understood; **blocks** US1–US3 merges if constitution lags.
- **User Story 1 (Phase 3)** → depends on Phase 2 constitution update **before** merge to `master`.
- **User Story 2 (Phase 4)** → may start after Phase 3 **T010** is drafted; should merge together with
  Phase 3 for CI greenness.
- **User Story 3 (Phase 5)** → can proceed in parallel with Phase 4 once Phase 3 POM edits are
  stable (docs must match final behavior).
- **Polish (Phase 6)** → after Phases 3–5 on the integration branch.

### User story dependencies

- **US1 (P1)**: Blocks **US2** if CI would compile with the old JDK (merge POM + CI in one PR or
  stacked PRs with no intermediate broken state).
- **US2 (P2)**: Depends on US1 POM/enforcer truth.
- **US3 (P3)**: Depends on US1 behavior for accurate docs; can proceed alongside US2.

### Parallel opportunities

- **T001** and **T002** in parallel.
- **T012–T020** in parallel (different workflow files).
- **T021–T024** in parallel (different documentation files).
- **T027** can run after merges; **T029** can be parallel with **T028** if different owners.

---

## Parallel example: User Story 2

```bash
# Assign different workflow files independently:
Task: "Update .github/workflows/ci-linux.yml …"
Task: "Update .github/workflows/ci-macos.yml …"
Task: "Update .github/workflows/ci-windows.yml …"
# … continue through publish-maven-packages.yml
```

---

## Implementation strategy

### MVP first (User Story 1 + constitution)

1. Complete Phase 1 inventory.
2. Complete Phase 2 constitution alignment (**T003**).
3. Complete Phase 3 (**T004–T011**) and stop to validate **FR-001** / **FR-002** locally.

### Incremental delivery

1. Land **T003–T011** (build truth on JDK 25).
2. Land **T012–T020** (CI uses JDK 25).
3. Land **T021–T026** (contributor experience).
4. Land **T027–T030** (polish + full test sweep).

### Suggested MVP scope

- **T003–T011** plus **T012–T020** in a single integration branch/PR series so `master` never has POM
  requiring **25** while CI still installs **21**.

---

## Notes

- **Total tasks**: 30  
- **Per story**: US1 → 8 tasks (T004–T011, all labeled `[US1]`); US2 → 9 tasks (T012–T020); US3 → 6
  tasks (T021–T026); Setup 2; Foundational 1; Polish 4 (T027–T030).
- **Format**: Every implementation line uses `- [ ]`, sequential `Txxx`, optional `[P]`, story
  labels **`[US1]`** on **T004–T011**, **`[US2]`** on **T012–T020**, **`[US3]`** on **T021–T026**
  (no story labels on setup, foundational, or polish phases), and an explicit file path.
- **Open**: **T011** and **T030** remain unchecked until a machine with **JDK 25**, valid
  `build.properties`, and HDF native libraries runs `mvn clean compile -DskipTests` and `mvn test`
  successfully (local failure observed was missing HDF/`object` antrun paths, not Java baseline).
