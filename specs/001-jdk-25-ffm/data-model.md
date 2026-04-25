# Phase 1 — Data model (governance & configuration)

This feature has no persistent application database. Entities are **configuration and policy
objects** enforced across build, CI, and docs.

## Entity: `JavaBaselinePolicy`

| Field | Type | Description |
|-------|------|-------------|
| `minimumMajorVersion` | integer | **25** — required for compile, test, documented runs |
| `enforcedBy` | set | `maven-compiler-plugin`, `maven-enforcer-plugin`, GitHub Actions
`setup-java`, `run-hdfview.*`, `.specify/memory/constitution.md` |
| `rationale` | text | Unlock FFM API for future native interop; single supported baseline |

**Validation rules**

- All declared enforcement points MUST agree on **25** (no file says 21 after merge).
- CI primary jobs MUST NOT install JDK &lt; 25 for compile/test.

## Entity: `MavenBuildProfile`

| Field | Type | Description |
|-------|------|-------------|
| `compilerRelease` | integer | **25** |
| `rootPomPath` | path | `pom.xml` |
| `modulePomPaths` | list | `object/pom.xml`, `hdfview/pom.xml` (only if they pin compiler
explicitly) |

**Relationships**

- `MavenBuildProfile` realizes `JavaBaselinePolicy` for command-line Maven users.

## Entity: `CiJavaMatrixRow`

| Field | Type | Description |
|-------|------|-------------|
| `workflowFile` | path | e.g. `.github/workflows/ci-linux.yml` |
| `javaVersion` | string | `'25'` passed to `actions/setup-java` |
| `distribution` | string | `temurin` (default) unless workflow-specific constraint |

**Relationships**

- Many `CiJavaMatrixRow` records collectively enforce `JavaBaselinePolicy` in automation.

## Entity: `ContributorDocSet`

| Field | Type | Description |
|-------|------|-------------|
| `files` | list | `README.md`, `CLAUDE.md`, optionally `docs/*` if JDK setup is duplicated |
| `verificationCommand` | string | Documented one-liner to print Java version / Maven Java home |

**Validation rules**

- Cross-doc scan: no remaining “Java 21” **requirement** statements unless explicitly historical
  (prefer removal or “HDFView x.y used Java 21” in changelog only—not in setup docs).
