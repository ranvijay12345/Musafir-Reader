# Adding a book to the Musafir Reader Android app

There are **two ways** to add or edit books. Pick based on whether you want to
avoid publishing an app update.

| Way | Edit what | Requires app update? | Best for |
|-----|-----------|----------------------|----------|
| **A. Remote catalog** (recommended) | a `books.json` hosted online | **No** — the app downloads it on launch | Adding/editing books any time after publishing |
| **B. Bundled catalog** | `app/src/main/assets/books.json` | Yes — rebuild & upload | The books that ship inside the APK / offline fallback |

Both files use the **exact same JSON shape** (documented below), so you can copy
between them freely and keep the website in sync too.

---

## Way A — Remote catalog (add books without republishing) ⭐

Once this is set up, adding a book is: **edit one file online → open the app →
it's there.** No Android Studio, no Play Store review, no waiting.

### One-time setup

1. **Host a `books.json` at a public HTTPS URL.** Easiest options:
   - **GitHub** (free): commit `books.json` to a repo and use its *raw* URL, e.g.
     `https://raw.githubusercontent.com/<you>/<repo>/main/books.json`
   - **Your website**: upload it next to your site, e.g.
     `https://yoursite.com/books.json`
   - **A storage bucket** (Google Cloud Storage, S3, Firebase Hosting, etc.) — any
     URL that returns the raw JSON over **HTTPS** works.
2. **Point the app at it.** In `app/build.gradle.kts`, set the `BOOKS_URL` field:
   ```kotlin
   buildConfigField("String", "BOOKS_URL", "\"https://raw.githubusercontent.com/you/repo/main/books.json\"")
   ```
3. Rebuild **once** and publish. That's the only app update you ever need for
   content.

### From then on

- Edit your hosted `books.json` (add/change/remove books, reorder them).
- The app fetches it **every launch**, caches it, and shows the new content.
  Users can also tap the **↻ refresh** button on the home screen to pull the
  latest immediately.
- **Offline & safety net:** the app shows the last downloaded copy when offline,
  and if it has never connected it uses the `books.json` **bundled in the APK**.
  A broken or unreachable URL never breaks the app — it just keeps the last good
  catalog.

> Keep the bundled `assets/books.json` reasonably current too — it's what a
> brand-new install sees before its first successful download.

If you leave `BOOKS_URL` as `""` (the default), remote loading is simply off and
the app runs entirely from the bundled catalog (Way B).

---

## Way B — Bundled catalog

Everything about the built-in books lives in **one file**:

```
app/src/main/assets/books.json
```

You do **not** touch any Kotlin code to add, edit, remove, or reorder books.
Edit the JSON, rebuild, and the new book appears. This is the same content
model as the website, so you can keep both in sync easily.

---

## The 4 steps

1. Open `app/src/main/assets/books.json`.
2. Copy one existing book object (everything inside one pair of `{ ... }` in the
   `"books"` list).
3. Paste it as a new entry in the list (comma-separated) and change the details.
4. Rebuild / re-run the app. Done. 🎉

> Tip: the order of books in the list is the order they appear on the home
> screen.

---

## The shape of a book

```jsonc
{
  "id": "atomic-habits",                 // UNIQUE, lowercase, dashes, no spaces
  "author": "James Clear",               // shared across both languages
  "cover": { "c1": "#3a6ea5", "c2": "#1b3a5c" },  // gradient: top color, bottom color
  "languages": {
    "en": {
      "title": "Atomic Habits",
      "blurb": "Tiny changes, remarkable results.",
      "image": "https://.../atomic-habits-en.jpg",
      "chapters": [
        {
          "title": "Chapter 1 — The Surprising Power of Small Habits",
          "text": "First paragraph goes here.\n\nSecond paragraph after a blank line.\n\nThird paragraph."
        }
      ]
    },
    "hi": {
      "title": "एटॉमिक हैबिट्स",
      "blurb": "छोटे बदलाव, असाधारण परिणाम।",
      "image": "https://.../atomic-habits-hi.jpg",
      "chapters": [
        {
          "title": "अध्याय 1 — छोटी आदतों की ताकत",
          "text": "पहला अनुच्छेद यहाँ।\n\nखाली पंक्ति के बाद दूसरा अनुच्छेद।"
        }
      ]
    }
  }
}
```

## The rules (follow these and nothing breaks)

- **`id` must be unique** and never change once readers start a book — progress
  is saved against it. Use lowercase with dashes: `"rich-dad-poor-dad"`, not
  `"Rich Dad"`.
- **`author` and `cover` are shared.** Put a separate optional `image` inside
  each language (`languages.en.image` and `languages.hi.image`). A legacy
  top-level `image` is still accepted as a fallback.
- **`languages` needs at least one of `en` / `hi`.** Include only `en` to make
  the book English-only; the language toggle simply won't offer Hindi for it.
  (The app does **not** translate for you — you supply the text for each
  language.)
- **Paragraphs are separated by `\n\n`** (a blank line) inside the `text`
  string. The app paginates into ~230-word pages automatically — you never
  paginate by hand.
- Because this is JSON, remember:
  - Wrap every string in **double quotes** `"..."`.
  - Escape double quotes inside text as `\"`, and newlines as `\n`.
  - Put a **comma between items**, but **no trailing comma** after the last one.

## Cover images

- **No image?** Leave out `"image"`. The app draws a nice gradient cover using
  `cover.c1` (top) and `cover.c2` (bottom) with the title on it.
- **Have an image?** Set `"image"` inside each language to a full `https://`
  URL. English and Hindi can therefore use different covers. It loads over the
  network (Coil) and automatically falls back to the gradient if the image is
  missing or the device is offline.
- GitHub page links containing `/blob/` are converted to raw file URLs
  automatically, though copying the file's **Raw** URL is still recommended.
- Bundled local images are also possible: drop a file in
  `app/src/main/assets/covers/` and reference it — ask if you want that wired up
  (it needs a one-line loader tweak).

## If a book doesn't show up

- The app skips any book with no readable content in any language.
- Check your JSON is valid — a missing comma or unescaped quote is the usual
  culprit. Paste the file into any "JSON validator" online if unsure, or watch
  Logcat in Android Studio for the parse error.
