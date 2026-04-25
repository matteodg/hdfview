# Phase 1 — Data model (build & supply chain)

No application database. Entities describe **Maven coordinates**, **platform scope**, and
**documentation** obligations from `spec.md`.

## Entity: `Hdf5NativeArtifact`

| Field | Type | Description |
|-------|------|-------------|
| `groupId` | string | **`org.hdfgroup`** |
| `artifactId` | string | **`hdf5-native`** |
| `version` | semver string | **`2.2.0`** (exact) |
| `packaging` | string | As published by HDF Group (verify: `jar` vs `pom` vs other) |

**Validation rules**

- MUST match **FR-001** exactly (no range, no different group).

**Relationships**

- Supplies **native binaries** consumed by Windows packaging or unpack steps (see `research.md`).

## Entity: `Hdf5JavaFfmArtifact`

| Field | Type | Description |
|-------|------|-------------|
| `groupId` | string | **`org.hdfgroup`** |
| `artifactId` | string | **`hdf5-java-ffm`** |
| `version` | semver string | **`2.2.0`** |
| `classifier` | string | **`windows-x86_64`** |

**Validation rules**

- MUST match **FR-002** and **FR-003** (pinned version + fixed classifier for the Windows FFM line).

**Relationships**

- Provides **Java FFM** bindings intended for JDK 25+ code paths (consumption may be phased).

## Entity: `WindowsX86_64MavenProfile`

| Field | Type | Description |
|-------|------|-------------|
| `profileId` | string | Stable id (e.g. `windows-x86_64-hdf5-ffm`) |
| `activation` | struct | Maven `os` activation: Windows family + amd64/x86_64 arch |
| `dependencies` | list | References to `Hdf5NativeArtifact` and `Hdf5JavaFfmArtifact` |

**Validation rules**

- When profile is **inactive** (non-Windows CI), build MUST NOT require Central access to these
  artifacts.

## Entity: `ResolverVisibilityNote`

| Field | Type | Description |
|-------|------|-------------|
| `preCentral` | text | How to populate local cache / internal mirror |
| `postCentral` | text | Same coordinates resolve from default public resolver |
| `publishedSurfaces` | set | Files that MUST contain the note (**SC-003**), e.g. `README.md` |

**Relationships**

- Documents **FR-004** and **User Story 3**.

## Entity: `LegacyJarHdf5Line` (context, not removed by spec)

| Field | Type | Description |
|-------|------|-------------|
| `groupId` | string | **`jarhdf5`** |
| `versionProperty` | string | **`${hdf5.version}`** (currently **2.0.0** at root `pom.xml`) |

**Validation rules**

- **FR-005**: Docs MUST clarify whether Windows devs still touch `hdf5.lib.dir` for JNI path while
  FFM artifacts are present.

**Relationships**

- Coexists with `Hdf5JavaFfmArtifact` until a future migration removes JNI dependency.
