# Phosphene Segue Generator (PSG)

A single self-contained HTML file that generates animated **scene-transition stings** — the vivid,
loading-screen-style wipes you see between scenes in racing games — and exports them with a **true alpha
matte** so you can composite them straight over video. Press **Randomize** and you always get a clean,
on-brand transition; shape it with the controls; export a lossless RGBA PNG sequence.

`psg.html` is the entire program. No install, no build, no dependencies, no network — it runs from a
`file://` path. Just open it.

---

## Quick start

1. Open **`psg.html`** in a modern desktop browser (Chrome, Edge, or Firefox — it needs **WebGL2**).
2. Hit **RANDOMIZE** until you like it, or pick a **family** tile and tweak.
3. Click **⬇ Export** → you get a `.zip` of a lossless PNG sequence + a `manifest.json`.

If WebGL2 isn't available the page says so instead of rendering.

---

## Using it

The screen is a live preview on the left, a transport bar along the bottom, and the control panel on the
right.

### Randomize & seed
- **RANDOMIZE** rolls a whole new transition. Every result is deterministic from its **seed** — the seed
  field (top-right of the panel) shows the current one; click it to select/copy, or paste a seed in and
  press **Enter** to reproduce that exact transition.

### The one interaction model (everywhere)
Every button that picks from a set — **family tiles**, the **scheme** buttons, any option row (e.g. *Intro
reveal*), and the **curve** presets — works the same way:

- **Single click** → set it now (manual choice).
- **Double-click *or* long-press** → toggle whether **Randomize** is allowed to pick it. An item that
  Randomize may choose shows a **dotted outline**.

So Randomize only ever rolls within the options you've left "dotted". By default all families are dotted
(Randomize roams them); leave only one dotted to keep Randomize inside it.

### Families
Seven content families: **Waves · Bars · Grid · Split · Mosaic · Crystal · Simple**. Each carries its own
element controls and its own set of reveals (how it enters: wipe, grow-out, shatter, dot-sweep,
white-bloom).

### Colour
An OKLCH colour picker built so the palette always stays in-gamut and on-brand:

- **Schemes** (presets): **Mono** (true greyscale), **Colourful** (rich, near-maximum chroma), **Full
  range** (open the whole colour space to Randomize). Same click / double-click model as everything else.
- **Base colour** sliders — **Hue, Lightness, Chroma, Opacity** — each track is painted with the actual
  colours along that axis, with out-of-gamut regions hatched. A **2-D pad** (lightness × chroma) and a
  **3-D gamut** view show where the base sits.
- A row of **role swatches** (Base / Dark / Light / Partner / Accent / Ground) — click one to read it out
  below in **hex / OKLCH / RGB / Figma** (pick the format from the dropdown; click the field to copy).
- The derived roles (siblings, gradient partner, accent partner, ground) expand for finer control.
- **Opacity** makes the transition see-through to its own background (the export alpha matte is preserved);
  100% by default.

### No locks — use the bands
There are no lock or pin buttons. To **fix a value through Randomize**, narrow its range. Tick **show
ranges** (top of the panel) and every slider shows three handles: the two outer ones bound the **randomize
band**, the middle one is the current value. Drag the two outer handles onto the same spot and that value
is then fixed — it survives every Randomize.

### Controls
Grouped into **Structure / reveal · Element · Palette · Motion · Effects · Title · Timing**. Sliders are
the three-handle kind above; on/off settings are an **Off · Either · On** tri-state (*Either* = let
Randomize decide); motion/reveal easing uses a draggable **curve editor**. The **Title** group has an
editable title field, font, size, and style (knockout / plate / ink) — the title is burned into the export.

### Player
Play/pause, a **loop** toggle (off = play once and hold the last frame), and a scrubber with **tick markers
where the intro / loop / outro segments begin** and a **hover thumbnail preview**. The **Intro · Loop ·
Outro · Full** tabs scope playback to one phase; a **speed dropdown** (0.25×–3×) sets the playback rate.

---

## Export

**⬇ Export** renders every frame at full resolution and gives you a `.zip` containing:

- `frame_00000.png`, `frame_00001.png`, … — **lossless RGBA** PNGs, **straight (non-premultiplied) alpha**.
- `manifest.json` — `{ width, height, fps, frames, durationSec, alpha, seed, familyKey, params, … }`.

The alpha channel **is** the matte: transparent where there's nothing, opaque where the transition is.
Drop the sequence on a timeline (or convert it) and it composites directly over your footage.

### To a video file with alpha
Unzip, then with any recent **ffmpeg**:

```bash
# ProRes 4444 (.mov) — editor-friendly, widely supported
ffmpeg -framerate 30 -i frame_%05d.png -c:v prores_ks -profile:v 4444 -pix_fmt yuva444p10le transition.mov

# VP9 with alpha (.webm) — web overlays
ffmpeg -framerate 30 -i frame_%05d.png -c:v libvpx-vp9 -pix_fmt yuva420p transition.webm

# QuickTime Animation / RLE (.mov) — lossless, simple
ffmpeg -framerate 30 -i frame_%05d.png -c:v qtrle transition.mov
```

Match `-framerate` to the `fps` you exported at (in `manifest.json`).

---

## Notes

- **Self-contained:** `psg.html` inlines everything it needs. It runs offline from `file://`; nothing is
  uploaded anywhere.
- **Determinism:** same seed → identical pixels, so a transition is fully described by its seed plus any
  manual tweaks (the exact parameters are recorded in `manifest.json`).
- **Requirements:** a WebGL2-capable desktop browser. Export size is bounded by your GPU's maximum texture
  size. Desktop layout only (no mobile/responsive view).
