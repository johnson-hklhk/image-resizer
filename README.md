# Image Compressor & Resizer

**→ [Try it live](https://johnsonhklhk.com/image-resizer/)** · **[Source on GitHub](https://github.com/johnson-hklhk/image-resizer)**

A browser-based image compressor and resizer in a single HTML file. Drop in a batch of PNG / JPG / WebP files, set a max size, and get resized + compressed versions back — with a before/after slider on every image.

**Nothing is uploaded.** All decoding, resizing and compression happens locally in your browser via Canvas. No server, no backend, no telemetry.

---

## Features

- **Batch processing** — load any number of images and process them in one pass
- **Max size box** — the width/height you type is an upper bound, not an exact size: every image is scaled down to fit inside that box, always keeping its aspect ratio. Both fields are hard caps, so mixed portrait/landscape batches all land within bounds. The height defaults to 1:1 with the width, which caps the longest side of every image regardless of orientation.
- **Never enlarges** — an image already smaller than the box is passed through untouched; "max" is a ceiling, so there is no upscaling to opt out of
- **Quick presets** — 50% / 75% / 100%, each applied per image against *its own* size, or fixed 1920 / 1280 / 1080 square boxes. A percent preset takes over from the max box while it's active; click it again to hand control back.
- **Smart PNG compression** — PNG has no quality parameter, so it's compressed via median-cut colour quantization with Floyd–Steinberg dithering (the TinyPNG approach). Full resolution and sharp edges are preserved; per-pixel alpha is kept intact.
- **Quality slider** — a single 0–100 control (default 100) driving JPG/WebP encoder quality and the PNG palette size; 100 = lossless PNG
- **Before/after compare** — draggable divider on every card, plus an expanded modal view
- **Two downloads per image** — *Resized* (geometry only, quality 1) and *Compressed* (resized + compressed)
- **Running total** of bytes saved across the whole batch

---

## Usage

The hosted version lives at **<https://johnsonhklhk.com/image-resizer/>** — nothing to install, and since all processing is local your images never leave your machine even on the hosted copy.

To run it yourself: no build step, no dependencies to install.

```bash
git clone https://github.com/johnson-hklhk/image-resizer.git
cd image-resizer
open index.html          # macOS — or just double-click the file
```

Or serve it locally if you prefer:

```bash
python3 -m http.server 8000
# then open http://localhost:8000
```

It is a static file, so it can be dropped onto any static host as-is — which is exactly how the live version is deployed.

### Workflow

1. Drop images onto the dropzone (or click to pick them).
2. The max width/height fields are seeded from the **first** image loaded — width from its width, height matching it 1:1. Nothing else is relative to that image; every image is measured against its own dimensions.
3. Adjust the **max width / max height**, or pick a percentage preset, and set the **Quality** slider.
4. Hit **Process all images**.
5. Download per image from the card buttons, or **Download all** from the bottom action bar.

Downloaded files keep their original filenames — no `_resized` / `_compressed` suffix is added, so be careful not to overwrite your source files.

---

## How compression works

| Input | Output format | Method |
| --- | --- | --- |
| JPG | JPEG | `canvas.toBlob` quality — the slider value / 100 |
| WebP | WebP | `canvas.toBlob` quality — the slider value / 100 |
| PNG | PNG | Colour quantization (median cut) + Floyd–Steinberg dithering, then DEFLATE. Quality `0–99` maps to roughly `4–255` palette colours; quality `100` skips quantization entirely and exports losslessly. |

Fewer distinct colours means the PNG's DEFLATE stage compresses far better, while resolution and edge sharpness are untouched.

Output format always matches the input format — there is no cross-format conversion (e.g. PNG → WebP).

---

## Limits & known behaviour

- **Max dimension: 8000px.** Images wider or taller than this are skipped with a toast, to avoid freezing the canvas.
- **File size is not enforced in code.** The dropzone copy suggests ≤30MB as sensible guidance, but nothing rejects a larger file — very large images will simply be slow.
- **"Download all" is not a ZIP.** It triggers individual downloads 300ms apart, so most browsers will ask permission to download multiple files. Chrome and Safari may block the batch until you allow it.
- **Processing is synchronous on the main thread.** Large batches will make the page unresponsive while working; the spinners render first, but there is no Web Worker.
- **Compression is not guaranteed.** For an already well-optimized source, re-encoding can produce a *larger* file. The savings percentage is clamped at 0% and never shown as negative — check the actual byte sizes on the card.
- **Accepted input:** `image/png`, `image/jpeg`, `image/webp` only. Other types are rejected with a toast.

---

## Tech

- Vanilla JavaScript, no framework, no bundler
- Canvas 2D API for resizing and encoding
- [Tailwind CSS](https://tailwindcss.com) via CDN for layout, re-skinned on top by a hand-written monochrome HUD stylesheet
- Google Fonts: Kode Mono, Roboto Mono, Libre Barcode 128 Text

The two CDN links (Tailwind and Google Fonts) are the only external requests. Image processing itself works entirely offline — without a network connection the tool still functions, it just loses its styling and fonts.

### Browser support

Needs a reasonably modern browser. Uses `<dialog>` + `showModal()`, `:has()` in CSS, Pointer Events and `canvas.toBlob`. WebP output depends on browser encoder support (fine in current Chrome, Firefox and Safari).

---

## Project structure

```
index.html    # everything — markup, styles, and all logic
```

That is deliberate. It keeps the tool trivially portable: one file you can email, drop in a folder, or host anywhere.

---

## License

MIT.

---

Built by Johnson Lee, Front-End Developer — [@johnson-hklhk](https://github.com/johnson-hklhk)

Issues and pull requests: <https://github.com/johnson-hklhk/image-resizer/issues>
