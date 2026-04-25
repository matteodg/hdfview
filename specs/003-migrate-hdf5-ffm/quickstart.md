# Quickstart: Verify jarhdf5 → hdf5-java-ffm migration

Use this after implementation lands to confirm **spec SC-002**, **FR-004**, and [contracts/single-hdf5-java-binding.md](./contracts/single-hdf5-java-binding.md).

## 0. GitHub Packages authentication (local & CI)

HDF Group publishes **`org.hdfgroup:hdf5-java-ffm`** / **`hdf5-native`** to **GitHub Packages** before
they appear on Maven Central. The root `pom.xml` includes repository id **`hdfgroup-github-packages`**.

Add to **`~/.m2/settings.xml`** (use a GitHub PAT with **`read:packages`** scope):

```xml
<settings>
  <servers>
    <server>
      <id>hdfgroup-github-packages</id>
      <username>YOUR_GITHUB_USERNAME</username>
      <password>YOUR_PAT</password>
    </server>
  </servers>
</settings>
```

CI workflows use **`secrets.HDFGROUP_PACKAGES_USER`** and **`secrets.HDFGROUP_PACKAGES_TOKEN`** with
`actions/setup-java` `server-id: hdfgroup-github-packages`.

## 1. Resolver sanity (`object`)

```bash
cd /path/to/hdfview
mvn -q -DskipTests dependency:tree -pl object
```

- Expect **`org.hdfgroup:hdf5-java-ffm:2.2.0`** with your platform’s classifier.
- Expect **no** `jarhdf5:jarhdf5`.

## 2. Native + properties

Ensure `build.properties` (or CI env) still points **`hdf5.lib.dir`** / **`platform.hdf.lib`** at HDF5 **2.2.x** natives compatible with the FFM artifact. Wrong natives → load errors (constitution II); fix paths before blaming Java.

## 3. Tests (`object`)

```bash
mvn -q test -pl object
```

If you see **`IllegalCallerException`** related to restricted native access, check Surefire `argLine` includes **`--enable-native-access=ALL-UNNAMED`** (and existing `--add-opens` lines).

## 4. App smoke

```bash
./run-hdfview.sh --validate   # or your usual launch path
```

Open one HDF5 sample from test resources; browse a dataset.

## 5. Repo hygiene

```bash
rg "jarhdf5" -g "*.xml" -g "*.yml" -g "*.sh" -g "*.bat" -g "*.md"
```

Trend: **no** `jarhdf5` coordinates in POMs/workflows; docs reference **org.hdfgroup** FFM line only (except explicitly archived notes).
