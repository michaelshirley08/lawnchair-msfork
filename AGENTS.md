# AGENTS.md

Lawnchair 16: an Android launcher built on AOSP Launcher3 + Quickstep (Android 16).
Single Gradle app module; Lawnchair's own code is in `lawnchair/src/app/lawnchair/`
(Kotlin, Jetpack Compose). Everything else (`src/`, `quickstep/`, `systemUI/`, `wmshell/`)
is mostly upstream AOSP.

More detail: `docs/ARCHITECTURE.md`, `docs/DEVELOPMENT.md`,
`docs/testing/REGRESSION_TEST_PLAN.md`.

## Setup
- JDK 21 (CI uses Zulu 21). Android SDK with compileSdk 37 / build-tools 37.0.0.
- The `platform_frameworks_libs_systemui` git submodule must be initialised, or the build fails:
  `git submodule update --init --recursive`

## Commands
Variants are `lawn` + `WithQuickstep` + channel (`Github`, `Nightly`, `Play`) + `Debug`/`Release`.
```bash
./gradlew assembleLawnWithQuickstepGithubDebug   # build debug APK (CI also builds PlayDebug and NightlyRelease)
./gradlew installLawnWithQuickstepGithubDebug    # install on a connected device
./gradlew spotlessCheck                          # formatting check, run by CI
./gradlew spotlessApply                          # fix formatting
./gradlew lintLawnWithQuickstepGithubDebug       # Android lint (accepted findings in lint-baseline.xml)
```
Output APK: `build/outputs/apk/**/Lawnchair.<version>.<channel>.<buildType>.apk`.
There is no working single-test command yet (see Testing).

## Architecture
- Launcher3, Quickstep and Lawnchair are **source sets of one module**, not separate Gradle
  modules (root `build.gradle`, `sourceSets` block): `main` = `src`, `src_plugins`, `compose/*`;
  `lawn` = `lawnchair/src`, `src_flags`, `src_shortcuts_overrides`, `tests/shared`;
  `withQuickstep` = `quickstep/src`, `quickstep/dagger`, `quickstep/recents_ui_overrides/src`.
  Flavor dimensions are `app` × `recents` × `channel`; package IDs `app.lawnchair`,
  `app.lawnchair.nightly`, `app.lawnchair.play`.
- Entry points: `LawnchairApp` (Application, init of prefs/root helper/`LawnchairProcessInitializer`)
  and `LawnchairLauncher extends QuickstepLauncher`, which wires Lawnchair gestures, popups,
  theming, feed overlay and sleep. Settings UI is Compose, hosted by `ui/preferences/PreferenceActivity`.
- Two preference layers: legacy `PreferenceManager` (SharedPreferences, still read by View code)
  and `PreferenceManager2` (DataStore + Flow; use `firstCached()` for synchronous reads, never
  block the main thread on the flow). `IdpPreference` grid values feed `InvariantDeviceProfile`;
  changing them goes through `ReloadHelper` / `DeviceProfileOverrides` to reload the launcher.
- Two databases: AOSP `launcher.db` (workspace model, loaded via `LauncherModel` /
  `Executors.MODEL_EXECUTOR`) and Lawnchair's Room `AppDatabase` (folders/Lawndeck, icon overrides,
  wallpaper). Backup (`backup/LawnchairBackup`) zips both plus prefs.
- All-apps search: `LawnchairSearchAlgorithm` picks ASI, Local or App-only from the
  `searchAlgorithm` pref; providers live in `search/algorithms/engine/provider`.
  Search-bar (QSB) web providers are separate, in `qsb/providers`.

## Where changes go
- New feature: a package under `lawnchair/src/app/lawnchair/`.
- New setting: entry in `preferences2/PreferenceManager2.kt` (DataStore), UI in
  `ui/preferences/destinations/`, English string in `lawnchair/res/values/strings.xml`.
  `preferences/PreferenceManager.kt` is the legacy SharedPreferences layer; don't add new prefs there.
- Prefer subclassing/overrides (`override/`, `factory/`) over editing AOSP code. If you must edit
  `src/`, `quickstep/` or `build.gradle` internals, keep the diff minimal and mark it with a
  `Lawnchair` comment, since these files conflict most on upstream merges.
- Android-version differences go in `compatLib/compatLibV*`.

## Rules
- Kotlin for new code; Compose for new settings UI.
- Spotless scope: Java everywhere (google-java-format AOSP style); Kotlin **only** under
  `lawnchair/src/**` (ktlint + compose rules). Run `spotlessApply` before committing.
- Don't hand-edit translated `values-*` resources; Crowdin manages them.
- Don't edit `Android.bp`, `CleanSpec.mk`, `OWNERS`, `PREUPLOAD.cfg`: AOSP leftovers, unused by Gradle.
- Commit/PR titles use Conventional Commits, as in history: `fix:`, `feat(scope):`, `chore(deps):`.
- Never commit `keystore.properties`, `*.jks`, or other signing material.
- Dependency bumps are done by Renovate; don't bump versions in `gradle/libs.versions.toml` unless asked.

## Testing
- No tests exist for `app.lawnchair.*`. `tests/` holds upstream AOSP Launcher3 tests, which are
  wired for the AOSP build (`Android.bp`); it hasn't been verified that they run under Gradle here.
- Behaviour changes: check the relevant cases in `docs/testing/REGRESSION_TEST_PLAN.md` and add a
  case for every bug fixed.
- Pure logic (flowerpot parser, backup zip, search calculator, mode `fromString`) is the easiest
  to unit-test; see `docs/DEVELOPMENT.md` §7.

## Gotchas
- Gradle sync/build can't work without the submodule (see Setup).
- Some features are gated at runtime: ASI search (API 31+ and Quickstep enabled), Google hotseat /
  smartspace (need the Google app), root-only paths (QuickSwitch, root sleep), and the Play flavor
  restricts some features. Handle the "unavailable" case without crashing.
- CI runs on pushes to `*-dev` branches and on PRs; it ignores markdown-only changes.
