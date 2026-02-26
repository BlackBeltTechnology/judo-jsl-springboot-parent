# Contributing to JUDO

This guide covers what you need to develop, build, and submit changes to the `judo-jsl-springboot-parent` project.

## Development Environment

### Required Software

| Tool | Version | Notes |
|------|---------|-------|
| Java JDK | 21+ | Source and target level |
| Maven | 3.9.4+ | Wrapper included (`./mvnw`) |

For full environment requirements, see the parent project's [CONTRIBUTING guide](https://github.com/BlackBeltTechnology/judo-community/blob/develop/CONTRIBUTING.adoc).

## Code Structure

This project is a **parent POM only** — it contains no source code. Its purpose is to define dependency versions, plugin configurations, and build lifecycle for child projects that generate JUDO JSL-based Spring Boot applications.

The key file is `pom.xml`, which defines:
- Dependency management (Spring Boot, JUDO Runtime, EMF, testing libraries)
- Plugin management (Tatami JSL workflow, Surefire, JaCoCo, Lombok delombok)
- Build profiles for deployment, artifact signing, and documentation generation

## Build Commands

```bash
# Run tests
./mvnw clean test

# Full build and install to local repository
./mvnw clean install
```

## Submitting an Issue

Before filing a new issue, search the [issue tracker](https://github.com/BlackBeltTechnology/judo-jsl-springboot-parent/issues) — your problem may already be reported or resolved.

When filing a bug, include:
- Output of `java -version` and `mvn -version`
- Your `pom.xml` or `.flattened-pom.xml` (when applicable)
- A minimal reproduction case that demonstrates the failure

A minimal reproduction helps maintainers confirm and fix bugs quickly. We will ask for one if not provided.

File new issues using the [issue form](https://github.com/BlackBeltTechnology/judo-jsl-springboot-parent/issues/new/choose).

## Submitting a Pull Request

This project follows [GitHub's standard forking model](https://guides.github.com/activities/forking/). Fork the repository, create a branch, and submit a pull request.

> **Important:** All commits must include a JIRA ticket number (e.g., `JNG-123`). There is no commit without a ticket number.
