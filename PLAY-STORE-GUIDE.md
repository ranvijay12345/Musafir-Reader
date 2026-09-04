# 📱 Google Play Store — Step‑by‑Step Upload Guide (Musafir Reader)

This is a plain‑English, do‑this‑then‑that guide to put **Musafir Reader** on the Google
Play Store. No prior experience needed. Set aside about **2–3 hours** the first
time (plus Google's review wait, usually a few days).

> You will also want **[ADMOB-GUIDE.md](ADMOB-GUIDE.md)** for turning on real
> ads, and **[ADD-A-BOOK.md](ADD-A-BOOK.md)** for adding/managing books.

---

## ✅ What you need before you start

| Thing | Where to get it | Cost |
|-------|-----------------|------|
| A Google account | you already have one | Free |
| **Play Console developer account** | play.google.com/console → sign up | **$25 once** (lifetime) |
| Your app icon (logo) | your designer / Canva / AI | — |
| A few screenshots of the app | take them on your phone/emulator | — |
| A **privacy policy web page** | see §6 (required — we show ads) | Free |
| The signed app file (**.aab**) | you build it — see §4 | — |

---

## 1. Create your Play Console account (one time)

1. Go to **https://play.google.com/console** and sign in.
2. Choose **"Create a developer account"** → pick **Yourself** (personal) unless
   you have a registered company.
3. Pay the **one‑time $25** fee.
4. Fill in your name, address, and phone. Google may ask you to **verify your
   identity** with an ID document — do it now, it can take a day or two.

> You cannot publish until identity verification is approved, so start this first.

---

## 2. Make the app "yours" (identity)

The name and package are **already set** for you:

1. **App name** — `app/src/main/res/values/strings.xml` → `app_name` is
   **"Musafir Reader"**. Change it here if you ever want a different display name.
2. **Package name (permanent!)** — in `app/build.gradle.kts`, `applicationId` is
   **`com.musafir.reader`** (verified free on Google Play). This is your Play
   Store identity. ⚠️ You can **never change it after publishing**.
   *(The internal `namespace` stays `com.shelf.reader` — it's a code-only folder
   name, never shown to users, so it doesn't need to match.)*
3. **App icon (logo)** — right‑click the `res` folder →
   **New ▸ Image Asset ▸ Launcher Icons**, point it at your **512×512** logo,
   click through, Finish. This replaces the placeholder book icon.

---

## 3. Create your signing key (one time — DO NOT LOSE IT)

Every Play app must be signed with a key that is **the same forever**. Lose it
and you can never update your app again.

1. In Android Studio: **Build ▸ Generate Signed Bundle / APK ▸ Android App Bundle**.
2. Click **Create new…**, choose a path like `shelf-release.jks`, set strong
   passwords, fill the certificate fields, and Finish.
3. **Back up** the `.jks` file and passwords somewhere safe (password manager +
   a second location). Never commit them to git.
4. Create a file named `keystore.properties` in the `android/` folder:
   ```properties
   storeFile=shelf-release.jks
   storePassword=YOUR_STORE_PASSWORD
   keyAlias=YOUR_KEY_ALIAS
   keyPassword=YOUR_KEY_PASSWORD
   ```
   The project reads this automatically to sign release builds. This file is
   already git‑ignored — keep it that way.

> **Tip:** Also enable **Play App Signing** when Play Console offers it (it will).
> Google keeps a secure copy of your signing key as a safety net.

---

## 4. Build the release file (.aab)

The Play Store wants an **Android App Bundle** (`.aab`), not an APK.

- Easiest: **Build ▸ Generate Signed Bundle / APK ▸ Android App Bundle**, choose
  your key, pick **release**, Finish. Android Studio shows a link to the file.
- Or from a terminal in the `android/` folder: `./gradlew bundleRelease`
  → the file lands at `app/build/outputs/bundle/release/app-release.aab`.

That single `.aab` is what you upload.

---

## 5. Take screenshots + prepare store art

Play requires a few images (uploaded on the website, **not** inside the app):

| Asset | Size | Needed |
|-------|------|--------|
| **App icon** | 512×512 PNG | Yes |
| **Feature graphic** | 1024×500 PNG/JPG | Yes |
| **Phone screenshots** | 2–8 images, min 1080px on a side | Yes (min 2) |
| 7‑inch / 10‑inch tablet shots | optional | Recommended |

**How to screenshot:** run the app, open a book, use the phone's screenshot
button (or the emulator camera icon). Grab the library screen, a book page, and
the language toggle to show off the English/Hindi feature.

---

## 6. Create a privacy policy (required — we show ads)

Because the app uses **AdMob** (which collects an advertising ID), Google
**requires** a public privacy policy URL.

- Fastest free options: a free generator like **app-privacy-policy-generator.firebaseapp.com**,
  a GitHub Pages page, a Google Site, or a Notion public page.
- It must mention that you show **Google AdMob** ads and that Google may collect
  an advertising identifier. A short paragraph is fine.
- Copy the **public URL** — you'll paste it into Play Console twice (App content
  → Privacy policy, and the Data safety form).

---

## 7. Set up the app in Play Console

In **Play Console → Create app**:

1. **App name**, default language, **App** (not game), **Free**. Accept the
   declarations.
2. Left menu → **"Set up your app"** — work through each item:
   - **App access** — "All functionality available without special access"
     (unless you add a login — you haven't).
   - **Ads** — **Yes, my app contains ads** (you show AdMob ads).
   - **Content rating** — fill the questionnaire (a reading app is typically
     rated *Everyone*).
   - **Target audience** — choose your age groups (avoid "under 13" unless you
     intend a children's app, which adds strict rules).
   - **Data safety** — see §8.
   - **Privacy policy** — paste your URL from §6.
3. **Store listing** — app name, short description (max 80 chars), full
   description, then upload the **icon, feature graphic, and screenshots** from §5.

---

## 8. Fill the "Data safety" form

Musafir Reader itself keeps reading progress and settings **only on the device** and has
no login. The one thing that collects data is **AdMob**. Answer roughly:

- **Does your app collect or share user data?** → **Yes** (because of ads).
- Under data types, select **Device or other IDs** (the advertising ID).
- **Purpose:** Advertising or marketing.
- **Collected or shared?** → Shared (with Google/AdMob for advertising).
- When in doubt, follow Google's **AdMob "Data safety" guidance** page — it
  lists exactly what to tick for apps that use the Ads SDK.

---

## 9. Upload the .aab and roll out

1. Left menu → **Testing ▸ Internal testing** (recommended first) or
   **Production**.
2. **Create new release** → upload your `app-release.aab` from §4.
3. Add a short **release note** (e.g. "First release — bilingual reading app").
4. **Review release → Start rollout**.
5. For internal testing you get a link to install on your own device and check
   everything works. When happy, promote the same release to **Production**.

---

## 10. Wait for review, then go live

- Google reviews new apps — usually **a few days** (sometimes hours, sometimes
  longer for a brand‑new account).
- You'll get an email when it's live or if something needs fixing.
- **Ads:** real ads may show as blank at first. AdMob fully approves serving
  **after** the app is live with real content and a valid privacy policy — this
  is normal. See **ADMOB-GUIDE.md**.

---

## 🔁 Publishing an update later

1. In `app/build.gradle.kts`, bump **`versionCode`** (e.g. 1 → 2) and optionally
   `versionName` (e.g. "1.0" → "1.1"). Play rejects an upload with a used
   `versionCode`.
2. Rebuild the `.aab` (§4) with the **same signing key**.
3. Play Console → **Production → Create new release → upload → roll out**.

---

## ⚡ Quick checklist

- [ ] $25 developer account created + identity verified
- [ ] `applicationId` / `namespace` set to your final package
- [ ] App icon replaced with your logo
- [ ] Real AdMob **App ID** + 3 unit ids set (see ADMOB-GUIDE.md)
- [ ] Signing key created and **backed up**; `keystore.properties` in place
- [ ] `.aab` built (`bundleRelease`)
- [ ] Privacy policy hosted at a public URL
- [ ] Icon, feature graphic, 2+ screenshots ready
- [ ] Data safety, Ads, and content‑rating forms completed
- [ ] Release uploaded and rolled out
