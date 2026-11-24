# Lawnchair 16 — Regression Test Plan

**120 test cases** in 18 sections. Machine-readable copy: [`regression-test-cases.csv`](regression-test-cases.csv) (import into any test-management tool).

## 1. Purpose & scope
A standard, repeatable regression checklist for the Lawnchair 16 launcher (Launcher3 + Quickstep + Lawnchair layer). It was derived from reading the source (settings, feature packages, recent fixes in `git log`) — **not** from running the app. Treat expected results as the intended behaviour and correct them where actual product behaviour differs; mark any such correction in the Notes column of your run sheet.

## 2. How to use
* **Smoke (≈30 min):** all `P0`.
* **Release candidate:** `P0` + `P1`.
* **Full:** everything, plus a second device from the matrix below.
* Record result per case: `Pass / Fail / Blocked / N-A`, build (flavor + versionName), device, Android version, tester, bug link.
* A build **fails regression** if any P0 fails, or ≥2 P1 fail without an accepted waiver.

**Priority:** P0 = launcher unusable / data loss if broken; P1 = major feature; P2 = minor/edge.
**Type:** *Manual* = needs human/hardware; *Automatable* = can be a JVM/Robolectric unit test (see `docs/DEVELOPMENT.md` §7); *UI-automation candidate* = can be scripted with UI Automator/TAPL.

## 3. Test environment matrix (minimum)
| Axis | Values |
|---|---|
| Android | 16 (primary, Quickstep path); 15; one of 13/14; API 26–28 for minSdk smoke |
| Form factor | phone; tablet or foldable (at least once per release) |
| Flavors | `lawnWithQuickstepGithubDebug/Release` and `...PlayRelease` (Play restricts some features) |
| Providers | Device with Google app; device/emulator without it; with and without Smartspacer, an icon pack, a work profile |
| Root | One rooted device (QuickSwitch/sleep root mode) and one unrooted |

## 4. Preconditions for every run
1. Clean install of the build under test **and** an upgrade install from the previous release.
2. Submodule initialised before building: `git submodule update --init --recursive`.
3. `adb logcat -c` before starting; save logcat at the end.

## 5. Known regressions to re-verify (from recent history)
| Change | Related case |
|---|---|
| Dock icon vertical spacing + startup crash (#7478/#7580) | TC-DOCK-005, TC-INST-003 |
| YouTube search provider icon solid circle (#7581) | TC-SRCH-009 |
| Animation callbacks during widget placement (#7579) | TC-HOME-005, TC-HOME-006 |
| ANR when restoring popup / migrating legacy pref (#7571) | TC-PREF-003, TC-INST-002 |
| `defaultHomePage` used in more places (#7473) | TC-HOME-010 |
| QuickSwitch Baklava split screen (#7570) | TC-QS-005 |

## 6. Summary
| Priority | Count |
|---|---|
| P0 | 30 |
| P1 | 57 |
| P2 | 33 |

| Type | Count |
|---|---|
| UI-automation candidate | 27 |
| Manual | 80 |
| Automatable (JVM/Robolectric) | 13 |


## 7. Test cases

### Install, launch & lifecycle

Baseline sanity. Run on every build.

| ID | Pri | Type | Title | Preconditions | Steps | Expected |
|---|---|---|---|---|---|---|
| TC-INST-001 | P0 | UI-auto | Fresh install launches as home app | Clean device, APK installed | 1. Install APK. 2. Press Home. 3. Choose Lawnchair as default home app. | Home screen renders with default layout, no crash/ANR, status bar and dock visible. |
| TC-INST-002 | P0 | Manual | Upgrade install preserves data | Previous release installed with custom layout, icon pack and settings | 1. `adb install -r` new APK. 2. Press Home. | Layout, folders, widgets, dock, icon pack and settings unchanged; legacy pref migration (SharedPreferences → DataStore) causes no ANR. |
| TC-INST-003 | P0 | UI-auto | Cold start time & no crash | App force-stopped | 1. Force-stop. 2. Press Home 5 times, measuring time to first frame. | Home appears; no crash in logcat (`FATAL EXCEPTION`/`ANR in`); startup comparable to previous build (±20%). |
| TC-INST-004 | P1 | UI-auto | Process death restore | Home screen with 2 pages, one folder open | 1. `adb shell am kill app.lawnchair*`. 2. Press Home. | Launcher restarts to home page (or last page per 'remember position'), layout intact. |
| TC-INST-005 | P0 | Manual | Reboot persistence | Customised layout | 1. Reboot device. 2. Unlock. | Lawnchair is home; layout, widgets, wallpaper, settings intact. |
| TC-INST-006 | P2 | Manual | Each channel installs side-by-side correctly | github, nightly, play APKs | 1. Install `app.lawnchair`, `app.lawnchair.nightly`, `app.lawnchair.play`. | All three coexist (distinct application IDs); each launches independently. |
| TC-INST-007 | P2 | Manual | Uninstall/ clear data | Installed | 1. Clear app data. 2. Press Home. | Launcher rebuilds default layout with no crash. |

### Home screen / workspace

Core Launcher3 behaviour as modified by Lawnchair.

| ID | Pri | Type | Title | Preconditions | Steps | Expected |
|---|---|---|---|---|---|---|
| TC-HOME-001 | P0 | UI-auto | Add app shortcut by drag from drawer | Drawer open | 1. Long-press an app in drawer. 2. Drop on home page. | Icon placed in the cell under finger; persists after restart. |
| TC-HOME-002 | P0 | UI-auto | Move icon within and across pages | ≥2 pages | 1. Long-press icon. 2. Drag to other page edge. 3. Drop. | Icon moves; page auto-scrolls; neighbours reflow per grid rules; no overlap. |
| TC-HOME-003 | P1 | UI-auto | Remove icon / uninstall target | Icon on home | 1. Drag icon to Remove. 2. Drag app icon to Uninstall (cancel). | Remove deletes shortcut only; Uninstall prompts system dialog. |
| TC-HOME-004 | P1 | Manual | Add / remove / reorder pages | Home screen | 1. Drag icon to right edge to create new page. 2. Empty a page. | New page created; empty pages are removed according to AOSP rules; page indicator updates. |
| TC-HOME-005 | P0 | UI-auto | Add widget (normal) | Widget with no config activity | 1. Long-press → Widgets. 2. Drag widget to home. | Widget placed and renders; survives restart. |
| TC-HOME-006 | P1 | Manual | Add widget with configuration activity | Widget needing config (e.g. calendar) | 1. Add widget. 2. Complete config. | Config activity shown; widget placed after OK; cancel removes it cleanly. |
| TC-HOME-007 | P1 | Manual | Widget resize + 'force resize' / 'unlimited size' / 'allow overlap' prefs | Settings → Home screen widget options toggled | 1. Enable options. 2. Resize widget beyond provider limits / overlap another. | Behaviour follows each toggle; disabling reverts to constrained behaviour. |
| TC-HOME-008 | P2 | Manual | Rounded widgets toggle | Widget on home | 1. Toggle 'rounded widgets'. | Widget corners update (where supported) with no crash. |
| TC-HOME-009 | P1 | UI-auto | Lock home screen | Setting 'Lock home screen' enabled | 1. Long-press icon/empty area; try to drag. | Drag/edit blocked; popup lock button state consistent; disabling restores edit. |
| TC-HOME-010 | P1 | Manual | Remember last page vs default home page | Settings 'Remember position' / `defaultHomePage` | 1. Go to page 3. 2. Press Home / leave and return. | Behaviour matches the setting. |
| TC-HOME-011 | P2 | Manual | Wallpaper & depth effect | Depth effect available | 1. Toggle wallpaper depth effect. 2. Open drawer. | Parallax/zoom as configured; no jank/crash. |
| TC-HOME-012 | P2 | Manual | Status bar & dark status bar options | Settings | 1. Toggle 'show status bar', 'dark status bar', 'status bar clock'. | Status bar visibility/ icon colour follow settings over light and dark wallpapers. |
| TC-HOME-013 | P1 | Manual | Rotation (if rotation allowed) and tablet/foldable | Rotatable device or foldable | 1. Rotate. 2. Fold/unfold. | Grid recomputed, no lost icons, no crash; items stay on same pages. |

### Grid, sizes & layout settings

`DeviceProfileOverrides`, `IdpPreference`, size factors.

| ID | Pri | Type | Title | Preconditions | Steps | Expected |
|---|---|---|---|---|---|---|
| TC-GRID-001 | P0 | UI-auto | Change home grid rows/columns | Settings → Home screen → Grid | 1. Increase then decrease rows & columns. | Launcher reloads; icons are re-placed without loss; reduced grid pushes extra icons to new page rather than deleting. |
| TC-GRID-002 | P1 | Manual | Icon size factor (home & drawer) | Defaults | 1. Move `homeIconSizeFactor` / `drawerIconSizeFactor` to min and max. | Icons scale; no clipping beyond cell; reset returns to default. |
| TC-GRID-003 | P1 | Manual | Label size & visibility (home, folder, drawer) | Defaults | 1. Toggle labels off/on; change label size factors; enable two-line drawer labels. | Label state follows settings in each context independently. |
| TC-GRID-004 | P2 | Manual | Padding / margin factors (workspace, widget, drawer, hotseat bottom) | Defaults | 1. Adjust each factor to extremes. | Layout adjusts without overlap with dock/status bar; reset works. |
| TC-GRID-005 | P1 | Manual | Drawer grid (columns, cell height) | Defaults | 1. Change drawer columns / cell height factor. | Drawer reflows; fast-scroller still works. |
| TC-GRID-006 | P1 | Manual | Settings survive restart and backup | Custom grid | 1. Restart launcher. 2. Back-up and restore. | Grid values restored exactly. |

### Dock / hotseat

`HotseatMode`: disabled / lawnchair / google_search.

| ID | Pri | Type | Title | Preconditions | Steps | Expected |
|---|---|---|---|---|---|---|
| TC-DOCK-001 | P0 | Manual | Hotseat mode: Lawnchair (default) | Settings → Dock | 1. Select 'Lawnchair'. | Lawnchair QSB shown in dock/above dock; apps in dock launch. |
| TC-DOCK-002 | P1 | Manual | Hotseat mode: Disabled | Dock settings | 1. Select 'Disabled'. | Search bar hidden (empty view); dock icons still usable. |
| TC-DOCK-003 | P1 | Manual | Hotseat mode: Google Search (Google app installed) | Google app installed and enabled | 1. Select 'Google Search'. | Google-style bar appears; tapping opens Google search. |
| TC-DOCK-004 | P1 | Manual | Hotseat mode: Google Search unavailable | Google app disabled/absent | 1. Open mode list. | Option unavailable/hidden (`isAvailable=false`); stored value 'google_search' falls back sanely; no crash. |
| TC-DOCK-005 | P1 | Manual | Dock enable/disable and background styling | Dock settings | 1. Toggle `isHotseatEnabled`. 2. Change background colour/ corner radius. | Dock hides/shows; background reflects settings; icons keep vertical spacing (regression: #7478, #7580). |
| TC-DOCK-006 | P2 | Manual | Paged hotseat | Hotseat paging available | 1. Swipe in dock. | Pages behave and persist. |
| TC-DOCK-007 | P2 | Manual | Dock label option | `enableLabelInDock` | 1. Toggle. | Labels in dock follow setting. |

### Folders

Home folders and app-drawer folders (Lawndeck).

| ID | Pri | Type | Title | Preconditions | Steps | Expected |
|---|---|---|---|---|---|---|
| TC-FOLD-001 | P0 | UI-auto | Create folder by dropping icon on icon | Two icons | 1. Drag one icon onto another. | Folder created containing both; persists. |
| TC-FOLD-002 | P0 | UI-auto | Rename folder | Folder | 1. Open → tap title → edit → Done. | Title persists after restart. |
| TC-FOLD-003 | P1 | UI-auto | Folder open/close animation & drag out | Folder with ≥3 items | 1. Open. 2. Drag item out. 3. Remove until 1 item left. | Item leaves; folder with 1 item collapses to single icon (AOSP behaviour); no crash. |
| TC-FOLD-004 | P2 | Manual | Folder style settings (colour, opacity, shape, preview) | Folder settings | 1. Change folder shape/colour/opacity. | Closed and open folder reflect changes. |
| TC-FOLD-005 | P1 | Manual | Lawndeck: auto folders in drawer | Drawer-folders feature enabled (`deckLayout`) | 1. Enable. 2. Open drawer. | Folders created from flowerpot categories; apps listed correctly; no duplicate folders. |
| TC-FOLD-006 | P1 | Manual | App-drawer folder CRUD | App drawer folders screen | 1. Create folder, select apps, rename, reorder, hide, delete. | Each operation persists (Room `FolderDao/Service`); deleted folder's apps back in main list. |
| TC-FOLD-007 | P2 | Manual | Folder with uninstalled app | App in drawer folder | 1. Uninstall app. | Entry removed/ignored; no crash on opening drawer. |

### App drawer

| ID | Pri | Type | Title | Preconditions | Steps | Expected |
|---|---|---|---|---|---|---|
| TC-DRAW-001 | P0 | UI-auto | Open drawer via swipe up and via gesture | Default gestures | 1. Swipe up from home. 2. Press Home to close. | Drawer opens/closes smoothly; scrim/background per settings. |
| TC-DRAW-002 | P0 | UI-auto | Alphabetical list, fast scroller, section letters | ≥50 apps | 1. Scroll. 2. Drag fast-scroller. | Letter bubble shows; list jumps correctly. |
| TC-DRAW-003 | P1 | Manual | Work profile tab | Work profile present | 1. Open drawer → Work tab. 2. Toggle work mode. | Tab styling per settings; apps greyed/paused when work off. |
| TC-DRAW-004 | P0 | UI-auto | Hidden apps | Settings → Hidden apps | 1. Hide an app. 2. Search for it (with and without 'show hidden apps in search'). | Not in drawer; appears in search only if `hiddenAppsInSearch`; unhide restores. |
| TC-DRAW-005 | P1 | Manual | Suggested/predicted apps row | Predictions enabled | 1. Launch several apps. 2. Open drawer. | Predictions row appears/updates; disabling removes it; dismissed predictions respected. |
| TC-DRAW-006 | P1 | Manual | Drawer search bar visibility & keyboard auto-show | `hideAppDrawerSearchBar`, `autoShowKeyboardInDrawer` | 1. Toggle each. | Bar hidden/shown; keyboard shows on open when enabled. |
| TC-DRAW-007 | P0 | UI-auto | App long-press popup (shortcuts, widgets, app info, uninstall) | App with static shortcuts | 1. Long-press app. | Popup lists system shortcuts, Lawnchair actions; ordering follows 'launcher popup order'. |
| TC-DRAW-008 | P2 | Manual | Drawer icon actions: drag to home, App info | Drawer open | 1. Drag an app from drawer to home. 2. Long-press → App info. | Shortcut added to home; App info screen opens; no crash. |

### Search (all-apps search & QSB)

Algorithm chosen by `searchAlgorithm` pref: ASI / Local / App-only.

| ID | Pri | Type | Title | Preconditions | Steps | Expected |
|---|---|---|---|---|---|---|
| TC-SRCH-001 | P0 | UI-auto | App search – prefix & substring | Drawer | 1. Type 'cal'. | Apps whose labels match listed first; clearing returns list. |
| TC-SRCH-002 | P1 | Auto | Fuzzy search toggle | `enableFuzzySearch` | 1. Search for a misspelling ('chorme'). | Fuzzy ON → finds Chrome; OFF → no match. |
| TC-SRCH-003 | P1 | Manual | Local algorithm: contacts, files, settings, shortcuts, history | Permissions granted; algorithm = Local | 1. Search for contact name, filename, 'wifi'. | Each section shows with limits from `max*ResultCount` prefs; without permission the section is absent / prompts. |
| TC-SRCH-004 | P1 | Auto | Calculator results | Local algorithm | 1. Search '2+2*3', '10/4', '(1+2)^2', invalid '2+'. | Valid expressions show correct result (8, 2.5, 9); invalid shows nothing; no crash. |
| TC-SRCH-005 | P1 | Manual | Web suggestions / providers (built-in, custom) | Network; web suggestion provider set | 1. Type a query. 2. Switch provider; add custom provider URL. | Suggestions limited to `maxWebSuggestionResultCount`; tapping opens selected provider; custom URL works with `%s` replacement. |
| TC-SRCH-006 | P1 | Auto | ASI search availability gating | API ≥31 + Quickstep vs. other | 1. Select ASI algorithm on supported and unsupported config. | Supported → ASI used; unsupported → falls back to App search (no crash). |
| TC-SRCH-007 | P2 | Manual | Recent searches / history | History provider on | 1. Search + open results. 2. Reopen search. | Recents appear; clear works. |
| TC-SRCH-008 | P2 | Manual | Search result limits & sections order | - | 1. Change limit prefs. | Counts match prefs; no duplicate dividers (`removeDuplicateDividers`). |
| TC-SRCH-009 | P1 | Manual | QSB provider choice (each major provider) | Search bar on home | 1. Pick Google, Bing, DuckDuckGo, Brave, Firefox, Kagi, Wikipedia, YouTube, GitHub… | Icon renders correctly (regression #7581: YouTube icon solid circle); tapping opens provider app or website; uninstalled-app providers fall back to web. |
| TC-SRCH-010 | P2 | Manual | QSB style: themed, matching hotseat, corner radius | - | 1. Toggle `themedHotseatQsb`, `matchHotseatQsbStyle`. | Colours/shape update live. |

### Gestures

`GestureType` = swipe up/down/left/right; double-tap handled by `GestureController`.

| ID | Pri | Type | Title | Preconditions | Steps | Expected |
|---|---|---|---|---|---|---|
| TC-GEST-001 | P0 | UI-auto | Default swipe-up opens drawer | Defaults | 1. Swipe up on home. | Drawer opens. |
| TC-GEST-002 | P1 | UI-auto | Swipe-down default (notifications) | Defaults | 1. Swipe down on home. | Notification shade opens (needs accessibility handler where applicable). |
| TC-GEST-003 | P1 | Manual | Assign each handler to each gesture | Settings → Gestures | 1. For each of Open drawer, Open app, App search, Search, Assistant, Notifications, Quick settings, Recents, Sleep: assign & trigger. | Handler runs; handlers needing accessibility prompt for it and degrade gracefully when not granted. |
| TC-GEST-004 | P1 | Manual | Double-tap to sleep (Auto/Root/Accessibility/Device-admin) | Each prerequisite configured | 1. Choose each `SleepMode`. 2. Double-tap. | Screen locks/turns off under each mode; missing prerequisite shows prompt, no crash. |
| TC-GEST-005 | P1 | Manual | Open specific app via gesture (+ app picker) | - | 1. Choose app. 2. Uninstall that app. 3. Trigger. | App opens; after uninstall no crash (no-op or reset). |
| TC-GEST-006 | P2 | Manual | Gestures disabled when home locked / in overview | - | 1. Trigger in non-home state. | No unintended actions. |
| TC-GEST-007 | P2 | Manual | Icon-level gestures | - | 1. Swipe on dock icon (if configured). | Configured action fires. |

### Icons, themes & colours

| ID | Pri | Type | Title | Preconditions | Steps | Expected |
|---|---|---|---|---|---|---|
| TC-ICON-001 | P0 | UI-auto | Apply icon pack | Icon pack installed | 1. Settings → Icon pack → select. 2. Return home. | Icons from pack applied (drawer, home, folders); fallback for unthemed apps; reselect system restores. |
| TC-ICON-002 | P1 | Manual | Per-app icon override (icon picker) | - | 1. Long-press → edit icon → pick drawable. 2. Restart. | Override persists (Room `IconOverride`); reset removes it. |
| TC-ICON-003 | P1 | Manual | Icon shape & custom shape | - | 1. Select each shape; create custom path. | Shape applied to adaptive icons, folder shape separate when set; invalid path rejected. |
| TC-ICON-004 | P1 | Manual | Themed icons (Material You) | Android 13+ with themed icon support | 1. Enable themed icons. 2. Change wallpaper. | Monochrome icons tint to palette; apps without monochrome handled via fallback. |
| TC-ICON-005 | P1 | Manual | Accent / colour style, dark & light mode | Wallpaper colours available | 1. Toggle system dark mode. 2. Change accent/colour style. | Drawer, popups, settings, folders recolour correctly; text contrast adequate. |
| TC-ICON-006 | P2 | Manual | Fonts selection & additional fonts | `enableFontSelection` | 1. Choose font; add custom font. | Launcher text uses selected font; invalid font file rejected. |
| TC-ICON-007 | P1 | Manual | Notification dots & counts | Notification access granted | 1. Post notification. 2. Toggle `showNotificationCount`; change dot colours. | Dot (and count) appear/disappear; clearing notification removes dot. |
| TC-ICON-008 | P1 | UI-auto | Always reload icons / package install, update, remove | - | 1. Install, update, uninstall an app via adb. | Icon appears/updates/disappears in drawer and on home within seconds. |

### Smartspace / At a Glance

`SmartspaceMode`: lawnchair / google / google_search / smartspacer.

| ID | Pri | Type | Title | Preconditions | Steps | Expected |
|---|---|---|---|---|---|---|
| TC-SMRT-001 | P0 | Manual | Enable Lawnchair smartspace | Settings → Smartspace | 1. Enable; mode = Lawnchair. | Date/time rows appear; `smartspaceTimeFormat` honoured. |
| TC-SMRT-002 | P1 | Manual | Providers: battery, torch, now playing, onboarding, calendar | Each trigger available | 1. Charge low battery / turn on torch / play music / have calendar event. | Matching card appears and clears. |
| TC-SMRT-003 | P1 | Manual | Smartspacer integration | Smartspacer installed | 1. Mode = Smartspacer. | Cards from Smartspacer shown; uninstalling it falls back with no crash. |
| TC-SMRT-004 | P1 | Manual | Google modes unavailable | Google app absent | 1. Select Google modes. | Option not available; no crash. |
| TC-SMRT-005 | P2 | Manual | Smartspace tap actions | - | 1. Tap date / card. | Calendar / relevant app opens. |

### Quickstep / Recents / navigation

Needs QuickSwitch/system integration; run on supported Android 15/16 device (root where needed).

| ID | Pri | Type | Title | Preconditions | Steps | Expected |
|---|---|---|---|---|---|---|
| TC-QS-001 | P0 | Manual | Swipe-up-to-home from app | Quickstep working | 1. Open app. 2. Swipe up. | Window-to-icon animation; lands on home; no stuck surface. |
| TC-QS-002 | P0 | Manual | Overview (recents) open, scroll, dismiss task, clear all | Several apps open | 1. Open overview. 2. Swipe task away. 3. Clear all. | Tasks update; no freeze. |
| TC-QS-003 | P1 | Manual | Quick switch (swipe on nav bar) | - | 1. Swipe horizontally on nav pill. | Switches to previous app. |
| TC-QS-004 | P1 | Manual | Overview actions (screenshot, select, share) & task shortcuts | - | 1. Open overview; use actions. | Actions fire; Lawnchair overlay tweaks visible. |
| TC-QS-005 | P1 | Manual | Split screen (incl. Baklava) | Android 16 device | 1. Overview → split-screen on a task. 2. Pick second app. | Split session starts (regression #7570) |
| TC-QS-006 | P2 | Manual | Taskbar on large screens / `enableTaskbarOnPhone` | Tablet or flag on | 1. Toggle taskbar. | Taskbar shown/hidden; drag-to-split works. |
| TC-QS-007 | P2 | Manual | Quickstep unavailable warning | Not system recents provider | 1. Launch app. | Warning shown (`showQuickstepWarningIfNecessary`); basic launcher still works. |

### Backup & restore

`LawnchairBackup` — ZIP `.lawnchairbackup`, `BACKUP_VERSION=1`.

| ID | Pri | Type | Title | Preconditions | Steps | Expected |
|---|---|---|---|---|---|---|
| TC-BKP-001 | P0 | Manual | Create backup (layout+settings) | Custom layout | 1. Settings → Backup → Create. 2. Choose layout & settings. 3. Save. | File `Lawnchair_Backup <date>.lawnchairbackup` created; opens as ZIP containing `info.pb`, `launcher.db`, prefs files, `screenshot.png`. |
| TC-BKP-002 | P1 | Manual | Create backup incl. wallpaper | - | 1. Tick wallpaper. | ZIP additionally has `wallpaper.png`. |
| TC-BKP-003 | P0 | Manual | Restore on same device | Backup exists, layout since changed | 1. Restore → pick file → confirm. | Launcher restarts; layout/settings equal to backup. |
| TC-BKP-004 | P1 | Manual | Restore on different device/screen size | Backup from other grid | 1. Restore. | Layout adapts; no crash; missing apps omitted. |
| TC-BKP-005 | P1 | Auto | Corrupt / wrong file / newer version | Truncated zip; random file; modified `info.pb` version | 1. Attempt restore. | Clear error; current layout untouched. |
| TC-BKP-006 | P2 | Manual | Restore Nova backup | Nova backup file | 1. Restore Nova backup. | Converted layout appears (`NovaBackupConverter`); unsupported items skipped. |
| TC-BKP-007 | P0 | Auto | Zip round-trip unit test | JVM | 1. Write backup to temp stream; read back handlers. | Entries identical; version constant checked. |

### Flowerpot (app categorisation)

Format spec: `flowerpot/README.md` — `$` version, `#` comment, `:` intent action, `;` category, `&` code rule, bare package name.

| ID | Pri | Type | Title | Preconditions | Steps | Expected |
|---|---|---|---|---|---|---|
| TC-FLWP-001 | P0 | Auto | Parse valid file | JVM | 1. Parse sample with `$1`, packages, `:`, `;`, `&Rule\|arg`. | Rules list has expected types and order. |
| TC-FLWP-002 | P0 | Auto | Version rules | JVM | 1. Duplicate `$` version line. 2. Unsupported version (e.g. `$99`). 3. Rule before version line. | Cases 1 and 2 throw `FlowerpotFormatException` ("can only appear once" / "Unsupported version"); verify case 3 against `LineParser` and record actual behaviour. |
| TC-FLWP-003 | P1 | Auto | Comments & blank lines ignored | - | 1. Parse file with `#` lines. | No rule produced. |
| TC-FLWP-004 | P1 | Auto | Matching packages / intents | Fake PackageManager | 1. Match app with package rule, intent-action rule, category rule. | Matches only declared apps. |
| TC-FLWP-005 | P2 | Auto | Unknown code rule | - | 1. Parse line `&Unknown`. | Behaviour per `CodeRules`/`LineParser`; record actual result and pin it as the expected value. |

### Preferences & settings UI

~108 DataStore prefs + ~80 legacy prefs.

| ID | Pri | Type | Title | Preconditions | Steps | Expected |
|---|---|---|---|---|---|---|
| TC-PREF-001 | P0 | UI-auto | Open every settings screen | - | 1. Navigate through every screen in Preferences dashboard, incl. Experimental, Feature flags, Debug. | No crash; back navigation works; screen state retained on rotation. |
| TC-PREF-002 | P0 | Auto | Defaults on fresh install | Empty DataStore | 1. Read all `PreferenceManager2` values. | Equal to documented defaults (snapshot test). |
| TC-PREF-003 | P1 | Auto | Legacy → DataStore migration | SharedPreferences with legacy values | 1. Run `SharedPreferencesMigration`. | Values migrated once; no ANR (regression #7571); idempotent. |
| TC-PREF-004 | P1 | Manual | Preference change reflected live | - | 1. Change setting while launcher visible (split/ multi-window). | Home updates without restart where designed; otherwise prompts restart. |
| TC-PREF-005 | P2 | Manual | Search within settings & about page | - | 1. Open About, licences. | Opens; links launch browser. |
| TC-PREF-006 | P2 | Manual | Settings dark/light + dynamic colour | - | 1. Toggle. | UI adapts; no unreadable text. |
| TC-PREF-007 | P2 | Manual | Reset / feature flags don't persist beyond intended | - | 1. Toggle flag; restart. | Flag persisted or reset per design. |

### Permissions, accessibility, root

| ID | Pri | Type | Title | Preconditions | Steps | Expected |
|---|---|---|---|---|---|---|
| TC-PERM-001 | P0 | Manual | Runtime permissions requested only when needed | Fresh install | 1. Use contacts/file search; calendar smartspace. | Permission requested at use; denial leaves feature disabled, no crash. |
| TC-PERM-002 | P1 | Manual | Accessibility service required actions | - | 1. Trigger gestures needing accessibility w/o service. | Prompt to enable; after enabling works (`LawnchairAccessibilityService`). |
| TC-PERM-003 | P2 | Manual | Root helper availability | Rooted and non-rooted | 1. Use root-only features. | Rooted: works. Non-rooted: `RootNotAvailableException` handled, user-visible message. |
| TC-PERM-004 | P1 | Manual | Notification listener access | - | 1. Grant/revoke. | Dots appear only when granted. |

### Multi-user, profiles, devices

| ID | Pri | Type | Title | Preconditions | Steps | Expected |
|---|---|---|---|---|---|---|
| TC-PROF-001 | P1 | Manual | Work profile apps & badges | Work profile | 1. Open work app; pause work. | Badge icon; paused state; widgets from work profile. |
| TC-PROF-002 | P2 | Manual | Private space (Android 15+) | Private space set up | 1. Lock/unlock private space. | Apps hidden/shown without crash. |
| TC-PROF-003 | P2 | Manual | Secondary user / guest | - | 1. Switch user. | Launcher loads per-user data. |
| TC-PROF-004 | P2 | Manual | Tablet, foldable, multi-display | - | 1. Run TC-HOME-013, TC-GRID-001. | Layouts valid on all form factors. |
| TC-PROF-005 | P2 | Manual | Locale / RTL / large font / display size | Arabic locale, 200% font | 1. Navigate home, drawer, settings. | No clipped essential text; RTL mirrored. |

### Stability & performance

| ID | Pri | Type | Title | Preconditions | Steps | Expected |
|---|---|---|---|---|---|---|
| TC-PERF-001 | P1 | UI-auto | Monkey / fuzz run | - | `adb shell monkey -p <pkg> --throttle 100 -v 20000` | No crash/ANR attributable to Lawnchair. |
| TC-PERF-002 | P2 | UI-auto | Memory leak smoke | - | Open/close drawer, settings, folders 100×. | Heap stable (LeakCanary / `dumpsys meminfo`). |
| TC-PERF-003 | P2 | UI-auto | Scroll jank | ≥100 apps | Fling drawer and home pages (`gfxinfo`). | Janky frames within agreed budget. |
| TC-PERF-004 | P2 | UI-auto | Startup with baseline profile | Macrobenchmark | Cold start benchmark. | No regression vs previous release. |
| TC-PERF-005 | P0 | Manual | Logcat scan | After full pass | `adb logcat -b crash,all \| grep -E 'FATAL\|ANR'` | No entries for Lawnchair processes. |

### Existing upstream (AOSP) tests

Reuse — not new.

| ID | Pri | Type | Title | Preconditions | Steps | Expected |
|---|---|---|---|---|---|---|
| TC-AOSP-001 | P1 | Auto | Launcher3 multivalent tests | Wired for AOSP build | Run suites under `tests/multivalentTests` and `tests/src_deviceless`. | All pass or known-failure list maintained. |
| TC-AOSP-002 | P1 | UI-auto | TAPL smoke | Device | Run `tests/src/.../ui/TaplTestsLauncher3Test`. | Home/ drawer / widget flows pass. |

## 8. Run sheet template
| Case ID | Result | Build | Device / Android | Tester | Date | Bug / notes |
|---|---|---|---|---|---|---|
| | | | | | | |

## 9. Maintenance
* Add a case (next free ID in section) for every bug fixed; link the PR.
* When a case becomes automated, change *Type* and reference the test class.
* Review quarterly and on every Android/Launcher3 upstream merge.
