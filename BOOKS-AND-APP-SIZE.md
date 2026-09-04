# 📦 Books, Covers & App Size (Musafir Reader)

Short answer to *"if I add 50 books, will the app get huge?"* → **No, not if you
do it the recommended way.** This explains why, and how to add book covers.

> How to actually add a book is in **[ADD-A-BOOK.md](ADD-A-BOOK.md)**. This doc
> is about **size** and **cover images**.

---

## What actually takes up space

Musafir Reader stores each book as **text inside a JSON file** (`books.json`). Text is
tiny. A full-length book of plain text is roughly **0.5–1 MB**; most "books"
here are chapter samples and are far smaller.

| What | Size impact |
|------|-------------|
| Book **text** (in `books.json`) | small — a few KB to ~1 MB per full book |
| Book **cover gradient** (`cover.c1`/`c2`) | **zero** — drawn by code, not a file |
| Book **cover image** (`image` URL) | **zero in the app** — loaded from the internet when shown |
| The app code + AdMob + Compose | fixed ~8–12 MB regardless of book count |

**Covers never bloat the app.** They're either (a) a two-color gradient the app
paints at runtime, or (b) a photo loaded from a URL over the internet (via Coil)
and cached on the device — neither is packed into the app file.

---

## So what happens with 50 books?

It depends **only on where the text lives**:

### Option A — Remote catalog (recommended) → app size barely changes
Host `books.json` online and set `BOOKS_URL` (see ADD-A-BOOK.md, "Way A"). The
50 books' text is **downloaded**, not shipped inside the app. Your `.aab` stays
around its base ~8–12 MB no matter how many books you add. You can add book #51,
#100, #500 **without ever republishing the app**.

### Option B — Bundled catalog → app grows by the text size only
If the 50 books ride **inside** the app (`assets/books.json`), the app grows by
the combined **text** size. Realistically:

| Books bundled | Rough text add | Resulting app size |
|---------------|----------------|--------------------|
| 4 (today) | ~1 MB | ~10 MB |
| 50 short samples | ~5–15 MB | ~15–25 MB |
| 50 full-length books | ~30–50 MB | ~40–60 MB |

Even the heavy case is a normal app size — but **remote (Option A) is better**:
smaller download, instant updates, no Play review wait.

> **Best of both:** keep a handful of books bundled (so a fresh install shows
> content instantly, even offline) and host the full 50 remotely. The app merges
> to the remote list on first launch.

---

## How to add a book cover ("banner")

You have three choices per book, set in that book's entry in `books.json`:

### 1. Gradient only (default, zero setup)
Leave out `image`. The app paints a nice gradient using the two colors and draws
the title on it:
```jsonc
"cover": { "c1": "#3a6ea5", "c2": "#1b3a5c" }
```
Pick any two hex colors (top, bottom). Great when you don't have artwork.

### 2. A real cover photo from a URL (recommended for real covers)
Host the image anywhere public (your site, GitHub, a storage bucket, even an
Open Library / book-cover URL) and point `image` at it:
```jsonc
"cover": { "c1": "#3a6ea5", "c2": "#1b3a5c" },   // still used as a fallback
"image": "https://example.com/covers/atomic-habits.jpg"
```
- Loads over the internet, cached after first view.
- **Automatically falls back to the gradient** if the URL is broken or the phone
  is offline — so it never shows an empty box.
- Recommended image: portrait, around **600×900 px**, JPG or PNG, under ~200 KB.

### 3. A bundled local image (only if you want it inside the app)
Drop the file in `app/src/main/assets/covers/` and reference it. This needs a
**one-line loader tweak** — ask and it can be wired up. Note this *does* add the
image's size to the app, so prefer option 2 for lots of covers.

---

## Step-by-step: add books + covers the recommended way

1. **Get artwork** (optional) — one portrait image per book, ~600×900 px each.
2. **Host your images** somewhere public (get an `https://…` URL for each).
3. **Host `books.json`** online and set `BOOKS_URL` once (ADD-A-BOOK.md "Way A").
4. For each book, add an entry to the hosted `books.json` with its `id`,
   `author`, `cover` colors, optional `image` URL, and the `en`/`hi` text.
5. Save the hosted file → open the app → tap **↻ refresh** on the home screen.
   The new books and covers appear. **No app rebuild, no Play review.**

That's the whole loop. Adding 50 books is just 50 entries in one online file,
with the app size essentially unchanged.

---

## Quick answers

- **Will 50 books make the app huge?** Only if bundled; host remotely and the app
  stays small.
- **Do covers increase app size?** No — gradients are code; image covers load
  from the internet.
- **Fastest way to add many books?** Remote `books.json` + image URLs, then
  refresh in-app. No republishing.
- **What if an image URL breaks?** The app shows the gradient cover instead —
  nothing looks broken.

---

## Where do I host `books.json` and the images? (GitHub works great) ⭐

**Yes, GitHub works** — it's the easiest free option. You need two kinds of
public HTTPS links: one for `books.json`, and one per cover image. Anything that
serves the raw file over `https://` works (GitHub, your website, Google Cloud
Storage / S3 / Firebase Hosting, Cloudflare R2, etc.). GitHub is recommended
because it's free, versioned, and needs no server.

### Option 1 — GitHub repo "raw" links (simplest)

1. Create a **public** repo, e.g. `shelf-content`.
2. Put your files in it:
   ```
   shelf-content/
     books.json
     covers/
       atomic-habits.jpg
       the-alchemist.jpg
   ```
3. The **raw URL** of any file is:
   `https://raw.githubusercontent.com/<user>/<repo>/main/<path>`
   e.g. `https://raw.githubusercontent.com/you/shelf-content/main/books.json`
   and `https://raw.githubusercontent.com/you/shelf-content/main/covers/atomic-habits.jpg`
4. In `app/build.gradle.kts`, set the catalog URL once:
   ```kotlin
   buildConfigField("String", "BOOKS_URL",
       "\"https://raw.githubusercontent.com/you/shelf-content/main/books.json\"")
   ```
5. In `books.json`, each book's `image` points at its raw cover URL:
   ```jsonc
   "image": "https://raw.githubusercontent.com/you/shelf-content/main/covers/atomic-habits.jpg"
   ```

> Tip: on the file's GitHub page, click **"Raw"** and copy the address bar — that
> is the exact URL to use.

### Option 2 — GitHub Pages (nicer URLs)

Enable **Pages** on the repo and your files are served at
`https://<user>.github.io/<repo>/books.json` and
`.../covers/atomic-habits.jpg`. Works identically; just prettier links.

> **Same file, both apps:** this is the *same* `books.json` shape the website
> uses, so one hosted file can feed both the Android app and the web app.

---

## Does it re-download every time? No — it's cached. ✅

You asked for the books to be downloaded once and then just *be there*. That's
exactly how it works now:

### Book text (`books.json`) — cached on disk
- On **refresh** (launch, or the ↻ button) the app downloads `books.json` from
  your GitHub URL and **saves it to internal storage**
  (`filesDir/books_cache.json`).
- On **every later launch** the app paints instantly from that saved copy — **no
  network needed** — then quietly checks GitHub once in the background for any
  changes. Offline? It keeps showing the cached books.
- It only replaces the cache when a newer download actually parses to real
  books, so a bad/unreachable URL never wipes your library.

### Cover images — cached on disk (now with a big, durable cache)
- Covers load from their URL **once**, then are stored in a dedicated on-disk
  image cache (`cacheDir/book_covers`, up to **250 MB** — enough for hundreds of
  covers). After that first view they display straight from disk on every later
  launch, **without re-downloading**, and the cache **survives closing the app**.
- Because covers are treated as immutable, the app won't re-fetch them over the
  network to "re-check" — it trusts the cached copy. **To change a cover, use a
  new image URL** (e.g. `atomic-habits-v2.jpg`) so the app fetches the new one.
- If a cover ever isn't cached and the phone is offline, the book still shows its
  gradient fallback — never a blank box.

**Net effect:** first launch with internet pulls text + covers once; from then
on the app opens instantly from its caches and only downloads again when you
actually change the hosted content.
