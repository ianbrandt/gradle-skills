---
name: dependency-updates
description: Check for and upgrade Gradle dependencies one at a time using version catalogs, with build validation after each change.
paths: "**/libs.versions.toml"
---

# Dependency Updates

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

### 2. Update one dependency at a time

Update each dependency individually so the maintainer can review and commit each change independently.

**Prioritize by compatibility relationships:**
1. Build toolchain plugins (compiler plugins, annotation processors, code generators) — update and test with the CURRENT language/platform version BEFORE upgrading the language/platform itself
2. BOM/platform dependencies before their constituent libraries
3. Core libraries before their dependents
4. Independent libraries last

**For each dependency:**
1. Update only its version in `libs.versions.toml`
2. Search the repository for usages of its catalog alias to identify affected modules
3. Run validation (step 3)
4. Report results to the maintainer

> **Stop after each dependency.** Wait for explicit maintainer confirmation before proceeding. Do not continue autonomously even if the user previously gave general approval.

**Watch for:**
- Compiler/toolchain API changes
- Breaking changes in build plugins or test frameworks
- Behavioral changes affecting existing code
- New deprecations or required source changes

**Batching:** Only batch updates when dependencies *must* be updated together (e.g., a library and its required companion version). Prefer single-dependency changes. If batching, explain why.

### 3. Validation

After each version change, run:

```
./gradlew build --rerun-tasks
```

If the project applies the dependency-analysis-gradle-plugin, also run:

```
./gradlew buildHealth
```

Both must pass before reporting results.

### 4. Reporting

- Summarize what changed and why
- Report validation results
- Separate follow-up ideas from completed work
- Do not create Git commits — leave changes for the maintainer to review

## Constraints

- **Version catalog:** Do not rename catalog aliases, bundles, or plugin aliases unless explicitly asked. Maintain existing formatting and style.
- **Scope:** Keep diffs focused and minimal. Do not perform unrelated refactors or change unrelated versions. Do not introduce new dependencies without clear justification.
- **Git:** Do not run `git commit`, `git push`, or create branches unless explicitly instructed.
