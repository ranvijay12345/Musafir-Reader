# 💰 AdMob Setup Guide (Musafir Reader) — turn on real ads

Musafir Reader is already fully wired for **Google AdMob**. Until you paste in your own
ids, it safely shows **Google's test ads** (no risk to your account). This guide
gets you real, paying ads.

> **App ads = AdMob**, not AdSense. AdSense is for websites. They share one
> Google account, but your app's ad money flows through **AdMob**.

---

## What ads does Musafir Reader show?

| Format | Where it appears | Frequency |
|--------|------------------|-----------|
| **Banner** | small strip at the bottom of the library & reader | always visible |
| **App Open** | full‑screen when you return to the app | **at most once every ~10 minutes**, never on first launch |
| **Interstitial** | full‑screen when you leave a book | **3‑minute cooldown** between shows |

The frequency caps are deliberate so users are **never spammed** — this keeps
your ratings high and keeps AdMob happy.

---

## Step 1 — Create your AdMob account

1. Go to **https://admob.google.com** and sign in with your Google account.
2. Accept the terms, set your country and timezone.
3. Add your **payment details** (so Google can pay you) — you can do this now or
   after you hit the payment threshold.

---

## Step 2 — Add your app in AdMob

1. AdMob → **Apps ▸ Add app**.
2. Platform: **Android**.
3. "Is your app listed on Google Play?" — if it's live, say **Yes** and search
   for it; if not yet, say **No** and add it later. Either is fine.
4. AdMob gives you an **App ID** that looks like:
   `ca-app-pub-1234567890123456~1234567890`
   (note the **`~`**). Copy it.

---

## Step 3 — Create THREE ad units

In AdMob → your app → **Ad units ▸ Add ad unit**. Create one of each:

| Ad unit type | Name it | You get an id like |
|--------------|---------|--------------------|
| **Banner** | "Musafir Reader Banner" | `ca-app-pub-…/1111111111` |
| **App Open** | "Musafir Reader App Open" | `ca-app-pub-…/2222222222` |
| **Interstitial** | "Musafir Reader Interstitial" | `ca-app-pub-…/3333333333` |

Each **unit id** has a **`/`** in it (the App ID has a `~`). Copy all three.

---

## Step 4 — Paste your ids into the project (only 4 changes)

### 4a. App ID → `AndroidManifest.xml`

Open `app/src/main/AndroidManifest.xml` and replace the sample value:

```xml
<meta-data
    android:name="com.google.android.gms.ads.APPLICATION_ID"
    android:value="ca-app-pub-XXXXXXXXXXXXXXXX~YYYYYYYYYY" />
```

### 4b. The three unit ids → `app/build.gradle.kts`

Find these three lines and paste your unit ids between the quotes:

```kotlin
buildConfigField("String", "ADMOB_BANNER_UNIT_ID",       "\"ca-app-pub-…/1111111111\"")  // Banner
buildConfigField("String", "ADMOB_APP_OPEN_UNIT_ID",     "\"ca-app-pub-…/2222222222\"")  // App Open
buildConfigField("String", "ADMOB_INTERSTITIAL_UNIT_ID", "\"ca-app-pub-…/3333333333\"")  // Interstitial
```

**That's it — those 4 values are the only things you change.** Everything else
(loading, showing, frequency caps, test‑vs‑real switching) is already coded.

---

## Step 5 — How test vs. real is chosen (nothing to do)

The app is smart about this automatically:

- **Debug builds** (running from Android Studio) → always **Google test ads**.
  You can tap them safely; it never touches your real account.
- **Release builds** (the `.aab` you upload) → your **real** ids from step 4.

So you never accidentally click your own live ads during development.

---

## Step 6 — Why real ads look blank at first (this is normal)

After you publish, real ads may show **blank for a while**. AdMob only turns on
real ad serving once:

1. Your app is **live on the Play Store** with real content, **and**
2. You have a valid **privacy policy** linked (see PLAY-STORE-GUIDE.md §6), **and**
3. Your AdMob account passes its **policy & identity checks**.

This can take a few hours to a couple of days after going live. Until then
you'll see blanks or test‑style ads — **don't panic, don't change code.**

---

## Step 7 — Link AdMob ↔ Play (recommended)

Once the app is live: AdMob → your app → **App settings ▸ link to Google Play**.
This improves reporting and helps ad targeting/earnings.

---

## ⚡ AdMob quick checklist

- [ ] AdMob account created + payment details added
- [ ] App added in AdMob → got the **App ID** (`~`)
- [ ] Created **Banner**, **App Open**, **Interstitial** units (each `/`)
- [ ] App ID pasted into `AndroidManifest.xml`
- [ ] 3 unit ids pasted into `app/build.gradle.kts`
- [ ] Rebuilt the release `.aab`
- [ ] App live + privacy policy linked → wait for real ads to activate
