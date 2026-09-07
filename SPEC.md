# Hawaii Booths — landing page brief

> Committed as-is for future reference. This is the brief the page was built
> from; where the finished page departs from it, the departure is noted in the
> "Deviations" section at the end.

You are the front-end lead building a marketing landing page for Hawaii Booths,
a premium photo / video / audio booth rental company serving weddings,
corporate events, birthdays, and celebrations. The client has already rejected
templated, generic-looking proposals. Every visual choice must be deliberate
and specific to this brief.

## Repository

Work in https://github.com/RalphNatal/Hawaii_PhotoBooth. It currently holds one
commit and a placeholder README — treat it as a clean slate, but do not
force-push or rewrite that initial commit.

```bash
git clone https://github.com/RalphNatal/Hawaii_PhotoBooth.git
cd Hawaii_PhotoBooth
git checkout -b feat/landing-page
```

Build on `feat/landing-page`, not directly on `main`. Merge to `main` via PR
when the acceptance checklist passes — that keeps `main` deployable at all
times and gives the client a single diff to review.

Commits: one per build phase (see section 8), conventional-commit style,
present tense:

```
feat: design tokens, sticky nav, and hero section
feat: services, pricing, gallery, and contact sections
feat: webcam capture with selectable overlay frames
feat: scroll-linked photo printing animation
docs: setup and asset-swap instructions
```

Never commit a captured photo, a `.env`, or anything from the camera. Add
`.gitignore`:

```
.DS_Store
Thumbs.db
.vscode/
node_modules/
*.log
captures/
```

## Deliverable

Create these files. No build step, no `npm install`, no bundler.

```
index.html                  ← entire page: markup + inline <script> + minimal <style>
assets/frames/README.md     ← spec for replacement frame files (dimensions, transparency)
assets/img/                 ← empty; where the client's real photos land
README.md                   ← replaces the current placeholder: what this is, how to run
                              it locally, how to swap placeholder assets, how to deploy
.gitignore
SPEC.md                     ← this file, committed as-is for future reference
```

Everything ships as one self-contained `index.html`. Tailwind comes from CDN.
Fonts come from Google Fonts. No other runtime dependencies unless explicitly
listed below.

### How it must be run (state this in README.md)

`navigator.mediaDevices.getUserMedia` only works in a secure context. Opening
`index.html` via `file://` will fail silently in Chrome. The README must
instruct:

```bash
npx serve .          # then open http://localhost:3000
# or: VS Code "Live Server" extension → Open with Live Server
```

`localhost` counts as secure. Production must be HTTPS.

### Deployment: GitHub Pages

Once merged to `main`, enable Pages (Settings → Pages → Source: Deploy from a
branch → `main` → `/root`). The site lands at:

```
https://ralphnatal.github.io/Hawaii_PhotoBooth/
```

That URL is HTTPS, so the camera works there — which makes it the link to send
the client for review. A local `file://` copy will not demo correctly.

**Path constraint — this breaks Pages if ignored.** A GitHub Pages project site
is served from the `/Hawaii_PhotoBooth/` subpath, not the domain root. Every
internal reference must therefore be relative:

```html
<img src="assets/img/hero.jpg">     <!-- correct -->
<img src="/assets/img/hero.jpg">    <!-- 404s on Pages, works locally — the worst kind of bug -->
```

Same rule for the frame PNGs in `FRAMES` and for any anchor `href`s. Document
this in README.md so it survives the handoff.

## 1. Design system

### Tailwind setup

Use the Tailwind v3 Play CDN with an inline config so custom tokens work as
utility classes:

```html
<script src="https://cdn.tailwindcss.com"></script>
<script>
  tailwind.config = {
    theme: {
      extend: {
        colors: {
          ink:   '#062A3A',  // deepest ocean — body text, dark sections
          ocean: '#0E4D63',  // primary brand blue
          surf:  '#2E8B9E',  // mid-tone accent, hovers, focus rings
          sand:  '#EADFC8',  // warm sand — soft section backgrounds
          gold:  '#C4922F',  // warm gold — CTA, rules, small accents only
          shell: '#FDFCFA',  // crisp off-white page background
        },
        fontFamily: {
          display: ['"Playfair Display"', 'Georgia', 'serif'],
          sans:    ['Inter', 'system-ui', 'sans-serif'],
        },
      },
    },
  }
</script>
```

Add a code comment noting the Play CDN is dev-only, and that production should
compile Tailwind properly (link to a one-line note about `npx tailwindcss` CLI).

### Typography

- **Playfair Display** — headings only. Weights 400/600. Tight leading
  (`leading-[1.1]`), generous letter-spacing at large sizes is wrong here —
  keep tracking normal to slightly tight.
- **Inter** — all body, nav, buttons, labels. Weights 400/500/600.
- Load with `<link rel="preconnect">` + a single `display=swap` Google Fonts
  request. Subset to the weights actually used.
- Body copy line length under 75 characters (`max-w-prose` or explicit
  `max-w-[62ch]`).

### Anti-generic guardrails (these matter — do not skip)

Do not produce:

- ALL-CAPS tracked-out eyebrow labels above every section heading.
- One word of each headline colored gold or italicized for "emphasis."
- Numbered markers (01 / 02 / 03) on the services grid — services are not a
  sequence. Numbering is correct on the "How it works" steps, because those
  genuinely are ordered.
- Identical rounded cards with the same `shadow-lg` under every one, regardless
  of hierarchy.
- Fade-and-slide-up entrance animations on every section. Motion budget for
  this page is spent almost entirely on the printer reveal. Everything else
  stays still.
- `→` appended to button and link text.
- Gradient washes used as decoration.

Do use:

- **Restraint.** Deep ocean and white carry the page; gold appears in small
  amounts (CTA fill, a hairline rule, the active frame indicator) and nowhere
  else.
- **Generous whitespace.** Section padding `py-24 md:py-32`.
- **One memorable moment:** the Try It Out section. Everything around it is
  disciplined and quiet.

### Motion & accessibility floor

- Respect `prefers-reduced-motion: reduce` — when set, the printer animation
  snaps to its final state instead of tracking scroll, and no transitions run.
- Visible keyboard focus everywhere: `focus-visible:ring-2
  focus-visible:ring-surf focus-visible:ring-offset-2`.
- All interactive controls are real `<button>` / `<a>` elements. Frame selector
  uses `role="radiogroup"` with arrow-key navigation and `aria-checked`.
- Colour contrast ≥ 4.5:1 for body text. Gold on white fails — never put gold
  text on white; gold is a fill or a rule, and text on gold is ink.
- Every image has meaningful `alt`; decorative SVG gets `aria-hidden="true"`.

## 2. Page sections

Mirror the real site's information architecture (from hawaiibooths.com),
upgraded.

### Navbar

- Sticky, transparent over the hero, transitions to `bg-shell/95 backdrop-blur
  border-b border-ink/10` once `window.scrollY > 40`. Use a passive scroll
  listener + a class toggle, not a per-frame style write.
- Wordmark: "HAWAII BOOTHS" in Playfair, ink. Small hairline rule beneath it,
  gold, 24px wide.
- Links: Home, About, Services, Pricing, Gallery, Contact.
- "Services" is a dropdown (hover on desktop, tap on mobile, Escape closes,
  focus trapped while open): Photo Booths, Video Booths, Audio Booths, Other
  Services.
- CTA button: "Book your date" — solid gold, ink text, `rounded-full px-6
  py-2.5`.
- Mobile: hamburger → full-screen overlay panel. `aria-expanded` on the toggle.
  Body scroll locked while open.

### Hero

- Full-viewport-height minus nav. Split layout: copy left (55%), image right
  (45%) on desktop; stacked on mobile with the image as a bleed-to-edge band.
- Headline: **Capture the moment, keep the memory** — Playfair, `text-5xl
  md:text-7xl`, ink.
- Subhead: Premium photo, video, and audio booth rentals for weddings,
  corporate events, birthdays, and celebrations across the islands.
  Professional prints in seconds, digital sharing instantly.
- Two CTAs: primary "Check your date" (gold), secondary "See how it works"
  (ghost button with ink border) that smooth-scrolls to the Try It Out section.
- A small trust row beneath: three short facts (e.g. "Booked 400+ events",
  "Unlimited prints on every package", "Attendant included"). Plain text with
  hairline dividers — no icon-badge treatment.

### Services grid

Four cards: Photo Booths, Video Booths, Audio Booths, Custom Enhancements.

Give the cards a hierarchy instead of four identical boxes: Photo Booths is the
flagship — make it span two columns on desktop with a larger image and one
extra line of copy. The other three are equal, smaller. Grid: `md:grid-cols-2
lg:grid-cols-3` with the flagship at `lg:col-span-2`.

Each card: image, name, one sentence of plain-language copy, and a text link
("Photo booth packages"). No hover-lift on all four — a subtle image scale on
hover is enough.

### How it works

Four numbered steps (numbering is justified here): Pick your date → Choose your
package → We set up and run it → Everyone goes home with prints. Horizontal on
desktop with a connecting hairline rule, vertical stack on mobile.

### Try it out — the centrepiece

Full spec in section 3 below.

### Pricing

Three packages — Classic / Signature / Island (2hr / 3hr / 4hr). Signature is
highlighted with an ocean background and white text, not with a "Most popular"
ribbon. Each lists 5–6 inclusions with a simple check glyph. Prices shown as
"from $X" with a note that final quotes depend on date and location. CTA per
card: "Request this package".

### Gallery

Masonry-ish grid, 8 images, `columns-2 md:columns-3` with `break-inside-avoid`.
Clicking one opens a lightweight lightbox (no library): fixed overlay, Escape
and backdrop click to close, arrow keys to move, focus returned to the trigger
on close.

### Testimonials

Two or three short quotes with name + event type. Serif, large, no
quotation-mark decoration graphics.

### Contact / booking

Two columns: a form (name, email, event date, event type select, message) and
contact details. The form does not submit anywhere — wire `onsubmit` to
`preventDefault()` and show an inline success state, with a `TODO:` comment
marking exactly where to POST to a real endpoint (Formspree / Netlify Forms /
custom API).

### Footer

Wordmark, nav column, service column, socials, copyright. `ink` background,
sand-tinted text.

## 3. The "Try it out" experience

This is the section the client is paying for. Build it carefully.

### Layout

```
┌─────────────────────────────────────────────────┐
│  Heading + one line of copy                     │
│                                                 │
│  ┌───────────────┐   ┌────────────────────┐     │
│  │               │   │  Choose a frame    │     │
│  │  LIVE PREVIEW │   │  [ ][ ][ ][ ]      │     │
│  │  (video +     │   │                    │     │
│  │   frame       │   │  [ Start camera ]  │     │
│  │   overlay)    │   │  [ Take photo   ]  │     │
│  │               │   │  [ Retake ] [Save] │     │
│  └───────────────┘   └────────────────────┘     │
│                                                 │
│           ── printer graphic, sticky ──         │
└─────────────────────────────────────────────────┘
```

### Frame selection

Four frames, defined in a single config array at the top of the script (see
section 5). Each frame is a transparent-centre PNG-equivalent, authored as an
inline SVG data URI so the page has zero external asset dependencies out of the
box:

- **Classic White** — thick white border, deep bottom bar (polaroid
  proportions), small "HAWAII BOOTHS" wordmark in the bottom bar.
- **Gold Leaf** — double gold hairline inset with small corner flourishes,
  otherwise transparent.
- **Ocean Deco** — ink art-deco corner brackets with stepped geometry, thin
  surf-coloured inner rule.
- **Tropical** — monstera and palm leaf silhouettes in ocean at two opposite
  corners, transparent elsewhere.

Selector UI: a row of four thumbnails showing each frame over a neutral
placeholder. Selected state = 2px gold ring + `aria-checked="true"`. Arrow keys
move between them.

**Critical SVG constraint:** an SVG that will be drawn to `<canvas>` must carry
explicit `width` and `height` attributes on the root element, not just a
`viewBox`. Without them, Firefox and Safari draw nothing. Set both to the
capture resolution.

### Live webcam preview

```js
// Requests camera access. Returns a MediaStream the browser owns; the user
// can revoke it at any time from the address bar.
//   video: ask for a 4:5 portrait framing at up to 1080px wide. `ideal`
//          rather than `exact` so laptops with fixed 16:9 sensors still work —
//          the browser gives us the closest match and we crop in the canvas.
//   audio: false. We never touch the microphone; saying so out loud in the UI
//          matters for a client-facing page.
const stream = await navigator.mediaDevices.getUserMedia({
  video: { width: { ideal: 1080 }, height: { ideal: 1350 }, facingMode: 'user' },
  audio: false,
});
videoEl.srcObject = stream;
await videoEl.play();
```

Requirements:

- Camera is **not** requested on page load. It starts only when the user
  presses "Start camera". Auto-prompting on load is hostile and most browsers
  block it anyway.
- `<video>` gets `autoplay`, `muted`, `playsinline` (iOS Safari refuses to play
  inline without all three).
- Preview is mirrored with CSS `transform: scaleX(-1)` so it behaves like a
  mirror. The frame overlay sits above it and is **not** mirrored.
- The preview container is a fixed 4:5 box (`aspect-[4/5]`) with `object-cover`
  on the video, so the on-screen framing matches what gets captured.
- Stop the stream when the user leaves the section or navigates away:
  `stream.getTracks().forEach(t => t.stop())`. Also stop it on
  `visibilitychange` → hidden. Leaving the camera light on is a trust problem.

Handle every failure path with a distinct, non-apologetic message rendered in
the preview box:

| Condition | Message |
|---|---|
| `NotAllowedError` | Camera access is blocked. Enable it in your browser's address-bar permissions, then press Start camera again. |
| `NotFoundError` / `OverconstrainedError` | No camera found. Connect one and try again. |
| `!navigator.mediaDevices` (insecure context) | The camera needs a secure connection. Open this page over https or localhost. |
| Any other error | Include the error name in a `console.error`, show a plain retry message. |

Under the preview, in small type: "Your camera runs entirely in your browser.
Nothing is uploaded, stored, or sent anywhere."

### Capture

```js
// OUTPUT_W / OUTPUT_H define the print resolution — change these to match the
// real booth's print size. 1080×1350 is 4:5, which matches a 4×5 print.
//
// Cover-crop math: the camera's native aspect ratio rarely matches our output,
// so we scale the video up until it covers the canvas on both axes, then centre
// it and let the overflow fall off the edges. Same behaviour as CSS object-fit:
// cover — this is what makes the capture match the preview exactly.
const scale = Math.max(OUTPUT_W / video.videoWidth, OUTPUT_H / video.videoHeight);
const drawW = video.videoWidth  * scale;
const drawH = video.videoHeight * scale;
const dx = (OUTPUT_W - drawW) / 2;
const dy = (OUTPUT_H - drawH) / 2;

// The preview is mirrored for the user's benefit, so the capture must be
// mirrored too or the photo won't match what they just saw. We flip the canvas
// coordinate system, draw the video, then RESTORE before drawing the frame —
// otherwise any text in the frame comes out backwards.
ctx.save();
ctx.translate(OUTPUT_W, 0);
ctx.scale(-1, 1);
ctx.drawImage(video, dx, dy, drawW, drawH);
ctx.restore();

ctx.drawImage(frameImage, 0, 0, OUTPUT_W, OUTPUT_H);  // overlay, unmirrored
```

- Canvas is hidden / off-DOM. Never shown to the user.
- Three-second countdown before the shutter (large numeral over the preview),
  then a white flash: an absolutely-positioned white div at `opacity-0` →
  `opacity-90` → `opacity-0` over ~350ms. Skip the flash under
  `prefers-reduced-motion`.
- Output via `canvas.toBlob(blob => …, 'image/jpeg', 0.92)` and
  `URL.createObjectURL`. Revoke the old object URL on retake to avoid leaking
  memory.
- After capture: swap the control row to "Retake" and "Save photo". Save = an
  `<a download="hawaii-booths.jpg">` pointed at the object URL.

**Canvas tainting warning** — put this in a comment: frame images loaded from
another origin taint the canvas and make `toBlob` / `toDataURL` throw a
`SecurityError`. The inline SVG data URIs are safe. If the client swaps in
hosted PNGs, they must either be served from the same origin
(`assets/frames/`) or be served with `Access-Control-Allow-Origin` and loaded
with `img.crossOrigin = 'anonymous'` set before `img.src`.

## 4. Scroll-linked printing animation

Once a photo exists, reveal the printer and tie the photo's emergence to scroll
position.

### Structure

```html
<div id="printer-stage" class="relative">
  <!-- Photo emerges from behind the printer's top edge. The mask clips
       everything below the slot line so the photo appears to come out of it. -->
  <div id="photo-mask" class="absolute inset-x-0 bottom-[SLOT_Y] overflow-hidden">
    <img id="printed-photo" class="will-change-transform" />
  </div>
  <!-- Printer chassis: inline SVG, sits above the photo in z-order -->
  <svg id="printer-body" class="relative z-10">…</svg>
</div>
```

The photo starts at `translateY(100%)` — fully below the mask's bottom edge,
i.e. hidden inside the printer. At progress 1 it sits at `translateY(0)` —
fully emerged above the slot.

### The math

```js
// Scroll progress is a 0→1 value derived from where the printer sits in the
// viewport, NOT from absolute page scroll — this keeps it correct regardless of
// how much content sits above the section, and survives layout changes.
//
//   START: printer top is 85% of the way down the viewport → progress 0
//   END:   printer top is 30% of the way down the viewport → progress 1
// Widen the gap between these two for a slower, longer print.
function printProgress() {
  const rect  = document.getElementById('printer-stage').getBoundingClientRect();
  const start = window.innerHeight * 0.85;
  const end   = window.innerHeight * 0.30;
  const raw   = (start - rect.top) / (start - end);
  return Math.min(1, Math.max(0, raw));   // clamp to [0, 1]
}

// Apply on every animation frame that follows a scroll, never inside the scroll
// handler itself — scroll fires far more often than the compositor paints.
let ticking = false;
window.addEventListener('scroll', () => {
  if (ticking) return;
  ticking = true;
  requestAnimationFrame(() => {
    const p = printProgress();
    photo.style.transform = `translateY(${(1 - p) * 100}%)`;
    // Subtle paper curl as it emerges — drop this line if the client wants it flat.
    photo.style.rotate = `${(1 - p) * -1.5}deg`;
    ticking = false;
  });
}, { passive: true });
```

Also:

- Recompute on resize (debounced) since the maths depends on `innerHeight`.
- On first capture, smooth-scroll the printer stage into view so the user
  discovers the effect instead of having to guess.
- Under `prefers-reduced-motion: reduce`, skip the listener entirely and set the
  photo to `translateY(0)` immediately.
- The printer chassis is an inline SVG in ink / surf with a visible slot, a
  small status LED (steady gold dot), and the Hawaii Booths wordmark on the
  body. Keep it stylised and flat — no skeuomorphic gradients.
- Optionally add a faint mechanical whirr using the Web Audio API, default off
  behind a mute toggle. If it complicates things, skip it.

**GSAP alternative:** implement in vanilla JS as above. Add a short comment
block showing the equivalent ScrollTrigger config (`scrub: 0.5`, `start: 'top
85%'`, `end: 'top 30%'`) so the client can swap later, but do not add the GSAP
dependency.

## 5. Asset configuration (make swapping trivial)

Every replaceable asset goes in one clearly-commented config object at the very
top of the script, before any other code:

```js
/* ─────────────────────────────────────────────────────────────
   ASSETS — swap these for the client's real files.
   Drop images into assets/ and change the paths below. Nothing
   else in this file needs to be touched.
   ───────────────────────────────────────────────────────────── */
const ASSETS = {
  hero:     'https://picsum.photos/seed/hawaii-hero/1200/1500',
  services: { photo: '…', video: '…', audio: '…', custom: '…' },
  gallery:  [ /* 8 URLs */ ],
};

const FRAMES = [
  { id: 'classic',  name: 'Classic White', src: 'data:image/svg+xml,…' },
  // To use a real PNG instead: src: 'assets/frames/classic.png'
  // Requirements: transparent centre, same aspect ratio as OUTPUT_W:OUTPUT_H,
  // ideally 1080×1350 or larger.
];

const OUTPUT_W = 1080;   // capture width in px
const OUTPUT_H = 1350;   // capture height in px (4:5)
```

**Important — do not use `source.unsplash.com`.** Unsplash retired that
endpoint; it now returns 503 and every image renders broken. Use
`https://picsum.photos/seed/<slug>/<w>/<h>` for placeholders — deterministic,
keyless, and always resolves. If on-theme stock is wanted later, use pinned
`https://images.unsplash.com/photo-<real-id>?w=1200&q=80` URLs copied from
actual Unsplash photo pages. Never invent a photo ID.

Give every `<img>` explicit `width`/`height` attributes and `loading="lazy"`
below the fold, so the page doesn't shift as images arrive.

## 6. Comment requirements

Write comments that explain **why**, aimed at a developer handing this to a
non-technical owner later. Specifically, block comments are required above:

- The `getUserMedia` call — what the constraints mean, why `ideal` not `exact`,
  why audio is off, why it's user-triggered.
- The capture function — the cover-crop maths and the mirror save/restore, line
  by line.
- The scroll progress function — what start and end mean in plain terms and
  which numbers to change to make the print slower.
- The `FRAMES` array — exact spec for replacement frame files (dimensions,
  transparency, aspect ratio, same-origin requirement).
- The Tailwind CDN block — that it's dev-only.

Skip narrating obvious lines. No comment should restate what the code plainly
says.

## 7. Acceptance checklist

Before telling me you're done, verify each of these yourself:

- [ ] Page renders with zero console errors on `http://localhost`.
- [ ] Nav dropdown and mobile menu open, close on Escape, and are
      keyboard-navigable.
- [ ] "Start camera" prompts for permission; denying it shows the
      blocked-access message, not a broken UI.
- [ ] Preview is mirrored; the saved photo matches what was on screen (no
      reversed frame text).
- [ ] Frame switching updates both the live overlay and the captured output.
- [ ] "Save photo" downloads a valid JPEG at 1080×1350 with the frame baked in.
- [ ] `toBlob` does not throw — canvas is untainted.
- [ ] Scrolling after capture drives the photo out of the printer slot
      smoothly, and reversing scroll reverses the animation.
- [ ] Camera light turns off when the tab is hidden and when the page unloads.
- [ ] `prefers-reduced-motion: reduce` disables the countdown flash and scroll
      animation.
- [ ] Layout holds at 375px, 768px, 1280px, and 1920px.
- [ ] Tab through the whole page: focus is always visible and never trapped
      outside a modal.
- [ ] No `source.unsplash.com` URLs anywhere.
- [ ] No leading-slash asset paths — `grep -n 'src="/' index.html` returns
      nothing.
- [ ] After merging to `main`, the deployed Pages URL loads with all images
      intact and the camera works there.

## 8. Working style

Build it in this order, committing and stopping for my review after each phase:
(1) design tokens + nav + hero, (2) services / how-it-works / pricing / gallery
/ testimonials / contact / footer, (3) the Try It Out section, (4) the printer
animation. Don't write the whole file in one pass, and don't batch four phases
into one commit — I want to be able to roll back a phase without losing the
others.

Phases 3 and 4 are the ones I'll actually scrutinise. If you're unsure about a
trade-off there, ask before implementing rather than guessing.

---

## Deviations from this brief

Recorded so the differences are deliberate and reviewable rather than
accidental.

1. **Printer is not sticky.** The layout sketch in section 3 labels the printer
   "sticky", but the progress maths in section 4 reads
   `#printer-stage`'s `getBoundingClientRect().top`. A stuck element's `top`
   stops changing, so progress would freeze mid-print. Decided in review: keep
   the stage in normal flow and use the section 4 maths verbatim.

2. **Focus is not trapped in the Services dropdown.** Section 2 asks for a
   trap. A trap belongs in a modal; in a nav menu it strands keyboard users.
   Tab out closes the menu instead. Escape still closes and returns focus to
   the trigger. The mobile overlay *is* a modal and does trap focus.

3. **Pricing uses the real booth names and prices**, not Classic / Signature /
   Island at 2/3/4hr. After the brief was written, the client asked for content
   to follow hawaiibooths.com, where every package is a two-hour rental priced
   per booth type. The three-card layout and the ocean-filled middle card are
   unchanged; the middle tier is the Digital Photo Booth at $625.

4. **The countdown numerals are kept under `prefers-reduced-motion`**, and only
   the white flash is skipped. The countdown is information — it tells you when
   the shutter fires — rather than decoration.

5. **Frame switching after a capture re-composites the existing photo** instead
   of requiring a retake. The raw camera frame is held on a second off-DOM
   canvas to make this possible.

6. **The mechanical whirr (optional in section 4) was skipped.**
