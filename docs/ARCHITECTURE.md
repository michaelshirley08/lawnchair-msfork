# Lawnchair 16 — Architecture Overview

> Scope: this document describes the structure of this repository (a fork of
> `LawnchairLauncher/lawnchair`, branch for Lawnchair 16 / Launcher3 from Android 16).
> It was written from a read-through of the source tree, `build.gradle` and
> `settings.gradle`. Statements are about the code as checked in; runtime behaviour
> was **not** exercised (no device/emulator was available when writing).

## 1. What this project is

Lawnchair is an Android home-screen app (launcher). It starts from AOSP **Launcher3**
(plus **Quickstep**, the Recents / gesture-navigation layer) and adds Pixel Launcher
features and a large set of customization options.

Design principle: **keep AOSP code as close to upstream as possible and put Lawnchair
code in its own source tree**, so that new Android releases can be merged with few conflicts.

## 2. Repository layout

| Path | Role | Origin |
|---|---|---|
| `src/`, `res/`, `src_plugins/`, `protos/` | Core Launcher3: workspace, all-apps, folders, widgets, drag & drop, `DeviceProfile`, model/DB | AOSP |
| `quickstep/` | Recents, overview, swipe-up-to-home, task views, taskbar | AOSP |
| `systemUI/` (`shared`, `plugin`, `plugin_core`, `common`, `log`, `animation`, `unfold`, `utils`) | SystemUI libraries Launcher/Quickstep depend on | AOSP |
| `wmshell/` | Window Manager Shell (split-screen, PiP, bubbles…) | AOSP |
| `platform_frameworks_libs_systemui/` | **Git submodule** (`LawnchairLauncher/platform_frameworks_libs_systemui`, branch `16-dev`): `iconloaderlib`, `searchuilib`, `animationlib`, `msdllib`, `viewcapturelib`, `displaylib`, `mechanics`, `contextualeducationlib` | AOSP (forked) |
| `compatLib/` | One module per Android version: `compatLibVQ` … `compatLibVV`, `compatLibVBaklava` (API shims) | Lawnchair |
| `hidden-api/` | Stubs for hidden framework APIs | Lawnchair |
| `lawnchair/` | **Lawnchair's own code** — `src/app/lawnchair/**` (Kotlin + Compose), `res/`, `aidl/`, `assets/`, `protos/` | Lawnchair |
| `modules/widgetpicker` | Widget picker module | Lawnchair/AOSP |
| `flowerpot/` | Python tooling + format spec for app-category rule files (Play Store scraping, merge) | Lawnchair |
| `flags/`, `aconfig/`, `dagger/`, `concurrent/`, `androidx-lib/` | Feature flags, DI, threading, androidx forks | mixed |
| `baseline-profile/` | Baseline/startup profile generation | Lawnchair |
| `tests/` | **AOSP Launcher3 tests** (TAPL, multivalent, deviceless). No Lawnchair-specific tests exist yet — see `docs/testing/` | AOSP |
| `fastlane/`, `github/`, `.github/workflows/` | Store metadata, CI | Lawnchair |
| `Android.bp`, `CleanSpec.mk`, `OWNERS`, `PREUPLOAD.cfg` | AOSP build leftovers — **not used** by Gradle | AOSP |

`:compose` is commented out in `settings.gradle`; Compose "facade/features" source
directories are instead attached to the main module (see below).

## 3. Build structure

* **Single Gradle app module** (root project `lawnchair`). Lawnchair, Launcher3 and
  Quickstep are combined as *source sets* in the root `build.gradle` (≈ lines 290–341),
  not as separate Gradle modules:
  * `main` java srcDirs: `src`, `src_plugins`, `compose/facade/core`,
    `compose/facade/enabled`, `compose/features`
  * Lawnchair srcDirs: `lawnchair/src`, `src_flags`, `src_shortcuts_overrides`, `tests/shared`
    (+ `lawnchair/aidl`, `lawnchair/res`, `lawnchair/assets`, `lawnchair/protos`)
  * `withQuickstep` srcDirs: `quickstep/src`, `quickstep/dagger`,
    `quickstep/recents_ui_overrides/src`, `quickstep/src_protolog`
* **Flavor dimensions:** `app` × `recents` × `channel`

  | Dimension | Values | Notes |
  |---|---|---|
  | app | `lawn` | only value |
  | recents | `withQuickstep` | `minSdk 26` |
  | channel | `github` (`app.lawnchair`), `nightly` (`app.lawnchair.nightly`), `play` (`app.lawnchair.play`, default) | Play build restricts some features |

  Example variant: `lawnWithQuickstepGithubDebug`.
* `minSdk 26`, `targetSdk 37`; `versionCode 16_00_02_01_00` at the time of writing.
* APK name: `Lawnchair.<versionName>.<channel>.<buildType>.apk`.
* Gradle plugins (root `build.gradle`): Android Gradle Plugin 9.x, Baseline Profile,
  Kotlin Compose / Parcelize / Serialization, KSP, Protobuf, Licensee, Rikka Refine
  (hidden-API access), Spotless (formatting); Compose stability config in
  `compose_compiler_config.conf`; Develocity build scans publish on CI only. A `:dagger`
  module provides DI.

## 4. Runtime architecture of the Lawnchair layer (`lawnchair/src/app/lawnchair`)

### 4.1 Entry points
* `LawnchairApp` — `Application`; initialises root helper, preference managers, process initializer
  (`LawnchairProcessInitializer`). Exposes `isRecentsEnabled` (Quickstep integration state).
* `LawnchairLauncher` — subclasses `QuickstepLauncher`; wires gestures
  (`GestureController`, `VerticalSwipeTouchController`), popups, theming, overlay/feed,
  sleep (`RootHelperManager`), and Lawnchair-specific states.
* `ui/preferences/PreferenceActivity` — settings UI host (Jetpack Compose).
* `gestures/ui/LawnchairShortcutActivity`, `ApplyIconPackActivity`, `BlankActivity` — auxiliary activities.

### 4.2 Settings / preferences (two generations)
| Layer | Location | Storage | Notes |
|---|---|---|---|
| Legacy | `preferences/PreferenceManager.kt`, `BasePreferenceManager`, `PrefEntry` | `SharedPreferences` | View-based code still reads these; ~80 entries |
| Current | `preferences2/PreferenceManager2.kt` | Jetpack **DataStore** + Kotlin `Flow` | ~108 `preference(...)` entries; `firstCached()` for sync reads; `IdpPreference` for grid values that feed `InvariantDeviceProfile` |

Settings screens live in `ui/preferences/destinations/*` (≈ 30 screens: General, Home Screen,
Home Screen Grid, Dock, App Drawer, App Drawer Folders, Folders, Icons/Icon Shape, Gestures,
Search, Search Providers, Smartspace, Predictions, Quickstep, Fonts, Hidden Apps,
Backup & Restore, Experimental, Feature Flags, Debug…). `PreferenceViewModel` /
`PreferenceInteractor` bridge UI ↔ managers; `PreferenceActivity` hosts navigation.

Changing a grid-related preference triggers `ReloadHelper` / `DeviceProfileOverrides`,
which reloads the launcher model/profile.

### 4.3 Feature packages
| Package | Responsibility | Key classes / variants |
|---|---|---|
| `gestures/` | Workspace swipe-up/down/left/right and double-tap → configurable actions | `GestureType` (`SWIPE_UP/DOWN/LEFT/RIGHT`), `GestureController` (also handles double-tap via `onDoubleTap`), handlers: OpenAppDrawer, OpenApp, OpenAppSearch, OpenSearch, OpenAssistant, OpenNotifications, OpenQuickSettings, Recents, Sleep (`SleepMode`: AUTO / ROOT / ACCESSIBILITY / DEVICE_ADMIN) |
| `hotseat/` | Dock | `HotseatMode`: `disabled`, `lawnchair` (default), `google_search` (needs Google app); `HotseatPagedView` |
| `qsb/` | Search bar widget | `LawnQsbLayout`, `GoogleQsbContainerView`; ~27 search provider definitions in `qsb/providers` (Google, Bing, DuckDuckGo, Brave, Firefox family, Kagi, Wikipedia, YouTube, GitHub, …) |
| `search/` | Global (all-apps) search | `LawnchairSearchAlgorithm` factory picks **ASI** (Android System Intelligence, API ≥ 31 + Quickstep enabled), **Local**, or **App-only** based on the `searchAlgorithm` pref. Providers (`engine/provider`): apps, contacts, files, settings, shortcuts, history, calculator, web (built-in + custom + suggestions) |
| `smartspace/` | At a Glance | `SmartspaceMode`: `lawnchair`, `google`, `google_search`, `smartspacer`; data providers: battery, torch, now-playing, onboarding, calendar, widget reader |
| `icons/` | Icon pipeline | `LawnchairIconProvider`, `LawnchairThemeManager`, `IconPackProvider` + `iconpack/*` (system / custom packs), `shape/*` (icon shapes, custom path shapes), themed icons |
| `theme/` | Colours | Material 3 Expressive; `ThemeProvider`, `ComposeColorScheme`, wallpaper/system colour extraction (`AccentColorExtractor`) |
| `allapps/` | App drawer | `LawnchairAlphabeticalAppsList`, search input, work-profile tab styling |
| `deck/`, `data/folder/` | "Lawndeck" automatic app-drawer folders | `LawndeckManager`, `AddFoldersWithItemsTask`; Room entities/DAO/service (`FolderDao`, `FolderService`) |
| `flowerpot/` | App categorisation rules | `Flowerpot`, parser (`FlowerpotReader`, `LineParser`), `rules/*`; format spec in `flowerpot/README.md` (version 1, codename *azalea*) |
| `data/` | Room database (`AppDatabase`) | folders, icon overrides (`IconOverride*`), wallpaper |
| `backup/` | Backup & restore | `LawnchairBackup` — ZIP (`.lawnchairbackup`, `BACKUP_VERSION = 1`) containing `info.pb`, `launcher.db`, shared prefs + DataStore prefs, optional `wallpaper.png`, `screenshot.png`. Options: layout+settings, wallpaper. Also `NovaBackupConverter` |
| `overview/` | Recents extras | `LawnchairOverviewActionsView`, `TaskOverlayFactoryImpl`, `TaskShortcutFactory` |
| `root/` | Root helper (QuickSwitch, screen-off) | `RootHelperManager`, `RootHelperBackend` (AIDL service), `RootNotAvailableException` |
| `wallpaper/` | Wallpaper depth effect, wallpaper service | |
| `nexuslauncher/`, `override/`, `factory/` | Pixel-launcher-compatible overlay (feed), Launcher3 override hooks and factories | `OverlayCallbackImpl`, `FeedBridge` |
| `ui/popup/` | Long-press popup menus | `LauncherOptionsPopup`, `LawnchairShortcut` |

### 4.4 Interaction with AOSP code
Lawnchair customises AOSP through **subclassing and override hooks** rather than editing
`src/` directly (e.g. `LawnchairLauncher extends QuickstepLauncher`, `override/` and `factory/`
packages providing replacement implementations of Launcher3 factories). Where a direct edit
of AOSP code was unavoidable, the repository comments mark it `Lawnchair-TODO` / `Lawnchair`.
When merging upstream, expect conflicts mainly in `src/`, `quickstep/` and `build.gradle`.

### 4.5 Threading & data
* Kotlin coroutines + Flows for UI-facing state (preferences2, Room).
* AOSP `Executors.MODEL_EXECUTOR` / `LauncherModel` for workspace/DB load; launcher DB is
  the AOSP `launcher.db` (SQLite) — Room `AppDatabase` is Lawnchair-only data.

## 5. Android-version support

`minSdk 26` (Android 8). `compatLib*` modules isolate API differences for Q (10) through
Baklava (16). Several features are gated at runtime:
* ASI search: API ≥ 31 **and** Quickstep recents enabled.
* Quickstep/QuickSwitch: Android 15–16 and root for the QuickSwitch path.
* Google hotseat / Google smartspace: require `com.google.android.googlequicksearchbox`.
* Play flavor: features may be restricted to satisfy Google Play policy.

## 6. Known gaps / things to be aware of

* The submodule `platform_frameworks_libs_systemui` must be initialised
  (`git submodule update --init --recursive`) or the build fails.
* The repository contains **AOSP tests** under `tests/`, but no unit/instrumentation tests
  specific to `app.lawnchair.*`. The regression suite in `docs/testing/` therefore starts
  as a manual/semi-automated plan; section 7 of `DEVELOPMENT.md` suggests which parts to
  automate first.
* Official developer docs live at <https://docs.lawnchair.app/developers>.
