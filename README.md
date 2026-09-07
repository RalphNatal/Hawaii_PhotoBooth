# Hawaii Booths — landing page

A single-file marketing landing page for Hawaii Booths, a photo / video / audio
booth rental company in Honolulu. It includes a working in-browser booth: the
visitor starts their own webcam, tries on one of four overlay frames, takes a
photo, and watches it print out of an illustrated printer as they scroll.

Everything lives in [`index.html`](index.html) — markup, styles, and script.
There is **no build step, no `npm install`, and no bundler**. Tailwind comes
from a CDN and the fonts come from Google Fonts. Nothing else is fetched at
runtime.

---

## Running it locally

**The camera will not work if you open the file by double-clicking it.**
`navigator.mediaDevices.getUserMedia` only runs in a *secure context*, and
`file://` is not one. Chrome fails this silently — no error, no permission
prompt, just a preview box that never fills. You need a local server.
`localhost` counts as secure.

### Option A — VS Code Live Server (no Node required)

1. Extensions panel (`Ctrl+Shift+X`) → search **Live Server** → Install.
2. Right-click `index.html` in the Explorer → **Open with Live Server**.

### Option B — Node

```bash
npx serve .          # then open http://localhost:3000
```

Either way, edit the file and save; the page reloads. You need a network
connection on first load for the fonts, Tailwind, and the placeholder images.

---

## Swapping in the real assets

Two config blocks at the top of the `<script>` in `index.html` control every
replaceable asset. **Nothing else in the file needs to be touched.**

### Images — the `ASSETS` object

```js
const ASSETS = {
  hero:  'https://picsum.photos/seed/hawaii-hero/1200/1500',
  about: '...',
  services: { photo: '...', video: '...', audio: '...', custom: '...' },
  gallery: [ /* 8 entries: { src, w, h, alt } */ ],
};
```

Drop the real photos into `assets/img/` and point the paths at them:

```js
hero: 'assets/img/hero.jpg',
```

Then **update the `alt` text** on each image to describe the real photo. The
gallery entries carry their own `alt`, `w`, and `h` — keep `w` and `h` accurate,
they reserve the space so the page does not jump while images load, and keep
the heights varied or the masonry columns flatten out.

Placeholders use `picsum.photos`, which is keyless and deterministic per seed.
**Do not use `source.unsplash.com`** — that endpoint was retired and returns 503
for every request.

### Frames — the `FRAMES` array

The four frames are drawn as inline SVG data URIs, so the page has no external
asset dependencies out of the box. To use real image files instead:

```js
{ id: 'classic', name: 'Classic White', src: 'assets/frames/classic.png' }
```

See [`assets/frames/README.md`](assets/frames/README.md) for the exact file
spec. The short version: transparent centre, 4:5, at least 1080 × 1350, PNG,
and **served from this same origin** — a cross-origin frame taints the canvas
and breaks "Save photo" entirely.

### Print size

```js
const OUTPUT_W = 1080;
const OUTPUT_H = 1350;   // 4:5
```

Change both together to match the real booth's print size, and re-author the
frames to the same ratio if you do.

---

## Paths must stay relative — this breaks the live site if ignored

GitHub Pages serves this project from `/Hawaii_PhotoBooth/`, **not** the domain
root. A leading slash resolves against the root instead:

```html
<img src="assets/img/hero.jpg">     <!-- correct -->
<img src="/assets/img/hero.jpg">    <!-- works locally, 404s in production -->
```

That is the worst kind of bug, because local testing never catches it. The same
rule applies to the frame paths in `FRAMES` and to any anchor `href`. To check
before you push:

```bash
grep -n 'src="/\|href="/' index.html     # should print nothing
```

---

## Deploying to GitHub Pages

Once the branch is merged to `main`:

1. **Settings → Pages**
2. **Source:** Deploy from a branch
3. **Branch:** `main`, folder `/root` → **Save**

The site lands at:

```
https://ralphnatal.github.io/Hawaii_PhotoBooth/
```

That URL is HTTPS, so the camera works there. **This is the link to send the
client** — a local copy opened over `file://` will not demo correctly.

---

## Before this goes live

Search `index.html` for `TODO` — these are the things that still need real
content or wiring:

- **The contact form does not submit anywhere.** It validates and shows an
  inline confirmation. The `TODO` in the `bookingForm` function marks exactly
  where to POST, with three options written out (Formspree is the one that
  works on GitHub Pages).
- **The two testimonial quotes are placeholder copy.** Replace them with real,
  attributable quotes from actual clients.
- **The footer social links** point at the bare `instagram.com`, `facebook.com`,
  and `tiktok.com` domains. Point them at the real profiles.
- **All photography is placeholder.** See the `ASSETS` section above.

Copy, package prices, and the contact details were taken from
`hawaiibooths.com`. Prices are shown as "from $" and are worth re-checking
before launch.

---

## Notes for whoever picks this up next

- **Tailwind is loaded from the Play CDN, which is a development tool.** It
  compiles CSS in the visitor's browser on every page load. Before real
  production traffic, compile it ahead of time
  (`npx tailwindcss -i src/input.css -o assets/tailwind.css --minify`), delete
  the two CDN script tags, and link the built file. The token config moves into
  `tailwind.config.js` unchanged. There is a comment saying this in the file.
- **The camera is never requested on page load** — only from the "Start camera"
  button. It is released when the tab is hidden, when the page unloads, and
  when the booth section scrolls out of view, so the camera light does not stay
  on.
- **Nothing is uploaded.** Capture happens on a canvas in the browser and the
  saved file comes from a local object URL. There is no server.
- **Motion is deliberately limited** to the printer reveal. If you are adding
  sections, resist adding entrance animations to them — the restraint is the
  design.
- `prefers-reduced-motion: reduce` disables the capture flash and the scroll
  animation, and the printed photo snaps straight to its finished position.
