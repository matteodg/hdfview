# Feature Specification: HDF5 native and Java FFM via published artifacts

**Feature Branch**: `002-hdf5-maven-deps`  
**Created**: 2026-04-26  
**Status**: Draft  
**Input**: User description: "I'd like you to use these two Maven dependencies (present in the local .m2 repo, but they will be available in Maven central in future): org.hdfgroup:hdf5-native:2.2.0; org.hdfgroup:hdf5-java-ffm:2.2.0 classifier windows-x86_64"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Build consumes the intended HDF5 artifact line (Priority: P1)

A release or build maintainer updates the project so that obtaining **HDF5 native libraries** and
**Java Foreign Function and Memory (FFM) bindings** for HDF5 happens through the **HDF Group’s
published version 2.2.0 artifacts**, using the same identifiers the team expects to publish to the
central package repository later. Today those artifacts may live only in a maintainer’s local
cache; tomorrow the same identifiers resolve from the central repository without changing the
declared versions.

**Why this priority**: Wrong or ad hoc native packaging is a top source of “works on my machine”
failures and security supply-chain risk; pinning to the official org.hdfgroup 2.2.0 line is the
foundation for reproducible Windows x86_64 builds.

**Independent Test**: On a machine where the two declared artifacts are resolvable (local cache or
mirror with equivalent coordinates), a clean dependency resolution step lists **2.2.0** for both
the native bundle and the Java FFM binding with the **Windows x86_64** platform selection, with
no competing duplicate HDF5 native lines at a different version for that path.

**Acceptance Scenarios**:

1. **Given** a clean checkout and a resolver configuration that can see the declared artifacts,
   **When** the maintainer runs the project’s standard dependency resolution for the HDFView build,
   **Then** the build pulls `org.hdfgroup:hdf5-native:2.2.0` and
   `org.hdfgroup:hdf5-java-ffm:2.2.0` with classifier `windows-x86_64` (and no silent substitution to
   a different version for those coordinates).
2. **Given** the artifacts are not visible to the resolver, **When** the maintainer runs the same
   step, **Then** they see a clear resolution failure that names the missing components (rather than
   a vague native crash later).

---

### User Story 2 - Windows x86_64 developers align with the new supply line (Priority: P2)

A contributor on **64-bit Windows** follows updated guidance and obtains HDF5 native and Java FFM
support through the same **2.2.0** artifact line, without hand-copying DLLs into the tree for the
standard developer workflow where this feature applies.

**Why this priority**: Developers are the daily consumers of dependency declarations; the Windows
x86_64 path called out by the classifier must be understandable and repeatable.

**Independent Test**: A contributor reads only the documentation touched by this feature and can
state which published artifact pair supplies HDF5 for Windows x86_64 at version 2.2.0, and how
that relates to local cache versus central availability.

**Acceptance Scenarios**:

1. **Given** contributor documentation updated in this delivery, **When** a Windows x86_64
   developer follows it, **Then** they can confirm they are on the **2.2.0** org.hdfgroup artifacts
   without undocumented side channels.
2. **Given** another CPU/OS combination the project still supports, **When** that developer reads
   the docs, **Then** they see whether this feature changes their path or defers to existing
   documented native setup (no contradictory “only use artifacts” message where legacy paths remain).

---

### User Story 3 - Transition to central publishing is frictionless (Priority: P3)

When the HDF Group publishes the same coordinates to the central package repository, **no
identifier or version change** is required in this repository for the Windows x86_64 line—only
resolver or mirror configuration may be simplified.

**Why this priority**: Avoids a second churn PR when central goes live.

**Independent Test**: A table or short note in assumptions or contributor docs states that local
resolution today and central resolution later use the **same** group, artifact names, version, and
classifier.

**Acceptance Scenarios**:

1. **Given** artifacts become available from the default public resolver, **When** a maintainer
   removes optional mirror/local-only setup that was only needed pre-central, **Then** the declared
   coordinates in the project remain **2.2.0** with the same identifiers.

### Edge Cases

- Resolver has **partial** visibility (only one of the two artifacts): failure must be explicit
  before compile/link/runtime.
- **Multiple HDF5 versions** on the machine: the build must not accidentally pick up a system-wide
  install when the intent is the 2.2.0 artifact line for the scoped path.
- **Platforms without an active Maven profile** for their `hdf5-java-ffm` classifier: behavior must
  remain defined (unchanged legacy path or documented gap) without breaking their existing supported
  workflow, and inactive profiles must not pull **foreign** platform classifiers.
- **Pre-central period**: contributors may need a one-time install into local cache or an internal
  mirror; docs should say so without implying the artifacts are universally public yet.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The project MUST declare a dependency on **`org.hdfgroup:hdf5-native:2.2.0`** so that
  HDF5 native material for the build is sourced from that published line.
- **FR-002**: The project MUST declare a dependency on **`org.hdfgroup:hdf5-java-ffm:2.2.0`** with
  the **platform classifier** that matches the build target OS and CPU architecture. The user
  supplied **`windows-x86_64`** as the canonical example for 64-bit Windows; **other platforms are
  assumed to have correspondent classifiers** published by HDF Group for the same artifact and
  version (same GAV except classifier).
- **FR-003**: Dependency resolution for the **intended** `hdf5-java-ffm` platform classifier MUST
  **not** silently fall back to a different version or classifier when **2.2.0** and that classifier
  are intended.
- **FR-004**: Contributor-facing documentation MUST mention that these coordinates may initially
  resolve only from a **local cache or private mirror**, and later from the **central package
  repository**, without changing the declared version or identifiers for this feature.
- **FR-005**: Documentation MUST state the **one-classifier-per-target-platform** rule for
  **`hdf5-java-ffm`** (with **`windows-x86_64`** as the documented example) and MUST clarify which
  platforms are wired in this delivery versus deferred, so readers do not assume a single classifier
  applies to every OS or that every OS is wired in the same change set.

### Key Entities

- **HDF5 native artifact (`hdf5-native`)**: The published bundle that supplies native HDF5 binaries
  for the build at version **2.2.0** under group **`org.hdfgroup`**.
- **HDF5 Java FFM artifact (`hdf5-java-ffm`)**: The published Java bindings using FFM for HDF5 at
  version **2.2.0**, with **one platform classifier per OS/arch** (example: **`windows-x86_64`**;
  other platforms use their **correspondent** HDF Group classifiers at the same version).
- **Resolver visibility**: Where the artifacts are found today (local/mirror) versus later
  (central), holding identifiers constant.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: On a reference Windows x86_64 environment configured per updated docs, **100%** of
  the **documented** “fetch build dependencies” checks (the exact commands listed in
  `specs/002-hdf5-maven-deps/quickstart.md` after implementation, e.g. `dependency:tree` /
  `dependency:get` for the two coordinates) show **2.2.0** resolved for both the native and Java FFM
  artifact identifiers in scope for this feature (no drift to another version in that check).
- **SC-002**: **100%** of sampled documentation paths that describe HDF5 acquisition for Windows
  x86_64 name the **same** org.hdfgroup identifiers and **2.2.0** version (no stale conflicting
  version numbers in the same doc set for that path).
- **SC-003**: Published contributor or maintainer notes include a **single short passage** (roughly
  one screen or less) that states how resolution works **before** central publishing versus
  **after**, without changing the declared **2.2.0** coordinates or the **per-platform classifier
  pattern** (using the Windows x86_64 line as the worked example).

## Assumptions

- **Build system** for declaring dependencies remains the repository’s current Java build (no change
  of build tool required by this specification alone).
- **Platform classifiers**: HDF Group is assumed to publish **`hdf5-java-ffm:2.2.0`** with a
  **correspondent classifier per target platform** (same scheme as **`windows-x86_64`** for
  64-bit Windows). This delivery may wire **one or more** platforms; the user-supplied classifier
  is the **reference** for Windows x86_64. Platforms not yet wired may keep existing
  `build.properties` / `hdf5.lib.dir` documentation until the same Maven pattern is extended.
- **Version 2.2.0** is the single supported line for this adoption; bumping to a newer HDF5 release
  is out of scope unless explicitly requested later.
- Artifacts are **trusted** first-party HDF Group publications; license and redistribution rules
  follow the HDF Group’s terms for those artifacts (no additional legal review assumed beyond normal
  project due diligence).
