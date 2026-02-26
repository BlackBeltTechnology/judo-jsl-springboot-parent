# JUDO JSL Springboot Parent - Project Documentation

## Project Overview

**Repository:** BlackBeltTechnology/judo-jsl-springboot-parent
**License:** Eclipse Public License 2.0 (EPL-2.0)
**Java Version:** 21
**Build System:** Maven 3.9.4+ with Maven Wrapper (`./mvnw`)

1. **Parent POM for JUDO JSL Spring Boot applications** — provides all dependency versions, plugin configurations, and build lifecycle that child projects inherit
2. **Code generation pipeline** — the Tatami JSL workflow plugin compiles `.jsl` model files into Java SDK classes, DAO interfaces, and Liquibase database changelogs during `generate-sources`
3. **Dual database support** — pre-configured for both HSQLDB (in-memory, default for development/testing) and PostgreSQL (production)
4. **Part of the JUDO ecosystem** — a building block of the [judo-community](https://github.com/BlackBeltTechnology/judo-community) aggregator project

## Directory Structure

```
judo-jsl-springboot-parent/
├── pom.xml                  # The parent POM — the core artifact of this project
├── .github/
│   ├── workflows/           # 10 GitHub Actions CI/CD workflow files
│   ├── CIFLOW.md            # CI/CD branching and versioning documentation
│   └── ISSUE_TEMPLATE/      # GitHub issue templates
├── .mvn/                    # Maven wrapper configuration
│   └── wrapper/
├── logback-test.xml         # Logback configuration for test execution
├── README.md                # Project documentation and bootstrap guide
├── CONTRIBUTING.md          # Contribution guidelines
├── LICENSE.txt              # EPL 2.0 license text
└── .gitignore               # Git ignore rules
```

## Core Modules

This project has **no modules** — it is a single parent POM (`<packaging>pom</packaging>`). Child projects that inherit from it typically contain:

| Child Project Component | Location | Purpose |
|------------------------|----------|---------|
| JSL model file | `src/main/resources/model/<modelName>.jsl` | Domain model definition in JUDO Specific Language |
| Spring Boot app class | `src/main/java/.../<ModelName>SpringApplication.java` | Application entry point |
| Application config | `src/main/resources/application.properties` | DataSource and Liquibase configuration |
| Tests | `src/test/java/.../<ModelName>SpringApplicationTests.java` | Spring Boot integration tests using generated DAOs |
| Generated sources | `target/generated-sources/model/` | SDK classes, DAOs, Liquibase changelogs (auto-generated) |

## Technology Stack

### Core Technologies
- **Spring Boot 3.5.0** — application framework with starters for JDBC, logging, and testing
- **JUDO Runtime Core** (`1.0.6.x`) — DAO layer, dispatcher, access manager, RDBMS support
- **JUDO Tatami JSL** (`1.1.4.x`) — JSL model compiler and code generation workflow
- **Tatami Core** (`1.1.4.x`) — model transformation engine (JSL → PSM → ASM → RDBMS)
- **Eclipse EMF** (Ecore 2.38.0, Common 2.41.0, XMI 2.16.0) — meta-modeling framework
- **Epsilon Runtime** (`2.8.0.x`) — model-to-model and model-to-text transformation
- **Liquibase** — database schema migration (changelogs auto-generated from model)
- **Atomikos** (4.0.6 / 6.0.0 Spring Boot 3 starter) — JTA distributed transaction management
- **HSQLDB 2.6.1** — in-memory database for development and testing
- **PostgreSQL** — production database support
- **Lombok 1.18.34** — annotation-based boilerplate reduction

### Build & Quality
- **Maven 3.9.4+** with Maven Wrapper
- **JUnit 5** (Jupiter 5.8.2) — test framework
- **Mockito 4.8.0** — mocking framework
- **Hamcrest 2.2** — assertion matchers
- **JaCoCo 0.8.12** — code coverage reporting
- **SonarQube** (Maven plugin 3.9.1.2184) — static analysis
- **Flatten Maven Plugin 1.3.0** — CI-friendly `${revision}` POM flattening

### Logging
- **SLF4J 2.0.16** — logging facade
- **Logback 1.5.12** — logging implementation
- **JUL-to-SLF4J bridge** — redirects java.util.logging
- **Sysout-over-SLF4J** — captures System.out/err to logging

## Build Commands

```bash
# Full build and install to local Maven repository
./mvnw clean install

# Run tests only
./mvnw clean test

# Build with a specific version
./mvnw clean install -Drevision=1.0.5

# Deploy to Judong Nexus (snapshots)
./mvnw deploy -Prelease-judong -Psign-artifacts

# Deploy to Maven Central
./mvnw deploy -Prelease-central -Psign-artifacts

# Local filesystem deployment (for testing)
./mvnw deploy -Prelease-dummy
```

### Maven Profiles

| Profile | Purpose |
|---------|---------|
| `sign-artifacts` | GPG-signs built artifacts for repository deployment |
| `release-judong` | Deploys to Judong Nexus (`nexus.judo.technology`) |
| `release-central` | Deploys to Maven Central via Sonatype OSSRH |
| `release-dummy` | Deploys to local `/tmp/` directory for testing |
| `generate-github-asciidoc-diagrams` | Generates PNG diagrams from AsciiDoc using PlantUML |
| `update-source-code-license` | Applies EPL 2.0 license headers to source files |

## Key Configuration Files

| File | Purpose |
|------|---------|
| `pom.xml` | Parent POM — the main artifact; defines all dependencies, plugins, and profiles |
| `logback-test.xml` | Logging configuration used during `mvn test` (referenced via Surefire `systemPropertyVariables`) |
| `.github/workflows/build.yml` | Main CI workflow: build, deploy, tag, create GitHub release |
| `.github/workflows/release.yml` | Manual release workflow: creates PRs for master and develop |
| `.mvn/wrapper/maven-wrapper.properties` | Maven Wrapper distribution URL (Maven 3.8.6) |

## Development Environment

**Required:**
- Java 21 JDK
- Maven 3.9.4+ (or use the included `./mvnw` wrapper)
- Git with SSH access to GitHub

**Surefire JVM configuration** (automatically applied by parent POM):
```
--add-opens java.base/java.lang=ALL-UNNAMED
--add-opens java.base/java.util=ALL-UNNAMED
--add-opens java.base/java.time=ALL-UNNAMED
```

## Git Workflow

- **Main Branch:** `develop`
- **Versioning:** `${revision}` property, currently `1.0.5-SNAPSHOT`
- **Strategy:** GitFlow — `develop`, `master`, `feature/JNG-*`, `release/*`, `bugfix/JNG-*`, `hotfix/JNG-*`
- **Commit convention:** Every commit must include a JIRA ticket number (`JNG-xxx`)
- **Develop versions:** `major.minor.qualifier.YYYYMMDD_HHMMSS_commitId_branchName`
- **Release versions:** `major.minor.qualifier` (clean semantic versioning)

## Important Notes

1. This project contains **no source code** — it is purely a parent POM. Changes here affect all child projects that inherit from it.
2. Two mandatory properties must be set by child projects: `sdkPackagePrefix` (Java package for generated SDK) and `modelName` (must match the `.jsl` filename).
3. The Tatami JSL workflow plugin runs during `generate-sources` and outputs to `target/generated-sources/model/`. The `build-helper-maven-plugin` adds these as source directories.
4. Database changelogs are generated for both HSQLDB and PostgreSQL dialects (`<dialects>hsqldb,postgresql</dialects>`).
5. The `flatten-maven-plugin` resolves `${revision}` in the POM for CI-friendly versioning — the `.flattened-pom.xml` is what gets deployed.
6. Deployment targets two repositories: Judong Nexus (internal snapshots) and Maven Central (public releases via Sonatype OSSRH).

## Related Documentation

- [README.md](README.md) — Project introduction and bootstrap guide
- [CONTRIBUTING.md](CONTRIBUTING.md) — Development environment and contribution guidelines
- [.github/CIFLOW.md](.github/CIFLOW.md) — CI/CD pipeline, branching strategy, and version numbering
- [judo-community](https://github.com/BlackBeltTechnology/judo-community) — Parent ecosystem documentation
