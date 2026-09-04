# Musafir Reader — Android Book Reader

A native Android reading app (Kotlin + Jetpack Compose) that mirrors the Shelf
website: a library of books, a clean paginated reader (~230 words per page),
bilingual **English / हिंदी** support with per-language reading progress, a
warm "paper" theme with light/dark mode, and an AdMob banner ready for
monetization after Play Store approval.

> **New to the project? Read this file top to bottom once.** It tells you how
> to open, build, add books, and publish. Two companion files go deeper:
> [`ADD-A-BOOK.md`](ADD-A-BOOK.md) and [`PUBLISHING.md`](PUBLISHING.md).

---

## What you need installed

- **Android Studio** (Ladybug 2024.2 or newer) — this is the easiest path and
  handles the Android SDK, JDK, and emulator for you.
- That's it. Android Studio bundles a JDK 17 and downloads the Gradle version
  this project expects (8.9).

---

## First open (IMPORTANT — one-time wrapper step)

This project ships every source file **except one binary**: the Gradle wrapper
JAR (`gradle/wrapper/gradle-wrapper.jar`). Binaries can't be authored as text,
so you generate it once — it takes seconds:

**Option A — just open in Android Studio (recommended).**
1. `File ▸ Open` and select the `android/` folder.
2. Android Studio detects the missing wrapper and offers to set it up, or it
   uses its own bundled Gradle. Let it sync. Done.

**Option B — command line (if you have Gradle installed globally).**
```bash
cd android
gradle wrapper --gradle-version 8.9
./gradlew assembleDebug        # macOS/Linux
gradlew.bat assembleDebug      # Windows
```

After the wrapper JAR exists, `./gradlew` works normally for every future build.

---

## Run it

- In Android Studio: pick a device/emulator and press **Run** ▶.
- Command line: `./gradlew installDebug` then launch **Musafir Reader** on the device.

The debug build uses Google's official **test** AdMob ids, so you'll see test
ads and never risk your account while developing.

---

## Project layout

```
android/
├─ app/
│  ├─ src/main/
│  │  ├─ assets/books.json          ← YOUR BOOKS live here (edit this to add books)
│  │  ├─ java/com/shelf/reader/
│  │  │  ├─ MainActivity.kt          app entry, sets up Compose + theme
│  │  │  ├─ ShelfApp.kt              initializes AdMob
│  │  │  ├─ data/                    Models, BookRepository (pagination), SettingsStore (DataStore)
│  │  │  └─ ui/
│  │  │     ├─ ShelfNavHost.kt       navigation (Home / Reader / About)
│  │  │     ├─ ShelfViewModel.kt     app state
│  │  │     ├─ UiStrings.kt          bilingual UI labels
│  │  │     ├─ theme/                colors, typography, light/dark theme
│  │  │     ├─ components/           BookCover, AdBanner
│  │  │     └─ screens/              HomeScreen, ReaderScreen, AboutScreen
│  │  └─ res/                        icons, themes, strings, colors
│  ├─ build.gradle.kts               app module config (SDK, signing, AdMob ids)
│  └─ proguard-rules.pro
├─ build.gradle.kts, settings.gradle.kts, gradle.properties
└─ gradle/libs.versions.toml         dependency versions (version catalog)
```

## Key design choices

- **Content lives in one file.** `assets/books.json` uses the same shape as the
  website's `js/data.js`. Adding a book is editing JSON — no code changes. See
  [`ADD-A-BOOK.md`](ADD-A-BOOK.md).
- **Pagination matches the website.** `BookRepository.buildPages()` splits
  chapters into ~230-word pages and never breaks a paragraph.
- **Progress is per book, per language.** Stored in DataStore under
  `progress::<bookId>::<lang>`, so English and Hindi positions never collide.
- **No Material You dynamic color.** The warm paper palette is intentional and
  consistent on every device.
- **Devanagari** renders through the system's bundled Noto Serif Devanagari
  fallback — no font binaries bundled.

## Before you publish

Monetization and Play Store steps (real AdMob ids, signing keystore, privacy
policy, Data Safety form, screenshots) are all in [`PUBLISHING.md`](PUBLISHING.md).
The app is wired for AdMob but ships with **test** ids — swap them in before you
upload a release.
