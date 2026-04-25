# Quickstart: verify HDF5 2.2.0 org.hdfgroup artifacts (per-platform classifiers)

Prerequisites: **JDK 25** and Maven per repository baseline; artifacts available to your resolver
(local install, mirror, or Maven Central).

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

## 5. Linux CI spot-check (from any OS)

```bash
mvn -q -DskipTests validate
```

Expected: Linux jobs do not resolve **`hdf5-java-ffm`** classifiers for **inactive** foreign
platforms (e.g. **`windows-x86_64`** when no Windows profile is active).
