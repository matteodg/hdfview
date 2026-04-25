# Tasks: Migrate HDF5 from jarhdf5 to hdf5-java-ffm

**Input**: Design documents from `/specs/003-migrate-hdf5-ffm/`  
**Prerequisites**: [plan.md](./plan.md), [spec.md](./spec.md), [research.md](./research.md), [data-model.md](./data-model.md), [contracts/](./contracts/), [quickstart.md](./quickstart.md)

**Tests**: Constitution III requires keeping **`object/`** HDF5 automated tests green; tasks include **verification** runs (not TDD-first).

**Organization**: Phases follow **build before behavioral proof**: foundational Maven/JVM changes, then **User Story 2** (supply chain / CI), then **User Story 1** (parity / tests), then **User Story 3** (docs), then polish.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no ordering dependency on siblings)
- **[Story]**: `US1` = end-user HDF5 parity, `US2` = build/packaging single binding, `US3` = contributor docs
- Paths are repo-relative from workspace root

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Baseline inventory for reviewers and implementers.

- [x] T001 [P] Append an **Implementation inventory** subsection to `specs/003-migrate-hdf5-ffm/research.md` listing paths that reference `jarhdf5` or `--enable-native-access=jarhdf5` (from `rg` across the repo, excluding `.git`).

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Maven graph, OS profiles, and JVM flags so **`object`** compiles/tests against **`org.hdfgroup:hdf5-java-ffm`** only. **No user-story work before this checkpoint.**

**⚠️ CRITICAL**: Blocks all user stories.

- [x] T002 Verify published **`hdf5-java-ffm`** / **`hdf5-native`** **2.2.0** artifacts resolve for each CI/host OS (minimum **Linux x86_64** for default CI); record exact `dependency:get` commands and outcomes in `specs/003-migrate-hdf5-ffm/research.md` (**Verified packaging** subsection).
- [x] T003 Extend root `pom.xml` `<dependencyManagement>` with paired **`hdf5-native`** + **`hdf5-java-ffm`** entries (pinned **2.2.0**) for each **non-Windows** classifier confirmed in T002 (follow existing `windows-x86_64` pattern).
- [x] T004 Add OS-activated `<profile>` blocks in `object/pom.xml` that attach **`org.hdfgroup:hdf5-native`** + **`org.hdfgroup:hdf5-java-ffm`** for each platform from T003 (mirror `hdf5-org-hdfgroup-ffm-windows-amd64` / `hdf5-org-hdfgroup-ffm-windows-x86_64` style).
- [x] T005 Remove the compile `<dependency>` on **`jarhdf5:jarhdf5`** from `object/pom.xml` (default `<dependencies>` block) once T004 covers every platform that compiles `object`.
- [x] T006 [P] Replace `--enable-native-access=jarhdf5` with `--enable-native-access=ALL-UNNAMED` in `object/pom.xml` Surefire `<argLine>` configuration.
- [x] T007 [P] Replace `--enable-native-access=jarhdf5` with `--enable-native-access=ALL-UNNAMED` in `hdfview/pom.xml` Surefire (or shared test JVM) `<argLine>` configuration (~line 836 region).
- [x] T008 [P] Update `run-hdfview.sh` JVM args: use **`--enable-native-access=ALL-UNNAMED`** for HDF5 FFM per `specs/003-migrate-hdf5-ffm/research.md` §2; retain **`--enable-native-access=jarhdf`** for HDF4 if still required.
- [x] T009 [P] Update `run-hdfview.bat` JVM args: same FFM policy as `run-hdfview.sh` (`run-hdfview.bat`).

**Checkpoint**: `mvn -q -DskipTests compile -pl object` succeeds on Linux + Windows with **no** `jarhdf5:jarhdf5` on the dependency tree.

---

## Phase 3: User Story 2 — Build and packaging (Priority: P1)

**Goal** (from [spec.md](./spec.md)): CI and local pipelines no longer install or depend on **`jarhdf5`**; single **`hdf5-java-ffm`** line per contract [contracts/single-hdf5-java-binding.md](./contracts/single-hdf5-java-binding.md).

**Independent Test**: `mvn -q -DskipTests dependency:tree -pl object` meets contract rows **B1–B2**; workflows do not run `install-file` for `jarhdf5` on success paths.

### Implementation for User Story 2

- [x] T010 [US2] Remove or replace the **`jarhdf5`** `maven-install-plugin` / `install-file` execution in `repository/pom.xml` so a standard build does not publish **`jarhdf5:jarhdf5`** into `~/.m2`.
- [x] T011 [P] [US2] Update `.github/workflows/ci-linux.yml` to drop **`jarhdf5`** `install-file` steps when org.hdfgroup artifacts supply HDF5 Java (`ci-linux.yml`).
- [x] T012 [P] [US2] Update `.github/workflows/maven-quality.yml`: remove **`jarhdf5`** install block (~196–206) and set Surefire `argLine` to **`--enable-native-access=ALL-UNNAMED`** (~318) (`maven-quality.yml`).
- [x] T013 [P] [US2] Update `.github/workflows/maven-security.yml` **`jarhdf5`** install-file section (~257–266) (`maven-security.yml`).
- [x] T014 [P] [US2] Update `.github/workflows/ci-windows.yml` **`jarhdf5`** install-file section (~189–196) (`ci-windows.yml`).
- [x] T015 [P] [US2] Update `.github/workflows/ci-macos.yml` **`jarhdf5`** install-file section (~172–177) (`ci-macos.yml`).
- [x] T016 [P] [US2] Update `.github/workflows/build-linux.yml` **`jarhdf5`** install-file section (~183–188) (`build-linux.yml`).
- [x] T017 [P] [US2] Update `.github/workflows/build-windows.yml` **`jarhdf5`** install-file blocks (~193–198 and ~449–454) (`build-windows.yml`).
- [x] T018 [P] [US2] Update `.github/workflows/build-macos.yml` **`jarhdf5`** download/install (~154–161) (`build-macos.yml`).
- [x] T019 [US2] Rework `.github/workflows/publish-maven-packages.yml` HDF5 Java acquisition (~104–133) so it does not publish **`jarhdf5:jarhdf5`** as the supported HDF5 Java line (`publish-maven-packages.yml`).

**Checkpoint**: CI configs align with **B1–B3** in `specs/003-migrate-hdf5-ffm/contracts/single-hdf5-java-binding.md`.

---

## Phase 4: User Story 1 — HDF5 files behave as before (Priority: P1)

**Goal**: Functional parity for open/browse/edit HDF5 paths; automated **`object`** regression green (spec **SC-001**, **FR-003**).

**Independent Test**: `mvn -q test -pl object` passes; manual smoke per [quickstart.md](./quickstart.md) §4.

### Verification & fixes for User Story 1

- [x] T020 [US1] Run `mvn -q -DskipTests dependency:tree -pl object` and confirm **B1** (**no** `jarhdf5:jarhdf5`, **`hdf5-java-ffm:2.2.0`** present) in `specs/003-migrate-hdf5-ffm/contracts/single-hdf5-java-binding.md`.
- [ ] T021 [US1] Run `mvn -q test -pl object` with HDF5 native paths configured (`build.properties` / CI `platform.hdf.lib`); triage failures against spec edge cases (`specs/003-migrate-hdf5-ffm/spec.md` §Edge Cases).
- [x] T022 [P] [US1] Update user-visible HDF5 limitation strings in `object/src/main/java/hdf/object/h5/H5ScalarDS.java` that name **`jarhdf5 2.0.0`** so they describe the current binding accurately (`H5ScalarDS.java`).
- [x] T023 [P] [US1] Align test expectations/messages in `object/src/test/java/object/TestComplexDatatype.java` with FFM-backed behavior (`TestComplexDatatype.java`).
- [ ] T024 [US1] Manual smoke: launch via `run-hdfview.sh` / `run-hdfview.bat` and open/browse at least one HDF5 sample per [quickstart.md](./quickstart.md) §4 (`run-hdfview.sh`, `run-hdfview.bat`).

**Checkpoint**: **SC-001** / **SC-004** evidence captured (test log + manual note in PR).

---

## Phase 5: User Story 3 — Contributors follow one HDF5 Java path (Priority: P2)

**Goal**: Contributor docs reference **FFM / org.hdfgroup** only; no **`jarhdf5`** as active binding (spec **FR-005**).

**Independent Test**: Doc search shows **zero** active `jarhdf5` setup steps except explicitly archived notes.

### Implementation for User Story 3

- [x] T025 [P] [US3] Update HDF5 Java / JVM flag guidance in `CLAUDE.md` (HDF5 **2.2.0** FFM, **`--enable-native-access=ALL-UNNAMED`**, remove **`jarhdf5`**-centric examples) (`CLAUDE.md`).
- [x] T026 [P] [US3] Update contributor-facing HDF5 sections in `README.md` to describe **`org.hdfgroup:hdf5-java-ffm`** workflow, not **`jarhdf5`** Windows profile (`README.md`).
- [x] T027 [US3] Reconcile **`002`** contributor quickstart with **`003`**: update `specs/002-hdf5-maven-deps/quickstart.md` (or add a supersession note pointing to `specs/003-migrate-hdf5-ffm/quickstart.md`) so readers are not instructed to keep **`jarhdf5`** for standard HDF5 (`specs/002-hdf5-maven-deps/quickstart.md`).
- [x] T028 [P] [US3] If modular metadata is ever re-enabled, update `object/src/main/java/module-info.java.disabled` and `object/src/test/java/module-info.java.disabled` to remove `requires jarhdf5;` or replace with the FFM JAR’s module name once known (`module-info.java.disabled` files under `object/`).

**Checkpoint**: **FR-005** / doc independent test satisfied.

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Repo-wide hygiene and acceptance checklist.

- [ ] T029 Run every step in `specs/003-migrate-hdf5-ffm/quickstart.md` and fix any gaps discovered (`quickstart.md` as checklist driver).
- [x] T030 [P] Repo-wide sanity: from repository root, `rg "jarhdf5"` on product surfaces (`*.xml`, `*.yml`, `*.sh`, `*.bat`, `*.md`) — **zero** `groupId>jarhdf5` / `install-file` for **`jarhdf5`** except explicitly marked historical notes (document residual hits in PR if any).

---

## Dependencies & Execution Order

### Phase Dependencies

- **Phase 1**: Start immediately.
- **Phase 2**: Depends on Phase 1 inventory (optional) but **logically** starts after T001; **blocks** all user stories.
- **Phase 3 (US2)**: Depends on Phase 2 completion (especially T005–T009 + workflow coherence).
- **Phase 4 (US1)**: Depends on Phase 3 for CI/local parity; **T020–T021** are the gate for **SC-001**.
- **Phase 5 (US3)**: Can start after Phase 4 **T021** passes (docs should match working behavior); may proceed in parallel with **T024** if staffed.
- **Phase 6**: After US1–US3 scope intended for the release is done.

### User Story Dependencies

| Story | Depends on |
|-------|------------|
| **US2** (build) | Phase 2 |
| **US1** (parity) | US2 + Phase 2 (resolved graph + JVM flags) |
| **US3** (docs) | US1 test signal recommended before merging doc-only PR |

### Parallel Opportunities

- **Phase 1**: T001 standalone.
- **Phase 2**: After T002–T005 chain, **T006–T009** may run in parallel (**[P]**).
- **Phase 3**: **T011–T018** may run in parallel (**[P]**); **T010** and **T019** touch distinct workflow semantics — **T019** after strategy for packages is clear.
- **Phase 4**: **T022** and **T023** in parallel (**[P]**).
- **Phase 5**: **T025**, **T026**, **T028** in parallel (**[P]**).
- **Phase 6**: **T030** parallel with final editorial pass on **T029**.

### Parallel Example: User Story 2

```bash
# After T010 is merged or coordinated, apply workflow edits in parallel workers:
# T011 ci-linux.yml
# T012 maven-quality.yml
# T013 maven-security.yml
# T014 ci-windows.yml
# T015 ci-macos.yml
# T016 build-linux.yml
# T017 build-windows.yml
# T018 build-macos.yml
```

---

## Implementation Strategy

### MVP First (User Stories 2 + 1)

1. Complete **Phase 2** (Maven + JVM + launchers).  
2. Complete **Phase 3 (US2)** — CI / `repository/pom.xml` free of **`jarhdf5`** coordinates.  
3. Complete **Phase 4 (US1)** — `mvn test -pl object` + minimal manual smoke.  
4. **STOP and VALIDATE** against [contracts/single-hdf5-java-binding.md](./contracts/single-hdf5-java-binding.md).

### Incremental Delivery

1. Add **Phase 5 (US3)** docs after behavior is proven.  
2. **Phase 6** quickstart + `rg` gate before merge.

### Parallel Team Strategy

- Developer A: Phase 2 (`pom.xml`, `object/pom.xml`, launchers).  
- Developer B: Phase 3 workflows (T011–T018).  
- Developer C: Phase 4 test triage + `H5ScalarDS` / `TestComplexDatatype` messages.  
- Tech writer: Phase 5 once **T021** is green.

---

## Notes

- **HDF4** (`jarhdf`) stays on its own JNI/native-access path until a separate feature migrates it.  
- If **`dependency:get`** fails for a planned Linux/mac classifier, **do not** land T005 until T002/T003/T004 are adjusted to a **published** classifier or documented mirror (**constitution I**).  
- Commit after each phase checkpoint; prefer small PRs: Phase 2+3 vs Phase 4 vs Phase 5.

---

## Task summary

| Metric | Value |
|--------|-------|
| **Total tasks** | 30 |
| **Phase 1** | 1 |
| **Phase 2** | 8 |
| **US2** | 10 (T010–T019) |
| **US1** | 5 (T020–T024) |
| **US3** | 4 (T025–T028) |
| **Polish** | 2 (T029–T030) |
| **Format** | All lines use `- [ ] Tnnn` with optional `[P]` and required `[USn]` on story phases; file paths in descriptions |
