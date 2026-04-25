# Phase 0 — Research: HDF5 Maven native + Java FFM (2.2.0, per-platform classifiers)

## 1. Where to declare `org.hdfgroup` artifacts

**Decision**: Declare versions in **root** `pom.xml` as properties (e.g. `hdfgroup.hdf5.native.version`
= **2.2.0**) and centralize GAV in **`dependencyManagement`**. Add concrete dependencies in
**`object/pom.xml`** behind **OS-scoped Maven profiles** so each CI/runtime OS resolves only its own
**`hdf5-java-ffm` classifier** (first concrete profile: **Windows x86_64** / `windows-x86_64`).

**Rationale**: Matches existing BOM-style root (`hdfview-bom`) and keeps module POMs thin; profile
activation matches **FR-005** (no foreign classifier on inactive platforms).

**Alternatives considered**:

- **Dependencies only in `hdfview/`** — rejected as primary: Java HDF API usage lives in **`object/`**
  first; UI module should not own API jars unless packaging-only unpack is required there.
- **No profile (always resolve)** — rejected: breaks non-Windows CI when artifacts are absent from
  public Central.

---

## 2. Coexistence with `jarhdf5` 2.0.0

**Decision**: Treat **`hdf5-java-ffm`** as the **FFM binding line** and **`jarhdf5`** as the
**existing JNI line** until implementation tasks explicitly migrate call sites. This feature’s
spec **requires declarations and docs**, not a full JNI removal.

**Rationale**: Avoids mixing two Java HDF5 APIs in one class without a designed adapter; minimizes
native crash risk during the first merge.

**Alternatives considered**:

- **Remove `jarhdf5` in the same PR** — high risk; out of scope unless spec is amended.
- **Shadow one artifact inside the other** — unnecessary complexity for v1 wiring.

---

## 3. Getting native binaries onto the runtime path (per platform)

**Decision**: Prefer **explicit unpack or copy** of payloads from **`hdf5-native`** into a
build-controlled directory (e.g. under `object/target/` or `hdfview/target/`) that **platform
packaging** already consumes, **or** document that developers set `hdf5.lib.dir` to the directory
produced by that unpack step for that OS—**one** source of truth per environment, never two
uncoordinated native copies on `PATH`.

**Rationale**: Satisfies spec story about avoiding hand-copied vendor DLLs in-repo while respecting
constitution II (no duplicate conflicting natives).

**Alternatives considered**:

- **Rely only on `java.library.path` magic from the JAR** — acceptable only if the published
  `hdf5-java-ffm` artifact documents supported loading; verify during implementation.
- **Always use system HDF5 in `Program Files`** — contradicts “published artifact line” goal for
  reproducible dev builds.

---

## 4. Enforcing FR-003 (no silent fallback)

**Decision**: Pin **exact** `2.2.0` (no property ranges). Optionally add **`maven-enforcer-plugin`**
rules or **`dependency:analyze`** in contributor docs to catch duplicate HDF5 natives. Use **one**
`<dependency>` entry per artifact without version ranges.

**Rationale**: Maven’s nearest-wins semantics can otherwise mask drift.

**Alternatives considered**:

- **Version range `[2.2.0,2.3.0)`** — rejected; violates “no silent substitution” spirit.

---

## 5. Pre–Maven Central contributor workflow

**Decision**: Document **`mvn install:install-file`** or internal **mirror `<mirrorOf>`** snippet
(whichever the team prefers) so identical coordinates work before Central publication; keep the
same GAV after Central goes live (**User Story 3**).

**Rationale**: Matches spec assumption that artifacts may exist only in local cache initially.

**Alternatives considered**:

- **Temporary `system` scope paths** — rejected; not portable and weak for CI.

---

## 6. Classifier and arch naming

**Decision**: Use **`windows-x86_64`** on **`hdf5-java-ffm`** for 64-bit Windows as the **reference**
classifier. **Assume HDF Group publishes correspondent classifiers** for other OS/arch targets at the
same **`2.2.0`** version; add additional Maven profiles using the same pattern when those artifacts
are available in the resolver. Confirm **`hdf5-native`** packaging (classifier/type) from the
published POM; adjust POM `type` if HDF Group uses non-default packaging for native bundles.

**Rationale**: Keeps one artifact ID with **per-platform classifiers**—validate each tuple against
the local `.m2` (or Central) layout during `/speckit-implement`.

**Alternatives considered**:

- **Guess classifier on `hdf5-native`** — do not guess; read the installed POM once during
  implementation.

---

## 7. Verified packaging (implementation, 2026-04-26)

**Source**: Local `~/.m2/repository/org/hdfgroup/` layout and upstream POMs at **2.2.0**.

| Artifact | Packaging in POM | Resolved JAR on disk | `dependency:get` notes |
|----------|-------------------|----------------------|-------------------------|
| `hdfgroup:hdf5-native:2.2.0` | `jar` | `hdf5-native-2.2.0-windows-x86_64.jar` | Requires **`-Dclassifier=windows-x86_64`** (no default `hdf5-native-2.2.0.jar` in repo). |
| `hdfgroup:hdf5-java-ffm:2.2.0` | `jar` | `hdf5-java-ffm-2.2.0-windows-x86_64.jar` | **`-Dclassifier=windows-x86_64`** (plus `-Dpackaging=jar`). |

**Dependency graph guard (constitution III)**: **Manual** verification for this delivery—run
`mvn -DskipTests dependency:tree -pl object` on the target OS and confirm org.hdfgroup entries show
**2.2.0** with the expected classifier. **`jarhdf5:2.0.0`** remains on the classpath alongside the
new line until a follow-on removes JNI; watch for incompatible duplicate `hdf5.dll` if
`hdf5.lib.dir` points at a different HDF5 major than the FFM bundle (see README precedence).
