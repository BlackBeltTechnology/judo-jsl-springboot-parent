# parent-pom Specification

## Purpose

Provides a Maven parent POM that child projects inherit to get a complete, pre-configured build environment for JUDO JSL-based Spring Boot applications — including dependency management, code generation pipeline, database support, and CI-friendly versioning.

## Architecture

The parent POM (`pom.xml`, packaging: `pom`) defines three layers of configuration:

- **Dependency management** — BOM imports (`spring-boot-dependencies`, `judo-runtime-core-dependencies`, `eclipse-platform-dependencies`) and direct dependency declarations with locked versions
- **Plugin management** — pre-configured plugins: `judo-tatami-jsl-workflow-maven-plugin` (code generation), `build-helper-maven-plugin` (source directory registration), `maven-surefire-plugin` (test execution with JVM flags), `jacoco-maven-plugin` (coverage), `flatten-maven-plugin` (CI-friendly POM)
- **Build profiles** — `sign-artifacts`, `release-judong`, `release-central`, `release-dummy`, `generate-github-asciidoc-diagrams`, `update-source-code-license`

Child projects declare `sdkPackagePrefix` and `modelName` properties, include the Tatami and build-helper plugins, and place a `.jsl` model file at `src/main/resources/model/<modelName>.jsl`.

## Requirements

### Requirement: Parent POM SHALL provide all dependencies needed for a JUDO JSL Spring Boot application

Child projects should not need to declare any additional dependencies for basic JUDO functionality.

#### Scenario: Child project inherits complete dependency set
- **GIVEN** a child project with `<parent>` referencing `hu.blackbelt.judo.jsl:judo-jsl-springboot`
- **WHEN** the child project runs `mvn dependency:tree`
- **THEN** the dependency tree includes Spring Boot starters, JUDO runtime core modules, EMF, Liquibase, HSQLDB, and Lombok without explicit declarations in the child POM

### Requirement: Code generation pipeline SHALL transform JSL models into Java SDK classes and Liquibase changelogs

#### Scenario: JSL model compilation during generate-sources
- **GIVEN** a child project with a valid `.jsl` file at `src/main/resources/model/<modelName>.jsl`
- **AND** `sdkPackagePrefix` and `modelName` properties are set in the child POM
- **WHEN** `mvn generate-sources` is executed
- **THEN** Java SDK classes are generated to `target/generated-sources/model/sdk/<modelName>/`
- **AND** Liquibase changelogs are generated for both HSQLDB and PostgreSQL dialects

#### Scenario: Generated sources are added to compilation
- **GIVEN** the `build-helper-maven-plugin` is declared in the child project's build plugins
- **WHEN** `mvn compile` is executed
- **THEN** `target/generated-sources/model/sdk/<modelName>/` is included as a source directory
- **AND** `target/generated-sources/model/` is included as a resource directory under `model/`

### Requirement: CI-friendly versioning SHALL use the `${revision}` property

#### Scenario: POM flattening for deployment
- **GIVEN** the parent POM uses `<version>${revision}</version>`
- **WHEN** `mvn install` is executed
- **THEN** the `flatten-maven-plugin` produces a `.flattened-pom.xml` with `${revision}` resolved to the actual version string

### Requirement: Test execution SHALL configure JVM for Java 21 module access

#### Scenario: Surefire runs with add-opens flags
- **WHEN** `mvn test` is executed in a child project
- **THEN** the Surefire plugin passes `--add-opens java.base/java.lang=ALL-UNNAMED`, `--add-opens java.base/java.util=ALL-UNNAMED`, and `--add-opens java.base/java.time=ALL-UNNAMED` to the test JVM
- **AND** file encoding is set to UTF-8

### Requirement: Dual database dialect support SHALL be pre-configured

#### Scenario: HSQLDB for development and testing
- **GIVEN** a child project with `application.properties` configured for HSQLDB
- **WHEN** the Spring Boot application starts
- **THEN** the HSQLDB in-memory database is used via `judo-runtime-core-spring-hsqldb` and `judo-runtime-core-dao-rdbms-hsqldb`

#### Scenario: PostgreSQL for production
- **GIVEN** a child project with `application.properties` configured for PostgreSQL
- **WHEN** the Spring Boot application starts
- **THEN** PostgreSQL is used via `judo-runtime-core-spring-postgresql` and `judo-runtime-core-dao-rdbms-postgresql`

### Requirement: Deployment profiles SHALL target correct repositories

#### Scenario: Judong Nexus deployment
- **GIVEN** the `release-judong` profile is active
- **WHEN** `mvn deploy` is executed
- **THEN** artifacts are deployed to `https://nexus.judo.technology/repository/maven-judong-snapshots/`

#### Scenario: Maven Central deployment
- **GIVEN** the `release-central` profile is active
- **WHEN** `mvn deploy` is executed
- **THEN** artifacts are staged to Sonatype OSSRH at `https://oss.sonatype.org/` with auto-release after close
