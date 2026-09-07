# Frame files

This folder is where replacement overlay frames go. It is empty by default —
the four frames shipped with the page are drawn as inline SVG data URIs inside
`index.html`, so the booth works with no files here at all.

Replace them only if you want artwork that is easier to edit in a design tool
than in code.

## Spec

| Requirement | Value |
|---|---|
| Dimensions | **1080 × 1350 px minimum** (larger is fine) |
| Aspect ratio | **4:5**, matching `OUTPUT_W` × `OUTPUT_H` in `index.html` |
| Format | **PNG** with an alpha channel, or SVG |
| Centre | **Fully transparent** — anything opaque covers the guest's face |
| Origin | **This repository.** See the warning below. |

A frame that is not 4:5 will be **stretched to fit, not letterboxed**. If you
change the print size, change `OUTPUT_W` / `OUTPUT_H` and re-author the frames
to match.

**JPEG will not work.** JPEG has no transparency, so a JPEG frame covers the
entire photo with an opaque rectangle.

## The cross-origin warning

Frames must be served from the same origin as the page — that means from this
folder, committed to this repository.

Loading a frame from another domain **taints the canvas**. Once tainted,
`toBlob()` throws a `SecurityError` and "Save photo" stops working completely.
The live preview will still look correct, so this breaks in a way that is easy
to miss until someone tries to save.

If a frame genuinely has to be hosted elsewhere, that host must send an
`Access-Control-Allow-Origin` header, and the image must be loaded with
`img.crossOrigin = 'anonymous'` set **before** `img.src` is assigned.

## Wiring a file up

In `index.html`, find the `FRAMES` array and replace the `src`:

```js
const FRAMES = [
  { id: 'classic', name: 'Classic White', src: 'assets/frames/classic.png' },
  ...
];
```

Keep the path **relative**. A leading slash (`/assets/frames/classic.png`)
works locally and 404s on GitHub Pages, which serves this project from
`/Hawaii_PhotoBooth/`.

The `name` is read out to screen readers and shown as the thumbnail tooltip, so
give it a real name rather than a filename.

## The four current frames

| id | name | Description |
|---|---|---|
| `classic` | Classic White | Polaroid proportions — even white border on three sides, deep bottom bar carrying the wordmark |
| `gold` | Gold Leaf | Double gold hairline inset with cut corner flourishes, otherwise clear |
| `deco` | Ocean Deco | Stepped art-deco brackets in ink at the corners, fine surf-coloured inner rule |
| `tropical` | Tropical | Monstera and palm silhouettes in ocean at opposite corners, clear through the middle |

## A note on text inside a frame

Canvas cannot use webfonts when drawing an SVG data URI, so the wordmark in
`Classic White` asks for Georgia and falls back to any serif the machine has.
If you need the brand typeface baked into a frame, export that frame as a PNG
with the text already rasterised.
