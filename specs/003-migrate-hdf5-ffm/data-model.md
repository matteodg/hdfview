# Phase 1 — Data model (HDF5 Java binding migration)

Entities describe **supply chain**, **runtime JVM policy**, and **verification** obligations from [spec.md](./spec.md). HDF4 (`jarhdf`) is out of scope except where JVM flags are shared.

## Entity: `Hdf5JavaBinding` (target)

| Field | Type | Description |
|-------|------|-------------|
| `groupId` | string | **`org.hdfgroup`** |
| `artifactId` | string | **`hdf5-java-ffm`** |
| `version` | semver string | **`2.2.0`** (exact; property-driven from root POM) |
| `classifier` | string | **Active platform only** (e.g. **`windows-x86_64`**, **`linux-x86_64`**, **`macos-aarch64`** — use HDF Group–published correspondent strings) |

**Validation rules**

- MUST appear on compile/test classpath for every module that imports `hdf.hdf5lib` for HDF5.
- MUST NOT coexist on the same resolved graph with **`jarhdf5:jarhdf5`** (spec FR-004).

**Relationships**

- Pairs with **`Hdf5NativeBundle`** for the same platform classifier.

## Entity: `Hdf5NativeBundle`

| Field | Type | Description |
|-------|------|-------------|
| `groupId` | string | **`org.hdfgroup`** |
| `artifactId` | string | **`hdf5-native`** |
| `version` | semver string | **`2.2.0`** |
| `classifier` | string | Same platform convention as `Hdf5JavaBinding` |

**Validation rules**

- Native material version/layout MUST match what **`hdf5-java-ffm`** expects for that release (constitution II).

## Entity: `LegacyJarhdf5Binding` (removed)

| Field | Type | Description |
|-------|------|-------------|
| `groupId` | string | **`jarhdf5`** |
| `artifactId` | string | **`jarhdf5`** |
| `version` | string | Historically **`2.0.0`** via `${hdf5.version}` |

**Validation rules**

- MUST NOT appear in **`object`**, **`hdfview`**, or parent BOM effective dependency trees after migration.
- MUST NOT be installed by CI **`install-file`** steps for in-scope builds.

## Entity: `JvmNativeAccessPolicy`

| Field | Type | Description |
|-------|------|-------------|
| `ffmNativeAccess` | string | **`--enable-native-access=ALL-UNNAMED`** for HDF5 FFM (per HDF Group docs) |
| `hdf4NativeAccess` | string | **`--enable-native-access=jarhdf`** while HDF4 remains JNI-based (existing pattern) |

**Validation rules**

- Surefire, exec-maven-plugin, jpackage-related JVM configs, and `run-hdfview.*` MUST be audited so **FFM** callers are not blocked by `IllegalCallerException`.

## Entity: `RegressionScenario`

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Stable test or manual checklist id |
| `surface` | enum | `object-test` \| `hdfview-manual` \| `ci` |
| `hdf5Feature` | string | Short description (open file, dataset read, attribute, reference, plugin, etc.) |

**Validation rules**

- SC-001: scenarios that passed pre-migration must pass post-migration or have an explicit waiver.
