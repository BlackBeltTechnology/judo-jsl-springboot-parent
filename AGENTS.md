# judo-jsl-springboot-parent — module agent doctrine

## Module purpose

`judo-jsl-springboot-parent` is the single `<parent>` a JUDO JSL business
application declares so that a `.jsl` model file plus a Spring Boot main class
is a complete, buildable, runnable application: the POM wires the Tatami JSL
compiler into `generate-sources`, registers the generated SDK/DAO/Liquibase
output as compile sources and resources, and pins one mutually-consistent set of
Spring Boot, JUDO runtime, Tatami, EMF, Atomikos JTA, HSQLDB/PostgreSQL and
logging versions so the application POM declares almost no versions of its own.

Artifact: `hu.blackbelt.judo.jsl:judo-jsl-springboot`, packaging `pom`, version
`${revision}` (default `1.0.5-SNAPSHOT`), Java 21, built with Maven 3.9.4+ via
the bundled wrapper. It carries **no source code** — the POM is the deliverable,
so every change here propagates to every inheriting application.

License: Eclipse Public License 2.0. Repository:
`BlackBeltTechnology/judo-jsl-springboot-parent`. Part of the
[judo-community](https://github.com/BlackBeltTechnology/judo-community)
aggregator ecosystem.

### What an inheriting application gets

- **Mandatory contract.** The application POM must set two properties:
  `sdkPackagePrefix` (Java package of the generated SDK) and `modelName` (must
  match the `.jsl` filename under `src/main/resources/model/`). `modelName` is
  interpolated into the generated-source path, so a mismatch yields an empty
  source root and a silent compile of nothing.
- **Generation pipeline.** `judo-tatami-jsl-workflow-maven-plugin` runs goal
  `default-model-workflow` at `generate-sources`, reading
  `${basedir}/src/main/resources/model` and writing
  `${basedir}/target/generated-sources/model`. `build-helper-maven-plugin` then
  adds `target/generated-sources/model/sdk/${modelName}` as a source root and
  the whole `target/generated-sources/model` tree as a resource under `model/`.
- **Dual dialect schema.** `<dialects>hsqldb,postgresql</dialects>` — Liquibase
  changelogs are generated for both, so HSQLDB in-memory (dev/test) and
  PostgreSQL (production) run off the same model.
- **Runtime stack, no versions required.** JUDO runtime core (DAO core, RDBMS
  DAO + hsqldb/postgresql variants, dispatcher, access manager, Spring
  integration), `judo-dao-api`, `judo-sdk-common`, the BlackBelt mapper api/impl,
  `judo-tatami-asm2rdbms`, the Liquibase meta-model, Spring Boot starter, data-jdbc
  starter, logging starter and test starter, Atomikos JTA (`transactions-jta`,
  `transactions-jdbc` 4.0.6, `transactions-spring-boot3-starter` 6.0.0), EMF
  (Ecore 2.38.0, Common 2.41.0, XMI 2.16.0), Lombok 1.18.34, Guava 30.0-jre,
  Gson 2.9.0, HSQLDB 2.6.1, and the SLF4J 2.0.16 / Logback 1.5.12 stack with the
  `jul-to-slf4j`, `sysout-over-slf4j` and `liquibase-slf4j` bridges.
- **Test stack.** JUnit Jupiter 5.8.2 (api/engine/params), Mockito 4.8.0 (+
  jupiter integration), Hamcrest 2.2, `jcabi-log`, and Spring Boot test starter
  (with `org.ow2.asm:asm` excluded and re-pinned to 9.8 at test scope).
- **Surefire JVM configuration**, applied automatically:
  ```
  -Dfile.encoding=UTF-8
  --add-opens java.base/java.lang=ALL-UNNAMED
  --add-opens java.base/java.util=ALL-UNNAMED
  --add-opens java.base/java.time=ALL-UNNAMED
  ```
  plus `trimStackTrace=false` and `logback.configuration` pointed at
  `${maven.multiModuleProjectDirectory}/logback-test.xml`.
- **Coverage and CI-friendly versioning.** JaCoCo 0.8.12 (`prepare-agent` +
  `report`, agent property `jacoco.agent`) and `flatten-maven-plugin` 1.3.0 in
  `resolveCiFriendliesOnly` mode run in the parent's own build, so `${revision}`
  is resolved into the deployed POM (`.flattened-pom.xml`).

A typical inheriting application therefore contains only: the model file at
`src/main/resources/model/<modelName>.jsl`, a
`src/main/java/.../<ModelName>SpringApplication.java` entry point, DataSource +
Liquibase settings in `src/main/resources/application.properties`, and
integration tests in `src/test/java/.../<ModelName>SpringApplicationTests.java`
that consume the DAOs generated under `target/generated-sources/model/`.

## Reactor map

`pom.xml` declares **no `<modules>`** — this is a single, non-aggregator parent
POM (`<packaging>pom</packaging>`). There is nothing to descend into; the whole
contribution of this repository is what inheritors receive.

<modules>
  (none — single parent/BOM POM, no reactor children)
</modules>

### Managed dependency sets (imported BOMs)

| Imported BOM | Governs |
|---|---|
| judo-runtime-core-dependencies (`${judo-runtime-core-version}`) | Every JUDO runtime, DAO, dispatcher, access-manager, mapper and SDK artifact — this is why those dependencies are declared without a `<version>`. |
| spring-boot-dependencies 3.5.0 | Spring Boot starters and the Spring ecosystem versions. |
| eclipse-platform-dependencies 4.22 (`fr.jmini.ecentral`) | Eclipse platform artifacts pulled in by EMF/Epsilon; resolved from the extra `ecentral` plugin repository. |

Additionally version-managed outside a BOM: `antlr-runtime` 3.2, and the Tatami
JSL artifacts `judo-tatami-jsl-jsl2psm` and `judo-tatami-jsl-workflow` at
`${judo-tatami-jsl-version}`.

### Managed plugins (`<pluginManagement>` — versions/config only, not bound here)

| Plugin | Managed contribution |
|---|---|
| judo-tatami-jsl-workflow-maven-plugin | JSL → SDK/DAO/Liquibase generation at `generate-sources`; sources/dialects/destination preconfigured. |
| build-helper-maven-plugin 3.3.0 | `add-source` for the generated SDK, `add-resource` for the generated model tree. |
| maven-surefire-plugin (`${surefire-version}` = 3.5.1) | JVM `--add-opens` flags, UTF-8, logback test config, full stack traces. |
| maven-compiler-plugin 3.10.1 | Compiler plugin version pin (source/target 21 from properties). |
| maven-install-plugin 2.5.2 / maven-deploy-plugin 3.0.0 | Install/deploy plugin version pins. |
| maven-javadoc-plugin 3.4.1 | `attach-javadocs`; `doclint` off, `failOnError` false, custom EMF tags (`@model`, `@generated`, `@ordered`, `@param`) so generated sources do not fail the build. |
| lombok-maven-plugin 1.18.20.0 | `delombok` at `generate-sources` into `target/delombok`. |
| jacoco-maven-plugin (`${jacoco.version}`) / sonar-maven-plugin (`${sonar-maven-plugin-version}` = 3.9.1.2184) | Coverage and static-analysis version pins. |
| lifecycle-mapping 1.0.0 (m2e) | Eclipse-only: tells m2e to ignore the `flatten` and `default-model-workflow` executions. No effect on the Maven build. |

### Key properties

| Property | Meaning |
|---|---|
| `revision` | Module version, currently `1.0.5-SNAPSHOT`; resolved into the deployed POM by the flatten plugin. |
| `judo-runtime-core-version` | JUDO runtime BOM coordinate (`1.0.6-SNAPSHOT`). |
| `judo-tatami-jsl-version` | JSL compiler/workflow plugin + artifacts (`1.1.4-SNAPSHOT`). |
| `judo-tatami-core-version` | Tatami core transformation engine (`1.1.4-SNAPSHOT`). |
| `maven` / `maven.version` | Wrapper distribution expectation (3.9.4) and minimum enforced Maven (3.8.3). |
| `maven.compiler.source` / `.target` | Java 21 for the parent and every inheritor. |
| `slf4j-version`, `logback-version`, `lombok-version` | 2.0.16 / 1.5.12 / 1.18.34. |
| `logback-test-config` | `${maven.multiModuleProjectDirectory}/logback-test.xml`, handed to Surefire as `logback.configuration`; this is the logging configuration used during `mvn test`. |
| `project-repositoryId` | `BlackBeltTechnology/judo-jsl-springboot-parent`; feeds `<url>`, `<scm>` and `<issueManagement>`. |
| `sonar.*`, `jacoco.version` | Code-quality wiring (JaCoCo 0.8.12, Java language, jacoco coverage plugin). |

### Profiles — trigger and effect

All profiles below are activated explicitly with `-P<id>`; none is active by
default.

| Profile | Trigger → effect |
|---|---|
| `sign-artifacts` | `-Psign-artifacts` → GPG-signs artifacts via `sign-maven-plugin` 1.1.0 before deployment. |
| `release-judong` | `-Prelease-judong` → distributionManagement points at Judong Nexus (`https://nexus.judo.technology/repository/maven-judong-snapshots/`) for both snapshots and releases. |
| `release-central` | `-Prelease-central` → `nexus-staging-maven-plugin` against OSSRH (`https://oss.sonatype.org/`), `autoReleaseAfterClose=true`, 15-minute staging timeout. |
| `release-dummy` | `-Prelease-dummy` → deploys to `file:///tmp/${project.groupId}-${project.artifactId}-${project.version}/...` for dry runs. |
| `generate-github-asciidoc-diagrams` | `-Pgenerate-github-asciidoc-diagrams` → `asciidoctor-maven-plugin` 2.2.2 (JRuby 9.3.4.0, AsciidoctorJ 2.5.6 + diagram 2.2.3, PlantUML, ditaa) renders `./.github` AsciiDoc at `generate-resources`, then `maven-resources-plugin` copies the generated PNGs from `target/generated-docs/images/` back into `.github/`. |
| `update-source-code-license` | `-Pupdate-source-code-license` → `license-maven-plugin` 2.0.0 applies EPL 2.0 headers (`update-file-header`, excluding `**/*.json`) and refreshes the project license file at `process-sources`; organization "BlackBelt Technology", inception year 2018. |

## Build commands

Use the bundled Maven wrapper (`./mvnw`), never a repo-root `mvn`.

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

## Repository layout

```
judo-jsl-springboot-parent/
├── pom.xml                  # The parent POM — the core (and only) artifact
├── .github/
│   ├── workflows/           # 10 GitHub Actions CI/CD workflow files
│   ├── CIFLOW.md            # CI/CD branching and versioning documentation
│   └── ISSUE_TEMPLATE/      # GitHub issue templates
├── .mvn/wrapper/            # Maven wrapper configuration (distribution URL)
├── logback-test.xml         # Logback configuration used during test execution
├── README.md                # Project documentation and bootstrap guide
├── CONTRIBUTING.md          # Contribution guidelines
├── LICENSE.txt              # EPL 2.0 license text
└── .gitignore
```

CI entry points: `.github/workflows/build.yml` is the main workflow (build,
deploy, tag, create GitHub release); `.github/workflows/release.yml` is the
manual release workflow that opens PRs against `master` and `develop`.

## Development environment

**Required:** Java 21 JDK; Maven 3.9.4+ (or the included `./mvnw`); Git with SSH
access to GitHub.

## Git workflow

- **Main branch:** `develop`.
- **Versioning:** the `${revision}` property, currently `1.0.5-SNAPSHOT`.
- **Strategy:** GitFlow — `develop`, `master`, `feature/JNG-*`, `release/*`,
  `bugfix/JNG-*`, `hotfix/JNG-*`.
- **Commit convention:** every commit must include a JIRA ticket (`JNG-xxx`).
- **Develop versions:** `major.minor.qualifier.YYYYMMDD_HHMMSS_commitId_branchName`.
- **Release versions:** `major.minor.qualifier` (clean semantic versioning).
- Deployment targets two repositories: Judong Nexus (internal snapshots) and
  Maven Central (public releases via Sonatype OSSRH).

## Code instructions

1. First think through the problem, read the codebase for relevant files.
2. Before you make any major changes, check in with me and I will verify the plan.
3. Please every step of the way just give me a high level explanation of what changes you made.
4. Make every task and code change you do as simple as possible. We want to avoid making any massive or complex changes. Every change should impact as little code as possible. Everything is about simplicity.
5. Maintain a documentation file that describes how the architecture of the app works inside and out.
6. Never speculate about code you have not opened. If the user references a specific file, you MUST read the file before answering. Make sure to investigate and read relevant files BEFORE answering questions about the codebase. Never make any claims about code before investigating unless you are certain of the correct answer - give grounded and hallucination-free answers.
7. For implementation use TDD (Test-Driven Development): write or update tests first to define the expected behaviour, verify they fail, then write the minimal implementation to make them pass.
8. Use DRY (Don't Repeat Yourself): extract reusable logic into separate classes, utilities, or components. If the same pattern appears in multiple places, refactor it into a shared helper.

## Related documentation

- [README.md](README.md) — project introduction and bootstrap guide
- [CONTRIBUTING.md](CONTRIBUTING.md) — development environment and contribution guidelines
- [.github/CIFLOW.md](.github/CIFLOW.md) — CI/CD pipeline, branching strategy, version numbering
- [judo-community](https://github.com/BlackBeltTechnology/judo-community) — parent ecosystem documentation

<!-- dox-doctrine -->
## Documentation Update Protocol (WRITE discipline)

Per-directory `AGENTS.md` files form a tree. Each directory `AGENTS.md` is the
per-file record for the files in that directory. This module-root `AGENTS.md`
holds doctrine + architecture pointers only — never a per-file index.

**Keep the root lean.** This file loads into every agent turn — every byte costs
tokens on every turn. A verbose root file buries the rules the model must follow
(signal dilution) and measurably degrades adherence; a lean file keeps doctrine
salient. Default assumption: your update does NOT belong in the root — route it
by the table below.

**Route every doc update by kind:**

| Kind of update | Goes in |
|---|---|
| New file in a directory, or its per-file detail / change history | Nearest directory `AGENTS.md`. Add a `` | `<basename>` | <purpose> | `` row, path-alphabetical. |
| Data flow, protocol, architecture rationale | `docs/architecture.md` or a `docs/<topic>.md` |
| End-user / developer setup | `README.md` |
| Cross-cutting rule every agent needs every turn (rare) | this module-root `AGENTS.md` |

**Read before editing (chain walk).** Before editing a file, read the nearest
`AGENTS.md` chain root→leaf so you know the file's recorded purpose, contracts,
and change history. Do not edit blind.

**Update after editing (closeout pass).** After changing a file, update its row
in the nearest directory `AGENTS.md`: find the file's row, update its purpose in
place; if absent, add it in path-alphabetical order. New directory → scaffold
its `AGENTS.md`. One row per file. The purpose carries a one-line summary, key
exported symbols, contracts/invariants, and `See change: <id>` history.

**Row style (caveman).** Short declarative fragments. Drop articles. Subject →
verb → object, present tense. One fact per row. Prefer concrete tokens (paths,
symbols, env vars) over prose. Keep identifiers verbatim.

**Size rule — split an over-large directory `AGENTS.md` file-based.** pi
auto-injects a directory `AGENTS.md` on every turn when cwd sits at/below it, so
an over-large directory `AGENTS.md` is not supported. Split it file-based: a row
exceeding the length threshold promotes to a per-file `<File>.AGENTS.md`
sidecar carrying that file's full detail (including every `See change:`). The
sidecar is pull-only — its name is not `AGENTS.md`, so pi never auto-injects it
— yet it stays search-indexed (`agents` doc_type). The directory `AGENTS.md`
keeps a one-line summary plus a `→ see `<File>.AGENTS.md`` pointer. Rows within
the threshold stay verbatim (lossless).

## Finding docs (READ discipline)

`kb_*` tools are faster and cheaper than raw search — they return a one-line
purpose + key exports per file, not raw bytes. **This fires on the ACTION, not
the intent** — before you `grep`/`rg` for a symbol, `cat`/read a file to learn
what it does, or chase an import, the kb call goes first. It fires **even
mid-task when you already know the file**; knowing the file does not exempt you.
When your reflex is the left column, run the right column instead:

| You're about to… | Do this FIRST instead |
|---|---|
| `grep -rn "SymbolName" src/` — find where a fn / type / const lives | `kb_search --doc-type agents "SymbolName"` — tree indexes key exports per file |
| `grep -rn "feature\|topic" src/` — how does X work / where's X handled | `kb_search "feature topic"` |
| `cat` / read a file just to learn its purpose before editing | `kb agents <path>` — one-line purpose + exports + change history |
| chase imports / callers across files | `kb_neighbors <path\|heading>` |
| read one doc section in full | `kb_get <path> <section>` |

**Fall-through (explicit):** if the kb call returns nothing relevant, `rg` /
source read is allowed — then add the missing directory `AGENTS.md` row per the
WRITE discipline. kb does NOT replace grep; it goes first.

## Scope guard

This module has no source tree — `pom.xml` *is* the product. Any version bump,
dependency addition or plugin-config change here silently changes the build of
every inheriting JUDO JSL application. Treat an edit as a release action:
confirm the coordinate exists in the imported BOMs before pinning it by hand,
and prefer moving a version into the BOM upstream over declaring it here.
