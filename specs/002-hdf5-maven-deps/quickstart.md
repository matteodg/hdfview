# Quickstart: verify HDF5 2.2.0 org.hdfgroup artifacts (per-platform classifiers)

**Update (003)**: HDFView no longer uses the legacy **`jarhdf5:jarhdf5`** coordinate; HDF5 Java is
**`org.hdfgroup:hdf5-java-ffm`** only. See **`specs/003-migrate-hdf5-ffm/quickstart.md`** for the
post-migration verification checklist and GitHub Packages authentication notes.

Prerequisites: **JDK 25** and Maven per repository baseline; artifacts available to your resolver
(GitHub Packages with PAT, local `~/.m2`, mirror, or Maven Central when published).

## 1. Confirm platform

**Reference path**: **64-bit Windows** (x86_64) using classifier **`windows-x86_64`** on
`hdf5-java-ffm`. **Other OS/arch**: assume HDF Group publishes **correspondent classifiers** for the
same **`org.hdfgroup:hdf5-java-ffm:2.2.0`** coordinates—use the classifier for **your** active Maven
profile when running **R2**-style checks from
[`contracts/developer-build-resolution.md`](./contracts/developer-build-resolution.md).

## 2. Pre-populate local Maven cache (pre–Maven Central only)

If the jars are not on Central yet, install them into `~/.m2/repository` using the method documented
in **`README.md`** after the feature lands (e.g. `mvn install:install-file` with the exact GAV, or
mirror configuration).

## 3. Resolve the two coordinates

From the repository root, run the **`dependency:get`** (or **`dependency:tree`**) commands listed in
[`contracts/developer-build-resolution.md`](./contracts/developer-build-resolution.md) **R1** and
**R2**.

Expected: **no** resolution to a version other than **2.2.0** and no missing-classifier errors for
the FFM artifact.

## 4. Full compile (Windows)

```bash
mvn clean compile -DskipTests
```

Expected: Windows profile activates; build completes if other prerequisites (`build.properties`,
HDF4 paths, etc.) remain satisfied per existing project rules.

## 5. Linux / macOS spot-check (from any OS)

```bash
mvn -q -DskipTests validate
```

Expected: **inactive** foreign classifiers are not pulled (e.g. **`windows-x86_64`** on Linux).

On **Linux amd64** or **macOS** with GitHub Packages credentials configured (see root `pom.xml`
`<repositories>` and `README.md`):

```bash
mvn -q -DskipTests dependency:tree -pl object
```

Expect **`org.hdfgroup:hdf5-java-ffm:jar:…:2.2.0:compile`** with the **host** classifier
(**`linux-x86_64`**, **`macos-aarch64`**, etc.) and **no** `jarhdf5:jarhdf5`.

## 6. When artifacts are missing (expected Maven output)

If **`org.hdfgroup`** artifacts are not in your resolver, Maven typically fails during
**`validate`**, **`compile`**, or **`dependency:get`** with messages similar to:

- `Could not find artifact org.hdfgroup:hdf5-native:jar:windows-x86_64:2.2.0 in ...`
- `Could not resolve dependencies for project ...`
- `was not found in https://repo.maven.apache.org/maven2 ... during a previous attempt ...`

Fix: install the JARs locally, add an internal mirror, or wait until Maven Central hosts the same
coordinates (see `README.md`).

## 7. Repeatable verification (SC-001 / SC-002)

These commands operationalize the success criteria in `spec.md` for maintainers.

**SC-001 — dependency graph at 2.2.0 (Windows x86_64 active)**:

```bash
mvn -DskipTests dependency:tree -pl object
```

Expect lines containing:

- `org.hdfgroup:hdf5-native:jar:windows-x86_64:2.2.0:compile`
- `org.hdfgroup:hdf5-java-ffm:jar:windows-x86_64:2.2.0:compile`

**SC-002 — docs name the same coordinates** (from repo root; requires `rg` or use `grep -E`):

```bash
rg -n "org\.hdfgroup:hdf5-native|org\.hdfgroup:hdf5-java-ffm|windows-x86_64|2\.2\.0" README.md CLAUDE.md
```

Expect hits in both files with **no** second conflicting HDF5 **2.2.0** story for the Windows path.
