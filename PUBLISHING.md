# Publishing Musafir Reader to Google Play + enabling ads

This is your end-to-end checklist to go from the project in this folder to a
live, monetized app on the Play Store. Work through it top to bottom. Nothing
here needs code changes beyond pasting in your own ids and building a release.

> **Ads terminology:** for **apps** you use **AdMob** (not AdSense — AdSense is
> for websites). AdMob and AdSense share a Google account, but an app's banner
> revenue flows through AdMob. This project is already wired for AdMob.

---

## 0. What I need from YOU (assets checklist)

You mentioned you'll provide the logo and anything else needed. Here's the full
list. Give me these and I'll drop them in; or you can add them yourself in
Android Studio's **Image Asset** tool.

| Asset | Size / format | Where it goes | Required? |
|---|---|---|---|
| **App icon** (your logo) | 512×512 PNG (and ideally a square vector/large PNG) | Replaces the placeholder book icon in `res/mipmap*` via `New ▸ Image Asset` | Yes |
| **Feature graphic** | 1024×500 PNG/JPG | Uploaded in Play Console (not in the app) | Yes |
| **Phone screenshots** | 2–8 images, min 1080px on a side | Uploaded in Play Console | Yes (min 2) |
| **App name & short/long description** | text | Play Console store listing | Yes |
| **Privacy policy URL** | a public web link | Play Console + Data Safety | Yes (because we show ads) |
| **AdMob App ID** | `ca-app-pub-XXXX~YYYY` | `AndroidManifest.xml` | Before ads work |
| **AdMob banner unit ID** | `ca-app-pub-XXXX/ZZZZ` | `app/build.gradle.kts` | Before ads work |
| **AdMob app-open unit ID** | `ca-app-pub-XXXX/ZZZZ` | `app/build.gradle.kts` | Before ads work |
| **AdMob interstitial unit ID** | `ca-app-pub-XXXX/ZZZZ` | `app/build.gradle.kts` | Before ads work |

> Until you replace them, the app uses Google's **official test** AdMob ids, so
> everything runs and shows test ads safely.

A placeholder app icon (a book on warm paper) is already included so the project
builds and looks finished. Replace it with your logo before launch.

---

## 1. Make it yours (identity)

- **Application ID** — already set to `com.musafir.reader` in
  `app/build.gradle.kts` (verified free on Google Play). This is your permanent
  Play Store identity — do NOT change it after publishing. The internal
  `namespace` stays `com.shelf.reader` (a code-only folder name, never shown to
  users, so it doesn't need to match).
- **App name** — `res/values/strings.xml` → `app_name` (set to "Musafir Reader").
- **Icon** — replace the placeholder via Android Studio → right-click `res` →
  `New ▸ Image Asset ▸ Launcher Icons` and point it at your 512×512 logo.

---

## 2. Set up AdMob (do this before your release build)

1. Create an account at **admob.google.com** and add your app (choose "Android",
   and you can link it to the Play listing later).
2. Copy your **App ID** (`ca-app-pub-…~…`) and paste it into
   `app/src/main/AndroidManifest.xml`, replacing the sample value in:
   ```xml
   <meta-data
       android:name="com.google.android.gms.ads.APPLICATION_ID"
       android:value="ca-app-pub-XXXXXXXXXXXXXXXX~YYYYYYYYYY" />
   ```
3. In AdMob create a **Banner** ad unit. Copy its unit id
   (`ca-app-pub-…/…`) into `app/build.gradle.kts`, replacing the value in:
   ```kotlin
   buildConfigField("String", "ADMOB_BANNER_UNIT_ID",
       "\"ca-app-pub-XXXXXXXXXXXXXXXX/ZZZZZZZZZZ\"")
   ```
   The app shows **three** ad formats, so create three ad units in AdMob and
   paste each one into `app/build.gradle.kts`:
   ```kotlin
   buildConfigField("String", "ADMOB_BANNER_UNIT_ID",       "\"…\"")  // Banner (bottom of reader)
   buildConfigField("String", "ADMOB_APP_OPEN_UNIT_ID",     "\"…\"")  // App Open (on foreground)
   buildConfigField("String", "ADMOB_INTERSTITIAL_UNIT_ID", "\"…\"")  // Interstitial (leaving a book)
   ```
   The **App Open** ad is deliberately infrequent (at most once every ~10 minutes,
   never on the first cold start) and the **Interstitial** has its own cooldown
   (~3 minutes), so users are never spammed.
4. Leave the debug/test behavior alone — the app automatically uses Google's
   test ids in debug builds, so you never click live ads on your own device.

> **Ad approval reality:** AdMob approves the app for serving real ads only
> after your app is **live on the Play Store** with real content and a valid
> privacy policy, and your AdMob account passes its policy/identity checks. So:
> publish first (steps 4–7), then ads get fully approved.

---

## 3. Create a signing key (one time)

Play requires a signed release. Create a keystore once and reuse it forever
(**back it up — losing it means you can't update your app**).

```bash
keytool -genkey -v -keystore shelf-release.jks \
  -keyalg RSA -keysize 2048 -validity 10000 -alias shelf
```

Then create `android/keystore.properties` (this project already reads it
automatically and is set up to ignore it from source control):

```properties
storeFile=/absolute/path/to/shelf-release.jks
storePassword=your-store-password
keyAlias=shelf
keyPassword=your-key-password
```

If this file is absent, release builds fall back to debug signing so the project
still compiles — but Play will **not** accept a debug-signed build.

---

## 4. Build the release bundle (AAB)

Google Play wants an **Android App Bundle** (`.aab`), not an APK:

```bash
cd android
./gradlew bundleRelease          # Windows: gradlew.bat bundleRelease
```

Output: `app/build/outputs/bundle/release/app-release.aab`. This build is
minified and resource-shrunk (R8) via the rules in `proguard-rules.pro`.

To sanity-check on a device first, you can also build a release APK:
```bash
./gradlew assembleRelease
```

---

## 5. Privacy policy (required — we serve ads)

Because the app uses AdMob (which collects an advertising identifier), you must
publish a privacy policy at a public URL and link it in Play Console.

A ready-to-use starter template is included: [`PRIVACY-POLICY.md`](PRIVACY-POLICY.md).
Fill in the bracketed blanks, host it somewhere public (GitHub Pages, your site,
or a free host), and use that URL.

---

## 6. Play Console: create and fill the listing

1. Pay the one-time **$25** Google Play developer registration (play.google.com/console).
2. **Create app** → name, default language, "App", "Free".
3. **Store listing:** short description, full description, your app icon,
   feature graphic (1024×500), and 2–8 phone screenshots.
4. **App content** (left menu) — complete every section:
   - **Privacy policy:** paste your URL.
   - **Ads:** answer **Yes, my app contains ads**.
   - **Data safety:** see section 7.
   - **Target audience / content rating:** fill the questionnaire.
5. **Production ▸ Create release** → upload `app-release.aab` → add release notes.

---

## 7. Data Safety form (what to declare)

Musafir Reader keeps reading progress **on the device** (DataStore) and does not run its
own analytics or accounts. The one thing that collects data is the **AdMob
SDK**. A safe, accurate baseline:

- **Does your app collect or share user data?** Yes (because of ads).
- **Data types:** *Device or other IDs* (the advertising ID), and typically
  *App activity* / *Approximate location* as used by the ads SDK. When in
  doubt, consult Google's AdMob "Data safety" guidance — it lists exactly what
  to declare for the ads SDK version.
- **Collected or shared?** Shared (with Google/AdMob for advertising).
- **Is it required or optional / is it for functionality or ads?** Ads.
- **Encrypted in transit?** Yes. **Can users request deletion?** Reading
  progress is local; uninstalling removes it.

> The app itself stores only your language choice, theme, font size, and
> per-book page position — all locally, never uploaded.

---

## 8. Review and go live

- First-time submissions can take a few days to review.
- After the app is **live**, return to AdMob; once its checks pass, real ads
  begin serving in the release build. Test ads will have been showing until then.

---

## Quick pre-launch checklist

- [ ] `applicationId` / `namespace` set to your final package
- [ ] App icon replaced with your logo
- [ ] Real AdMob **App ID** in `AndroidManifest.xml`
- [ ] Real AdMob **banner unit id** in `build.gradle.kts`
- [ ] Real AdMob **app-open unit id** in `build.gradle.kts`
- [ ] Real AdMob **interstitial unit id** in `build.gradle.kts`
- [ ] `keystore.properties` created and keystore backed up
- [ ] `./gradlew bundleRelease` produces `app-release.aab`
- [ ] Privacy policy hosted at a public URL
- [ ] Store listing assets ready (icon, feature graphic, screenshots, text)
- [ ] Data Safety, Ads, and content-rating forms completed
- [ ] `versionCode` bumped for each new upload (in `build.gradle.kts`)
