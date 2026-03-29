---
name: upgrade-gradle
description: Upgrade the Gradle wrapper to the latest version, with build validation.
paths: "**/gradle-wrapper.properties"
---

# Upgrade Gradle

Upgrade the Gradle wrapper to the latest available version and validate the build.

## Prerequisites

The target project must apply this Gradle plugin:

- [gradle-versions-plugin](https://github.com/ben-manes/gradle-versions-plugin) — provides the `dependencyUpdates` task

## Workflow

### 1. Discover available Gradle version

```
./gradlew dependencyUpdates --no-parallel
```

Look for a "Gradle release-candidate updates" section (or similar) in the output that reports a newer Gradle version.

### 2. Determine upgrade path

Check the root build script (`build.gradle.kts` or `build.gradle`) for a `wrapper` task configuration. This may appear as any of:

- `tasks.wrapper { ... }`
- `tasks.named<Wrapper>("wrapper") { ... }`
- `tasks.named("wrapper") { ... }`
- Other variations that configure the `Wrapper` task

The presence or absence of this configuration determines which path to follow.

### 3. Apply the upgrade

**Path A — wrapper task exists in build script:**

1. Update the `gradleVersion` value in the wrapper task block in the build script
2. Run `./gradlew wrapper` to regenerate wrapper files
3. Run `./gradlew help` to apply the new version

**Path B — no wrapper task:**

1. Update the `distributionUrl` in `gradle/wrapper/gradle-wrapper.properties` to reference the new version
2. Run `./gradlew help` to apply the new version

The `./gradlew help` step is necessary because it causes the wrapper to download and apply the new Gradle distribution, which may update `gradle/wrapper/gradle-wrapper.jar`, `gradlew`, and `gradlew.bat`.

### 4. Validation

Run:

```
./gradlew build
```

This must pass before reporting results.

**`--rerun-tasks` option:** Before running `./gradlew build`, check memory for a saved `--rerun-tasks` preference. If no preference is found, ask the maintainer:

> Would you like to run `./gradlew build` with `--rerun-tasks`? This forces all tasks to re-execute regardless of up-to-date checks.
> - **yes** — use `--rerun-tasks` this time
> - **no** — skip it this time
> - **always** — use it now and in future sessions (saves preference)
> - **never** — skip it now and in future sessions (saves preference)

If the maintainer answers "always" or "never", save that preference to memory for future invocations. This option applies only to `./gradlew build`, not to other Gradle tasks.

### 5. Reporting

- State the previous and new Gradle versions
- List all files modified (may include `gradle-wrapper.properties`, `gradle-wrapper.jar`, `gradlew`, `gradlew.bat`, and the build script if Path A was used)
- Report validation results
- Do not create Git commits — leave changes for the maintainer to review

## Constraints

- **Scope:** Only change the Gradle version. Do not update dependencies, modify build logic, or perform unrelated refactors.
- **Git:** Do not run `git commit`, `git push`, or create branches unless explicitly instructed.
