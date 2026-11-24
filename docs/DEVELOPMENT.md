# Development Guide

Official guides: <https://docs.lawnchair.app/developers>. This page summarises what is
specific to this checkout. Commands below were derived from the Gradle files; the full
Android build was **not** run when this was written (no SDK in the authoring environment),
so adjust to your setup if something differs.

## 1. Prerequisites
* JDK matching the AGP 9.x requirement (17+; check `gradle/libs.versions.toml` / wrapper).
* Android SDK with the `compileSdk` declared in `build.gradle` (targetSdk 37).
* Git submodule initialised:
  ```bash
  git submodule update --init --recursive
  ```
  Without it `:iconloaderlib`, `:searchuilib`, `:animationlib`, … are empty and Gradle sync fails.

## 2. Build & install
```bash
./gradlew assembleLawnWithQuickstepGithubDebug      # debug APK, GitHub channel
./gradlew assembleLawnWithQuickstepPlayDebug        # Play channel (default flavor)
./gradlew installLawnWithQuickstepGithubDebug       # install on connected device
```
Output: `build/outputs/apk/.../Lawnchair.<version>.<channel>.<buildType>.apk`.

Formatting / lint:
```bash
./gradlew spotlessCheck     # or spotlessApply to fix
./gradlew lintLawnWithQuickstepGithubDebug
```
(`lint-baseline.xml` records accepted existing findings.)

Set Lawnchair as the home app, then (for gesture navigation / Recents) it needs to be the
system's recents provider — on stock Android this requires the QuickSwitch module/root
(Android 15–16) per the README.

## 3. Where to put changes

| You want to… | Put it in… |
|---|---|
| Add a new Lawnchair feature | new package under `lawnchair/src/app/lawnchair/` |
| Add a setting | entry in `preferences2/PreferenceManager2.kt` + screen/row in `ui/preferences/destinations/` + strings in `lawnchair/res/values/strings.xml` (translations are handled via Crowdin — don't edit non-English `values-*` by hand) |
| Add a gesture action | new handler in `gestures/handlers/` + register in `GestureHandlerConfig`/options |
| Add a web search provider (search bar) | `qsb/providers/<Name>.kt` (+ register in the provider list) |
| Add a web search provider (all-apps) | `search/algorithms/engine/provider/web/BuiltInWebSearchProviders.kt` |
| Add an app category | flowerpot rule file (see `flowerpot/README.md`) |
| Change AOSP behaviour | prefer subclass/override via `override/` and `factory/`; edit `src/` / `quickstep/` only when necessary and mark with a `Lawnchair` comment |
| Add an Android-version shim | `compatLib/compatLibV*` |

## 4. Coding standards
* Kotlin first for new code; Compose for new settings UI.
* Run `spotlessApply` before committing; PR titles follow Conventional Commits as seen in
  history (`fix:`, `feat(scope):`, `chore(deps):`).
* Strings: English only in `values/`; Crowdin syncs the rest.
* Avoid reading `PreferenceManager2` flows on the main thread except via `firstCached()` for
  cached values.

## 5. CI
`.github/workflows/`:
| Workflow | Purpose |
|---|---|
| `ci.yml` | Debug build on PRs/pushes |
| `release_update.yml`, `build_release_apk.yml` | Release APKs |
| `crowdin*.yml` | Translation upload/download |
| `close_stale_issues.yml`, `close_low_effort_issues.yml` | Issue hygiene |

## 6. Existing automated tests
The `tests/` directory is **AOSP Launcher3's** suite (Robolectric "multivalent" tests,
device tests, TAPL — the Test Automation Platform for Launcher). Layout:

| Path | Kind |
|---|---|
| `tests/multivalentTests/src/**` | Run on JVM (Robolectric) *and* device: grid/`DeviceProfile`, prefs, model, folders, popups, widgets… |
| `tests/multivalentTestsForDevice/src/**` | Device-only multivalent tests |
| `tests/src_deviceless/**` | Pure JVM tests |
| `tests/src/com/android/launcher3/**` | Instrumentation tests: workspace, folder, dragging, allapps, widget, backup/restore, `ui/TaplTestsLauncher3Test` |
| `tests/tapl/**` | TAPL driver library |
| `tests/assets/**` | Fixture data (reorder algorithm, widgets, DB dumps) |

They are wired for the AOSP build (`Android.bp`); whether each runs under this Gradle
project has not been verified. There are **no tests for the `app.lawnchair.*` package**.

## 7. Recommended automation roadmap
Prioritised by value / ease; each maps to cases in `docs/testing/REGRESSION_TEST_PLAN.md`:

1. **JVM unit tests (no device)** — quickest wins
   * `HotseatMode.fromString`, `SmartspaceMode.fromString`, `GestureType.fromKey`
   * `Flowerpot` parser (`FlowerpotReader`, `LineParser`, rules) — pure text → rules (TC-FPT-*)
   * `LawnchairBackup` zip round-trip with temp files (TC-BKP-*)
   * Search calculator (`search/algorithms/data/calculator`) (TC-SRCH-CALC-*)
   * `LawnchairSearchAlgorithm.create` selection logic with fake prefs
2. **Robolectric / Room tests** — `FolderDao`/`FolderService`, `IconOverrideRepository`,
   `PreferenceManager2` defaults + migration (`SharedPreferencesMigration`)
3. **Instrumented smoke (UI Automator / TAPL)** — launch home, open drawer, search, open
   settings, create folder, add widget, reboot persistence.
4. **Screenshot tests** for the settings screens (Compose) and icon shapes.

Add test dependencies in `build.gradle` (`testImplementation`/`androidTestImplementation`)
and create `lawnchair/test/` / `lawnchair/androidTest/` source sets — none exist yet.
