# App icon — Musafir Reader

The launcher icon is a **guiding star rising above an open book** on a warm
terracotta background — "musafir" means *traveler*, so the star is the
wayfarer's guide and the book is the journey. It uses the app's own palette
(terracotta `#C9773C`→`#9E4E1C`, cream pages `#FBF6EC`, gold star `#F4B23E`).

## What's already done (in the app)

The in-app icon is **pure vector** (crisp at every screen density) and is
already wired up — nothing to rebuild by hand:

- `app/src/main/res/drawable/ic_launcher_foreground.xml` — star + book
- `app/src/main/res/drawable/ic_launcher_background.xml` — terracotta gradient
- `app/src/main/res/mipmap-anydpi-v26/ic_launcher.xml` / `_round.xml` — adaptive icon (API 26+)
- `app/src/main/res/mipmap/ic_launcher.xml` / `_round.xml` — legacy icon (API 24–25)

Just run the app to see it.

## The 512×512 PNG for the Play Store listing

Google Play's **Store listing → App icon** field needs a **512×512 PNG**
(32-bit, no transparency, square — Google rounds the corners for you).

The exact same artwork is provided as a vector here: **`ic_launcher_512.svg`**.
Turn it into the required PNG with any **one** of these (all free, ~10 seconds):

**Option A — online (no install).** Go to `https://cloudconvert.com/svg-to-png`
(or `svgtopng.com`), upload `ic_launcher_512.svg`, set width & height to **512**,
convert, download. Done.

**Option B — Android Studio.** It renders the vectors directly, so you don't
even need the SVG: right-click `res` → **New ▸ Image Asset ▸ Launcher Icons**,
pick "Image" and point it at the star/book, or just screenshot the 512 preview.
(The vector icon above already ships in the app; this is only for the *store*
image file.)

**Option C — command line (if you have one of these installed):**
```bash
# ImageMagick
magick -background none -density 512 ic_launcher_512.svg -resize 512x512 ic_launcher_512.png
# or rsvg-convert
rsvg-convert -w 512 -h 512 ic_launcher_512.svg -o ic_launcher_512.png
# or Inkscape
inkscape ic_launcher_512.svg -w 512 -h 512 -o ic_launcher_512.png
```

> Note: this machine's shell was unavailable when the icon was created, so the
> PNG could not be generated here — use one of the options above. The SVG is
> pixel-exact for 512×512, so the output needs no cropping or resizing.

## Want a different look?

Every color and shape is plain text in the files above. Change `startColor`/
`endColor` for the background, `#F4B23E` for the star, or swap the star for
another motif — then re-export the 512 PNG the same way.
