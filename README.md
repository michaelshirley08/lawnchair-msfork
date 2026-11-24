# Lawnchair 16

[![Build debug APK](https://github.com/LawnchairLauncher/lawnchair/actions/workflows/ci.yml/badge.svg)](https://github.com/LawnchairLauncher/lawnchair/actions/workflows/ci.yml)
[![Build release APK](https://github.com/LawnchairLauncher/lawnchair/actions/workflows/release_update.yml/badge.svg)](https://github.com/LawnchairLauncher/lawnchair/actions/workflows/release_update.yml)
[![Crowdin](https://badges.crowdin.net/e/188ba69d884418987f0b7f1dd55e3a4e/localized.svg)](https://lawnchair.crowdin.com/lawnchair)
[![OpenCollective](https://img.shields.io/opencollective/all/lawnchair?label=financial%20contributors&logo=open-collective)](https://opencollective.com/lawnchair)
[![Telegram](https://img.shields.io/endpoint?url=https%3A%2F%2Ftg.sumanjay.workers.dev%2Flccommunity)](https://t.me/lccommunity)
[![Discord](https://img.shields.io/discord/803299970169700402?label=server&logo=discord)](https://discord.gg/3x8qNWxgGZ)
[![GitHub Downloads](https://img.shields.io/github/downloads/LawnchairLauncher/lawnchair/total.svg?label=GitHub%20Downloads&logo=github)](https://github.com/LawnchairLauncher/lawnchair/releases)
[![Play Store Installs](https://img.shields.io/endpoint?color=green&logo=googleplay&logoColor=green&url=https%3A%2F%2Fplay.cuzi.workers.dev%2Fplay%3Fi%3Dapp.lawnchair.play%26l%3DPlay%2520Store%2520Installs%26m%3D%24shortinstalls)](https://play.google.com/store/apps/details?id=app.lawnchair.play)

<picture>
    <!-- Avoid image being clickable with slight workaround -->
    <source media="(prefers-color-scheme: dark)" srcset="docs/assets/device-frame-dark.webp" width="250px">
    <img alt="Smartphone home screen shown in day and night themes, featuring Android Material You dynamic color styling across the wallpaper, clock widget, and app icons." src="docs/assets/device-frame-light.webp" width="250px">
</picture>

Lawnchair is a free, open-source home app for Android. Taking Launcher3—Android’s default home app—as a starting point, it ports Pixel Launcher features and introduces rich customization options.

This branch houses the codebase of Lawnchair 16, which is currently in development and is based on Launcher3 from Android 16. For Lawnchair 9 to 15, see the branches with the `9-` to `15-` prefixes, respectively.

## Features

- Material 3 Expressive theming that follows your wallpaper and system colors.
- At a Glance widget support, with integration for [Smartspacer](https://github.com/KieronQuinn/Smartspacer).
- QuickSwitch support for Android Recents integration on Android 15-16 (root required).
- Global search for apps, contacts, and web results from the home screen.
- Customization options for icon packs, fonts, and color settings.

## Download

<p align="left">
  <a href="https://play.google.com/store/apps/details?id=app.lawnchair.play">
    <picture>
      <!-- Avoid image being clickable with slight workaround -->
      <source media="(prefers-color-scheme: dark)" srcset="docs/assets/badge-google-play.webp" height="60">
      <img alt="Get it on Google Play" src="docs/assets/badge-google-play.webp" height="60">
    </picture>
  </a>
  <a href="https://apt.izzysoft.de/fdroid/index/apk/app.lawnchair">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset="docs/assets/badge-izzyondroid.webp" height="60">
      <img alt="Get it on IzzyOnDroid" src="docs/assets/badge-izzyondroid.webp" height="60">
    </picture>
  </a>
  <a href="https://apps.obtainium.imranr.dev/redirect?r=obtainium://add/https://github.com/LawnchairLauncher/lawnchair/">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset="docs/assets/badge-obtainium.webp" height="60">
      <img alt="Get it on Obtainium" src="docs/assets/badge-obtainium.webp" height="60">
    </picture>
  </a>
    <a href="https://github.com/LawnchairLauncher/lawnchair/releases">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset="docs/assets/badge-github.webp" height="60">
      <img alt="Get it on GitHub" src="docs/assets/badge-github.webp" height="60">
    </picture>
  </a>
</p>

Lawnchair on Play Store will install as a different app compared to other sources. Features may be restricted to comply with Google Play’s publishing rules.

You can also [verify your installation](https://docs.lawnchair.app/getting-started/install-and-setup/verify) to check if you have installed an official build.

### Development builds

Interested in keeping yourself up-to-date with every Lawnchair development? Try our development builds!

These builds offer the latest features and bug fixes at a cost of performance and additional issues. Make backups before installing.

Download: [Obtainium][Obtainium link] • [GitHub][GitHub link] • [nightly.link][Nightly link]

## Sponsors

<p align="left">
  <a href="https://coderabbit.link/lawnchair">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset="docs/assets/sponsor-coderabbit-dark.svg" width="300">
      <img alt="CodeRabbit" src="docs/assets/sponsor-coderabbit-light.svg" width="300">
    </picture>
  </a>
</p>

[CodeRabbit](https://coderabbit.link/lawnchair) is an AI-powered code review platform that integrates directly into pull-request workflows and IDEs, examining code changes in context and suggesting improvements.

## Support Lawnchair

If you love what we do, consider [supporting us on Open Collective](https://opencollective.com/lawnchair)! Your contributions help keep Lawnchair independent and enable us to develop faster.

A huge thank you to our Core Backers ($5+):
*(These backers directly fund our Project Velocity Fund)*

[![Core Backers](https://opencollective.com/lawnchair/tiers/backer.svg?avatarHeight=64&width=890&button=false)](https://opencollective.com/lawnchair)

[Become a supporter](https://opencollective.com/lawnchair) to help us cover our operational costs, or become a Core Backer to be featured here!

## Contribute

Visit the [Lawnchair contributing guidelines](CONTRIBUTING.md) for information and tips on contributing to Lawnchair.

## Codebase guide (for developers and AI agents)

This section is a map of the repository. It is kept compact; details live in `docs/`.

**What it is:** Lawnchair's own code (Kotlin, Jetpack Compose) sits in `lawnchair/src/app/lawnchair/` on top of AOSP Launcher3 (`src/`), Quickstep/Recents (`quickstep/`), SystemUI (`systemUI/`) and WM Shell (`wmshell/`). Most of the repository is upstream AOSP code. It all builds as a **single Gradle app module**; `lawnchair/`, `src/` and `quickstep/` are source sets wired in the root `build.gradle`, not separate modules.

### Quick start

```bash
git submodule update --init --recursive     # required: platform_frameworks_libs_systemui
./gradlew assembleLawnWithQuickstepGithubDebug
./gradlew spotlessCheck                     # formatting; also run by CI
```

Requires JDK 21 and an Android SDK (compileSdk 37). `minSdk` is 26. Build variants combine `lawn` + `WithQuickstep` + a channel (`Github`, `Nightly` or `Play`) + `Debug` or `Release`.

### Repository map

| Path | Contents |
|---|---|
| `lawnchair/` | **Lawnchair-specific code**, resources, AIDL, assets and protos |
| `src/`, `res/`, `quickstep/`, `systemUI/`, `wmshell/` | Upstream AOSP code; keep diffs minimal |
| `platform_frameworks_libs_systemui/` | Git submodule (icon loader, search UI, animation libs) |
| `compatLib/` | Per-Android-version API shims (Android 10 to 16) |
| `flowerpot/` | App-categorization rule format and tooling |
| `tests/` | Upstream AOSP Launcher3 tests; none cover `app.lawnchair.*` yet |
| `docs/` | Architecture, development guide and regression test plan (below) |
| `fastlane/`, `.github/workflows/` | Store metadata and CI |

`Android.bp`, `CleanSpec.mk`, `OWNERS` and `PREUPLOAD.cfg` are AOSP leftovers that the Gradle build does not use.

### Feature map (`lawnchair/src/app/lawnchair/`)

| Feature | Package |
|---|---|
| Entry points | `LawnchairApp.kt`, `LawnchairLauncher.kt` |
| Settings (new, DataStore) / legacy (SharedPreferences) | `preferences2/` / `preferences/`; screens in `ui/preferences/destinations/` |
| Gestures and sleep modes | `gestures/` |
| Dock modes | `hotseat/` |
| Search bar (QSB) and its providers | `qsb/` |
| All-apps search | `search/` |
| At a Glance / Smartspacer | `smartspace/` |
| Icon packs, shapes, themed icons | `icons/`, `theme/` |
| App-drawer folders | `deck/`, `data/folder/` |
| Backup and restore | `backup/` (`.lawnchairbackup` zip) |
| Root helper | `root/` |

### Conventions

- New settings go in `preferences2/PreferenceManager2.kt`, not the legacy manager.
- Prefer subclassing or overrides (`override/`, `factory/`) over editing AOSP files; mark unavoidable edits with a `Lawnchair` comment.
- Formatting is enforced by Spotless: Java everywhere, Kotlin only under `lawnchair/src/**`.
- English strings only in `values/`; Crowdin manages translations.
- Commit and PR titles use Conventional Commits (`fix:`, `feat(scope):`, `chore(deps):`).
- CI runs on pushes to `*-dev` branches and on PRs, and ignores Markdown-only changes.

### Runtime gating to keep in mind

Some features depend on the device: ASI search needs API 31+ with Quickstep enabled; Google dock and smartspace modes need the Google app; QuickSwitch and root sleep need root; the Play flavor restricts some features. Code must handle the "unavailable" case without crashing.

### More documentation

- [`AGENTS.md`](AGENTS.md) / [`CLAUDE.md`](CLAUDE.md): short, command-oriented instructions for coding agents.
- [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md): structure, build wiring and feature internals.
- [`docs/DEVELOPMENT.md`](docs/DEVELOPMENT.md): build, where to put changes, and a test-automation roadmap.
- [`docs/testing/REGRESSION_TEST_PLAN.md`](docs/testing/REGRESSION_TEST_PLAN.md): 120 regression cases ([CSV](docs/testing/regression-test-cases.csv)).
- [`flowerpot/README.md`](flowerpot/README.md): app-category rule format.
- Official developer docs: <https://docs.lawnchair.app/developers>.

## Quick links

- [Website](https://lawnchair.app)
- [Documentation](https://docs.lawnchair.app/)
- [News on Telegram](https://t.me/lawnchairci)
- [Discord](https://discord.com/invite/3x8qNWxgGZ)
- [X (formerly Twitter)](https://x.com/lawnchairapp)
- [_XDA_ thread](https://xdaforums.com/t/lawnchair-customizable-pixel-launcher.3627137/)

<!-- Download links -->
[Nightly link]: https://nightly.link/LawnchairLauncher/lawnchair/workflows/ci/16-dev
[Obtainium link]: https://apps.obtainium.imranr.dev/redirect?r=obtainium://app/%7B%22id%22%3A%22app.lawnchair.nightly%22%2C%22url%22%3A%22https%3A%2F%2Fgithub.com%2Flawnchairlauncher%2Flawnchair%22%2C%22author%22%3A%22Lawnchair%20Launcher%22%2C%22name%22%3A%22Lawnchair%20(Debug)%22%2C%22preferredApkIndex%22%3A0%2C%22additionalSettings%22%3A%22%7B%5C%22includePrereleases%5C%22%3Atrue%2C%5C%22fallbackToOlderReleases%5C%22%3Afalse%2C%5C%22filterReleaseTitlesByRegEx%5C%22%3A%5C%22Lawnchair%20Nightly%5C%22%2C%5C%22filterReleaseNotesByRegEx%5C%22%3A%5C%22%5C%22%2C%5C%22verifyLatestTag%5C%22%3Afalse%2C%5C%22dontSortReleasesList%5C%22%3Afalse%2C%5C%22useLatestAssetDateAsReleaseDate%5C%22%3Afalse%2C%5C%22trackOnly%5C%22%3Afalse%2C%5C%22versionExtractionRegEx%5C%22%3A%5C%22%5C%22%2C%5C%22matchGroupToUse%5C%22%3A%5C%22%5C%22%2C%5C%22versionDetection%5C%22%3Afalse%2C%5C%22releaseDateAsVersion%5C%22%3Atrue%2C%5C%22useVersionCodeAsOSVersion%5C%22%3Afalse%2C%5C%22apkFilterRegEx%5C%22%3A%5C%22%5C%22%2C%5C%22invertAPKFilter%5C%22%3Afalse%2C%5C%22autoApkFilterByArch%5C%22%3Atrue%2C%5C%22appName%5C%22%3A%5C%22%5C%22%2C%5C%22shizukuPretendToBeGooglePlay%5C%22%3Afalse%2C%5C%22exemptFromBackgroundUpdates%5C%22%3Afalse%2C%5C%22skipUpdateNotifications%5C%22%3Afalse%2C%5C%22about%5C%22%3A%5C%22Lawnchair%20is%20a%20free%2C%20open-source%20home%20app%20for%20Android.%20(NOTE%3A%20This%20is%20the%20debug%20version%20of%20Lawnchair%2C%20for%20the%20beta%2Fstable%20versions%20see%20%5C%5C%5C%22Lawnchair%5C%5C%5C%22)%5C%22%7D%22%7D
[GitHub link]: https://github.com/LawnchairLauncher/lawnchair/releases/tag/nightly
