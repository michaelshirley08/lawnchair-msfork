# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

@AGENTS.md

## Claude Code notes
- `AGENTS.md` is the single source of truth for build commands and conventions; edit it there, not here.
- Environments without an Android SDK (e.g. cloud sandboxes) can't run the Gradle build. Say so
  rather than claiming a change was verified; limit verification to reading code, `git diff`, and
  `./gradlew spotlessCheck` if the wrapper can run.
- When editing files under `lawnchair/src/`, run `./gradlew spotlessApply` if available.
