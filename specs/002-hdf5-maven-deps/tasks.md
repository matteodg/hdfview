# Tasks: HDF5 native and Java FFM via published artifacts

**Input**: Design documents from `/specs/002-hdf5-maven-deps/`  
**Prerequisites**: [plan.md](./plan.md), [spec.md](./spec.md), [research.md](./research.md),
[data-model.md](./data-model.md), [contracts/](./contracts/), [quickstart.md](./quickstart.md)

**Tests**: Not required by `spec.md`; optional verification uses Maven commands in [quickstart.md](./quickstart.md)
and [contracts/developer-build-resolution.md](./contracts/developer-build-resolution.md).

**Organization**: Phases follow user stories (US1 → US2 → US3) after shared setup and foundation.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no unfinished dependencies)
- **[Story]**: `US1`, `US2`, `US3` — maps to priorities in `spec.md`
- Paths are relative to the repository root unless noted

---

## Phase 1: Setup (shared)

**Purpose**: Confirm how HDF Group publishes the artifacts before editing production POMs.

- [ ] T001 Inspect `~/.m2/repository/org/hdfgroup/hdf5-native/2.2.0/` and `~/.m2/repository/org/hdfgroup/hdf5-java-ffm/2.2.0/` (POMs, JAR layout, classifier layout) and append a short **Verified packaging** subsection to `specs/002-hdf5-maven-deps/research.md` (correct `dependency:get` `-Dpackaging` for `hdf5-native`).

---

## Phase 2: Foundational (blocking)

**Purpose**: Maven wiring and OS gating so **Linux/macOS CI** never requires unpublished Windows
artifacts; native precedence documented (**constitution II**).

**Checkpoint**: No user-story work until this phase completes.

- [ ] T002 Add properties `hdfgroup.hdf5.native.version` and `hdfgroup.hdf5.java.ffm.version` (both `2.2.0`) and `<dependencyManagement>` entries for `org.hdfgroup:hdf5-native` and `org.hdfgroup:hdf5-java-ffm` with classifier `windows-x86_64` in `pom.xml` (exact versions, no ranges).
- [ ] T003 Add a **Windows + amd64/x86_64**–activated `<profile>` in `object/pom.xml` with compile-scoped `<dependency>` elements referencing the managed coordinates (no version ranges on the declarations).
- [ ] T004 Run `mvn -q -DskipTests validate` at the repository root on **non-Windows**; iterate `object/pom.xml` profile `activation` until the graph does **not** pull `hdf5-java-ffm:windows-x86_64` when the profile is inactive.
- [ ] T005 Document **one** chosen precedence for Windows x86_64 natives (artifact unpack vs `hdf5.lib.dir` from `build.properties`) in `README.md` and align the same decision in `specs/002-hdf5-maven-deps/research.md` (per `plan.md` post-design re-check).

---

## Phase 3: User Story 1 — Build consumes the intended artifact line (Priority: P1) MVP

**Goal**: Resolver graph includes **`org.hdfgroup:hdf5-native:2.2.0`** and
**`org.hdfgroup:hdf5-java-ffm:2.2.0:windows-x86_64`** on Windows x86_64 with no silent version drift
(**FR-001–FR-003**).

**Independent Test**: On a Windows x86_64 machine with artifacts resolvable,
`mvn -q -DskipTests dependency:tree -pl object` lists both coordinates at **2.2.0**; on Linux,
`mvn -q -DskipTests validate` still passes.

- [ ] T006 [US1] On **Windows x86_64**, run `mvn -q -DskipTests dependency:tree -pl object` and fix `pom.xml` / `object/pom.xml` until both org.hdfgroup artifacts appear at **2.2.0** with the intended classifier for `hdf5-java-ffm`.
- [ ] T007 [US1] If `dependency:tree` shows **duplicate or conflicting** HDF5 Java stacks, add the minimal fix in `pom.xml` (Enforcer snippet, `<exclusions>`, or `dependencyManagement` ordering) and record the rationale in `specs/002-hdf5-maven-deps/research.md`.

**Checkpoint**: User Story 1 satisfied — Windows dependency line is pinned and observable.

---

## Phase 4: User Story 2 — Windows developers align with the supply line (Priority: P2)

**Goal**: Contributor docs explain the **2.2.0** org.hdfgroup line, pre-central resolution, and scope
for non-Windows (**FR-004**, **FR-005**).

**Independent Test**: A reader can name both coordinates and explain local cache vs Central from
`README.md` / `CLAUDE.md` alone.

- [ ] T008 [P] [US2] Add a **Windows x86_64** subsection to `README.md` documenting the two coordinates, how to populate `~/.m2` before Central, and pointer to `specs/002-hdf5-maven-deps/quickstart.md`.
- [ ] T009 [P] [US2] Add a short HDF5 **Maven coordinates** block to `CLAUDE.md` linking `specs/002-hdf5-maven-deps/quickstart.md` and stating **non-Windows** still follows existing `build.properties` / `hdf5.lib.dir` flow unless extended later.

**Checkpoint**: User Stories 1 and 2 independently verifiable (build graph + docs).

---

## Phase 5: User Story 3 — Transition to Central is frictionless (Priority: P3)

**Goal**: Docs state that **the same GAV** works pre-central and post-central (**SC-003**, User
Story 3).

**Independent Test**: One short passage in `README.md` explicitly compares “today” vs “after
Central” without changing coordinates.

- [ ] T010 [US3] Add the **same GAV before and after Maven Central** passage (≤ one screen) to `README.md` per `spec.md` SC-003.

---

## Phase 6: Polish & cross-cutting

**Purpose**: Contract alignment, optional packaging, CI smoke.

- [ ] T011 [P] Update R1/R2 rows in `specs/002-hdf5-maven-deps/contracts/developer-build-resolution.md` to match **verified** packaging and final `dependency:get` flags from T001 / implemented POMs.
- [ ] T012 Extend `hdfview/pom.xml` with Windows-only **unpack/copy** of `hdf5-native` payloads **only if** T001 shows packaging requires it for installers; otherwise add a one-line **N/A** rationale to `README.md` and skip code changes.
- [ ] T013 Confirm `.github/workflows/ci-windows.yml` (or the active Windows CI workflow) still succeeds for this branch after dependency changes (adjust workflow only if HDF Group artifacts must be primed on CI runners).

---

## Dependencies & execution order

### Phase dependencies

| Phase | Depends on | Notes |
|-------|------------|--------|
| 1 Setup | — | Start immediately |
| 2 Foundational | Phase 1 | Needs verified packaging notes |
| 3 US1 | Phase 2 | Dependency graph tasks |
| 4 US2 | Phase 2 | Docs can proceed in parallel with US1 **after** T005 precedence text exists |
| 5 US3 | Phase 4 | Same `README.md` as US2 — run T008/T009 before T010 to reduce merge friction |
| 6 Polish | US1–US3 | Contract + CI after behavior is stable |

### User story dependencies

- **US1**: Depends on Phase 2 only; no dependency on US2/US3.
- **US2**: Depends on Phase 2; **sequential vs US3 for `README.md`**: complete T008/T009 before T010.
- **US3**: Depends on US2 doc scaffolding in `README.md` (same file).

### Parallel opportunities

- **Phase 4**: T008 and T009 may run in parallel (different files).
- **Phase 6**: T011 may run in parallel with T012 (different files) once US1 POMs are stable.

### Parallel example: User Story 2

```text
# Different files, no mutual dependency:
- T008 [P] [US2] README.md Windows subsection
- T009 [P] [US2] CLAUDE.md coordinates block
```

### Parallel example: Phase 6

```text
- T011 [P] contracts/developer-build-resolution.md
- T012 hdfview/pom.xml OR README.md N/A note (exclusive by outcome of T001)
```

---

## Implementation strategy

### MVP (User Story 1)

1. Complete Phase 1 (T001) and Phase 2 (T002–T005).  
2. Complete Phase 3 (T006–T007).  
3. **Stop and validate** using `quickstart.md` / contract R1–R3 checks on Windows and Linux.

### Incremental delivery

1. MVP (US1) → mergeable supply-chain pin.  
2. Add US2 (docs) → contributor clarity.  
3. Add US3 (same-GAV passage) → Central transition story closed.  
4. Polish: contract sync, optional `hdfview` native unpack, CI confirmation.

### Parallel team strategy

- Developer A: Phase 2 POMs (T002–T004) after T001.  
- Developer B: T005 + Phase 4 docs (T008–T009) once T002–T003 skeleton exists.  
- Developer C: Phase 3 tree/enforcer (T006–T007) on Windows hardware or self-hosted runner.

---

## Task summary

| Metric | Value |
|--------|-------|
| **Total tasks** | 13 |
| **Phase 1 (Setup)** | 1 |
| **Phase 2 (Foundational)** | 4 |
| **US1 (P1)** | 2 |
| **US2 (P2)** | 2 (parallelizable pair) |
| **US3 (P3)** | 1 |
| **Polish** | 3 |
| **Format** | Every task uses `- [ ]`, sequential `T001`–`T013`, file paths in descriptions |

---

## Notes

- Do **not** remove `jarhdf5` in this feature unless a follow-up spec says so (`plan.md`).  
- Prefer **exact** `2.2.0` pins; avoid version ranges (**FR-003**).  
- If `repository/pom.xml` must align the local `jarhdf5` install path with the new line, treat it as
  a **follow-up** unless CI breaks without it.
