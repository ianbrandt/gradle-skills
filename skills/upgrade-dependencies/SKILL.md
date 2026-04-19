---
name: upgrade-dependencies
description: Check for and upgrade Gradle dependencies one at a time using version catalogs, with build validation after each change.
paths: "**/libs.versions.toml"
---

# Upgrade Dependencies

Check for, upgrade, and validate Gradle dependencies one at a time.

## Prerequisites

The target project must apply these Gradle plugins:

- [gradle-versions-plugin](https://github.com/ben-manes/gradle-versions-plugin) — provides the `dependencyUpdates` task
- [dependency-analysis-gradle-plugin](https://github.com/autonomousapps/dependency-analysis-gradle-plugin) (optional) — provides the `buildHealth` task

## Workflow

### 1. Check for updates

```
./gradlew dependencyUpdates --no-parallel
```

If the output contains a "dependencies exceed the version found at the milestone revision level" section, the metadata cache may be stale. Re-run with `--refresh-dependencies`:

```
./gradlew dependencyUpdates --no-parallel --refresh-dependencies
```

Do not use `--refresh-dependencies` on the initial run — it forces re-download of all metadata.

### 2. Self-update the Gradle Versions Plugin first

If the report lists an update for the Gradle Versions Plugin itself (plugin id `com.github.ben-manes.versions`), upgrade only that plugin before any other dependency. Validate (step 4), then re-run step 1 with the new version. A newer plugin may surface different or more accurate updates, so subsequent prioritization should be based on the refreshed report.

Treat this as a normal single-dependency turn: update, validate, report, **stop** and wait for the maintainer before re-running the check.

### 3. Update one dependency at a time

Update each dependency individually so the maintainer can review and commit each change independently.

**Prioritize by compatibility relationships:**
1. Build toolchain plugins (compiler plugins, annotation processors, code generators) — update and test with the CURRENT language/platform version BEFORE upgrading the language/platform itself
2. BOM/platform dependencies before their constituent libraries
3. Core libraries before their dependents
4. Independent libraries last

**For each dependency:**
1. Update only its version in `libs.versions.toml`
2. Search the repository for usages of its catalog alias to identify affected modules
3. Run validation (step 4)
4. Report results to the maintainer
5. **STOP. Do not touch another dependency.** Your turn is over. Wait for the maintainer to explicitly say to continue before doing anything else.

> **Hard stop after each dependency — no exceptions.**
> - Do not queue up the next update.
> - Do not mention what you plan to do next.
> - Do not continue even if the user previously said "go ahead" or gave general approval.
> - Resume only when the maintainer sends a new message explicitly asking you to proceed.

**Watch for:**
- Compiler/toolchain API changes
- Breaking changes in build plugins or test frameworks
- Behavioral changes affecting existing code
- New deprecations or required source changes

**Batching:** Only batch updates when dependencies *must* be updated together (e.g., a library and its required companion version). Prefer single-dependency changes. If batching, explain why.

### 4. Validation

After each version change, run:

```
./gradlew build
```

If the project applies the dependency-analysis-gradle-plugin, also run:

```
./gradlew buildHealth
```

Both must pass before reporting results.

**`--rerun-tasks` option:** Before the first `./gradlew build` in a session, check memory for a saved `--rerun-tasks` preference. If no preference is found, ask the maintainer:

> Would you like to run `./gradlew build` with `--rerun-tasks`? This forces all tasks to re-execute regardless of up-to-date checks.
> - **yes** — use `--rerun-tasks` this time
> - **no** — skip it this time
> - **always** — use it now and in future sessions (saves preference)
> - **never** — skip it now and in future sessions (saves preference)

If the maintainer answers "always" or "never", save that preference to memory for future invocations. This option applies only to `./gradlew build`, not to other Gradle tasks.

### 5. Reporting

- Summarize what changed and why
- Report validation results
- Separate follow-up ideas from completed work
- Do not create Git commits — leave changes for the maintainer to review

## Constraints

- **Version catalog:** Do not rename catalog aliases, bundles, or plugin aliases unless explicitly asked. Maintain existing formatting and style.
- **Scope:** Keep diffs focused and minimal. Do not perform unrelated refactors or change unrelated versions. Do not introduce new dependencies without clear justification.
- **Git:** Do not run `git commit`, `git push`, or create branches unless explicitly instructed.
