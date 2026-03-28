# gradle-skills

A [Claude Code](https://claude.ai/code) plugin providing skills for working with Gradle projects.

## Skills

### `dependency-updates`

Guides Claude through checking, upgrading, and validating Gradle dependencies one at a time.

**Requires:** [gradle-versions-plugin](https://github.com/ben-manes/gradle-versions-plugin) in the target project for the `dependencyUpdates` task. Optionally, the [dependency-analysis-gradle-plugin](https://github.com/autonomousapps/dependency-analysis-gradle-plugin) for the `buildHealth` task.

**Works with:** any Gradle project using a [version catalog](https://docs.gradle.org/current/userguide/platforms.html) (`libs.versions.toml`).

**Workflow:**
1. Run `dependencyUpdates` to identify available upgrades
2. Update one dependency at a time in `libs.versions.toml`
3. Run `build --rerun-tasks` (and `buildHealth` if available) after each change
4. Stop and wait for maintainer confirmation before proceeding

## Installation

Install at user scope (available across all your projects):

```
claude plugin install IanBrandt/gradle-skills
```

### Bash permissions (optional)

The dependency update workflow runs `./gradlew` tasks. To avoid per-invocation approval prompts, add these permissions in `~/.claude/settings.json`:

```json
{
  "permissions": {
    "allow": [
      "Bash(./gradlew dependencyUpdates:*)",
      "Bash(./gradlew build:*)",
      "Bash(./gradlew buildHealth)"
    ]
  }
}
```

## Usage

Invoke the skill by name:

```
/gradle-skills:dependency-updates
```

## License

MIT
