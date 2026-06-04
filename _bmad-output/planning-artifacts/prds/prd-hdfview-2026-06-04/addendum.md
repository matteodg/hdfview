# Addendum — Technical reference (not PRD requirements)

## Target HDF5 Maven coordinates (org.hdfgroup 2.2.0)

User-supplied dependencies already in local Maven repository. Classifier example: `windows-x86_64` (platform classifiers expected per OS/arch).

```xml
<dependency>
    <groupId>org.hdfgroup</groupId>
    <artifactId>hdf5-native</artifactId>
    <version>2.2.0</version>
    <classifier>windows-x86_64</classifier>
</dependency>
<dependency>
    <groupId>org.hdfgroup</groupId>
    <artifactId>hdf5-szip-native</artifactId>
    <version>2.2.0</version>
    <classifier>windows-x86_64</classifier>
</dependency>
<dependency>
    <groupId>org.hdfgroup</groupId>
    <artifactId>hdf5-zlib-native</artifactId>
    <version>2.2.0</version>
    <classifier>windows-x86_64</classifier>
</dependency>
<dependency>
    <groupId>org.hdfgroup</groupId>
    <artifactId>hdf5-java-ffm</artifactId>
    <version>2.2.0</version>
    <classifier>windows-x86_64</classifier>
</dependency>
<dependency>
    <groupId>org.hdfgroup</groupId>
    <artifactId>javahdf5</artifactId>
    <version>2.2.0</version>
</dependency>
```

**Replaces (Phase 2):** legacy `jarhdf5` JNI artifact, `repository/` module, and manual `build.properties` HDF5 native path wiring.

**Unchanged in Phase 2:** HDF4 continues to use JNI; `build.properties` (or equivalent) still required for HDF4 native paths on Windows x86_64.

**Platform scope (Phase 2):** Windows x86_64 only for org.hdfgroup classified artifacts; Linux/macOS classifiers deferred.

**Current baseline (brownfield):** Java 21 in `pom.xml`; runtime tested on Java 25; HDF5 natives 2.1.1 via `build.properties`; code uses `hdf.hdf5lib` / `jarhdf5`.
