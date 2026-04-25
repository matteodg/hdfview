# Phase 0 — Research: HDF5 Maven native + Java FFM (2.2.0, Windows x86_64)

## 1. Where to declare `org.hdfgroup` artifacts

**Decision**: Declare versions in **root** `pom.xml` as properties (e.g. `hdfgroup.hdf5.native.version`
= **2.2.0**) and centralize GAV in **`dependencyManagement`**. Add concrete dependencies in
**`object/pom.xml`** behind a **Windows x86_64** Maven profile so Linux CI does not resolve
unavailable classifiers.

**Rationale**: Matches existing BOM-style root (`hdfview-bom`) and keeps module POMs thin; profile
activation matches **FR-005** (other OS unchanged).

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

## 3. Getting native DLLs onto the runtime path (Windows)

**Decision**: Prefer **explicit unpack or copy** of payloads from **`hdf5-native`** into a
build-controlled directory (e.g. under `object/target/` or `hdfview/target/`) that **Windows
packaging** already consumes, **or** document that on Windows x86_64 developers set
`hdf5.lib.dir` to the directory produced by that unpack step—**one** source of truth per
environment, never two uncoordinated copies on `PATH`.

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

**Decision**: Use the user-supplied classifier **`windows-x86_64`** on **`hdf5-java-ffm`** verbatim.
Confirm **`hdf5-native`** artifact has **no classifier** (default JAR/POM type per HDF Group
packaging); adjust POM `type` if the published artifact uses a **pom** packaging for native bundles.

**Rationale**: Spec names exact coordinates; implementation must match published POMs—validate
against the local `.m2` copy during `/speckit-implement`.

**Alternatives considered**:

- **Guess classifier on `hdf5-native`** — do not guess; read the installed POM once during
  implementation.
