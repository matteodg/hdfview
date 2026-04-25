HDFView version 3.4.0 currently under development

![HDF5 Logo](src/HDFView.png)

## Build Status

[![CI Orchestrator](https://github.com/HDFGroup/hdfview/actions/workflows/maven-ci-orchestrator.yml/badge.svg)](https://github.com/HDFGroup/hdfview/actions/workflows/maven-ci-orchestrator.yml)
[![Linux Build](https://github.com/HDFGroup/hdfview/actions/workflows/ci-linux.yml/badge.svg)](https://github.com/HDFGroup/hdfview/actions/workflows/ci-linux.yml)
[![Windows Build](https://github.com/HDFGroup/hdfview/actions/workflows/ci-windows.yml/badge.svg)](https://github.com/HDFGroup/hdfview/actions/workflows/ci-windows.yml)
[![macOS Build](https://github.com/HDFGroup/hdfview/actions/workflows/ci-macos.yml/badge.svg)](https://github.com/HDFGroup/hdfview/actions/workflows/ci-macos.yml)

[![Code Quality](https://github.com/HDFGroup/hdfview/actions/workflows/maven-quality.yml/badge.svg)](https://github.com/HDFGroup/hdfview/actions/workflows/maven-quality.yml)
[![Security Scan](https://github.com/HDFGroup/hdfview/actions/workflows/maven-security.yml/badge.svg)](https://github.com/HDFGroup/hdfview/actions/workflows/maven-security.yml)

The HDF Group is the developer, maintainer, and steward of HDFView. Find more
information about The HDF Group, the HDFView Community, and other HDF software projects,
tools, and services at [The HDF Group's website](https://www.hdfgroup.org/). 


HELP AND SUPPORT
----------------
Information regarding Help Desk and Support services is available at

   https://help.hdfgroup.org/


FORUM and NEWS
--------------
The [HDF Forum](https://forum.hdfgroup.org) is provided for public announcements and discussions
of interest to the general HDFView Community.

   - News and Announcements
   https://forum.hdfgroup.org/c/news-and-announcements-from-the-hdf-group

   - HDFView (and Java) Topics
   https://forum.hdfgroup.org/c/hdfview-java-hdf-object-package

These forums are provided as an open and public service for searching and reading.
Posting requires completing a simple registration and allows one to join in the
conversation.  Please read the [instructions](https://forum.hdfgroup.org/t/quickstart-guide-welcome-to-the-new-hdf-forum
) pertaining to the Forum's use and configuration.

RELEASE SCHEDULE
----------------

![HDFView release schedule](docs/img/release-schedule.png) 

HDFView releases about once a year, following the most recent HDF5 and HDF4 releases.
Future HDFView releases indicated on this schedule are tentative.

**NOTE**: All HDFView releases are now based on the latest maintenance releases
of HDF5 and HDF4. Previous releases of HDFView that were based on HDF5 1.8,
1.10, and 1.12 (e.g. 3.1.x, 3.2.x) have been retired.

| Release | HDF5 | HDF4 | New Features |
| ------- | ---- | ---- | ------------ |
| 3.3.0 | 1.14.0 | 4.2.16 | HDF5 1.12 (new-style) references, Single-Writer/Multiple-Readers (SWMR) reads, bug fixes |
| 3.3.1 | 1.14.2 | 4.2.16-2 | Fixes a critical HDF4 + HDFView bug |
| 3.3.2 | 1.14.4 | 4.3.0 | Float16 support |
| 3.4.0 | 2.0.0 | 4.3.1 | Complex number support |


PREVIOUS RELEASES AND SOURCE CODE
--------------------------------------------
Source packages for current and previous releases are located at:
    
   https://support.hdfgroup.org/downloads/

Development code is available at our Github location:
    
   https://github.com/HDFGroup/hdfview.git


## HDF5 2.2.0 — org.hdfgroup Maven artifacts (Java FFM)

The **`object`** module can resolve **The HDF Group**’s published **HDF5 2.2.0** line from Maven:

- **`org.hdfgroup:hdf5-native:2.2.0`** — platform **classifier** on the JAR (e.g. **`windows-x86_64`**
  on 64-bit Windows). Other OS/arch pairs use **correspondent classifiers** from HDF Group at the
  same version when wired in `pom.xml`.
- **`org.hdfgroup:hdf5-java-ffm:2.2.0`** — same **per-platform classifier** pattern (reference:
  **`windows-x86_64`**).

**Before Maven Central**: install the JARs into `~/.m2` (e.g. `mvn install:install-file` with the
exact GAV + classifier) or use a corporate mirror. **After Central**: the same coordinates resolve
from the default public repository—no POM version change required.

Detailed commands and checks: **`specs/002-hdf5-maven-deps/quickstart.md`**.

### Native precedence (Windows)

HDFView still copies HDF5 JNI DLLs from **`hdf5.lib.dir`** (see `build.properties`) via the existing
**Windows** profile in `object/pom.xml` for **`jarhdf5`**. The org.hdfgroup **`hdf5-native`** JAR
also carries **bundled** HDF5 for the **FFM / SciJava native-lib-loader** classpath path. Prefer
**aligning `hdf5.lib.dir` with HDF5 2.2.0** while both stacks are present, or plan migration off JNI,
to reduce risk of loading **two incompatible `hdf5.dll`** builds.

### Optional `hdfview` unpack of `hdf5-native`

**N/A for this change** — natives are bundled inside the org.hdfgroup artifacts; no extra unpack
step was added to `hdfview/pom.xml`.

### Missing resolution errors

If Maven cannot download the org.hdfgroup artifacts, see **“When artifacts are missing”** in
`specs/002-hdf5-maven-deps/quickstart.md` for representative log lines (`Could not find artifact`,
`Could not resolve dependencies`, etc.).
