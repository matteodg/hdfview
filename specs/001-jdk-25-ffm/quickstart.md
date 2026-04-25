# Quickstart: verify JDK 25 for HDFView development

Prerequisites: HDF native libraries configured per `build.properties` (unchanged by this
feature).

## 1. Install JDK 25

Install a JDK whose **major version is 25** (team default: **Eclipse Temurin 25**).

## 2. Point your shell at that JDK

- Ensure `java` and `javac` on `PATH` refer to JDK 25, **or**
- Set `JAVA_HOME` to the JDK 25 install root and prepend `$JAVA_HOME/bin` to `PATH`.

## 3. Verify

```bash
java -version
javac -version
mvn -version
```

Expected: version lines report **25** (not 21 or 24).

## 4. Build

From the repository root:

```bash
mvn clean compile -DskipTests
```

If the active JDK is too old, Maven Enforcer (once the feature is implemented) MUST fail with an
explicit **Java 25 required** message.

## 5. Optional: launcher smoke check

```bash
./run-hdfview.sh --validate
```

(On Windows, `run-hdfview.bat` and choose validation if prompted.) The script MUST accept only
Java **25+** after the feature lands.
