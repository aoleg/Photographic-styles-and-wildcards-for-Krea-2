# Krea-2 Styles and Wildcards for Forge Neo

A `styles.csv` style library written specifically for **LLM-encoder image models** (Krea-2, and
anything else using a Qwen3-VL / T5 / similar text encoder), as opposed to the CLIP-L/CLIP-G
encoders that `styles.csv` was originally designed around.

The YAML wildcard version can be used with Forge Neo / Comfy / SwarmUI.

The category names now refer to wildcards; place the YAML file into the "film" subfolder of your
wildcards folder for this to work, e.g.
sd-webui-forge-neo\extensions\sd-dynamic-prompts\wildcards\film

Differences between V1 (descriptive) and V2 (fully stackable):
The "descriptive" version uses longer, more descriptive phrasing. The "fully stackable" version is
significantly more compact; the film stocks should now exclusively affect tonal and color values,
not composition. For example:

V1:
`Kodak Portra 400 [Soft & Flattering],"The photo is in the style of Kodak Portra 400 film: warm natural skin tones, fine grain, gentle contrast with shadow and highlight detail retained, and an understated, flattering portrait look.`

V2:
`Kodak Portra 400 [Soft & Flattering],"Style and tones: Kodak Portra 400, with warm natural skin tones, gentle contrast, retained shadow and highlight detail, and fine grain.`

In my tests, the V2 is more subtle, affects composition less than the original, and the stacking works better.
Every entry now opens with a labelled axis, so they concatenate cleanly. Nine labels:

* Style and tones: — all film stocks (Western, Soviet, DDR).
* Era: — decades.
* Style: — digital snapshots and the misc bin (device/process looks).
* Lighting: — the lighting section and flash (flash is a lighting event).
* Composition: — framing, camera position, and motion.
* Color: — the standalone grade section.
* Atmosphere:, Focus:, Quality: — the rest.

Example:

V1:
`Low Angle [Looking Up & Heroic],"The photo is taken from a low angle looking up at the subject: the camera positioned near ground level and tilted upward, the subject towering against the sky or ceiling, converging vertical lines rising through the frame, foreground ground plane compressed at the bottom edge, and an imposing, monumental presence."`

V2:
`Low Angle [Looking Up & Heroic],"Composition: viewed from below, low-angle shot, the subject towering, and converging verticals rising through the frame.",`


## Why this exists

The `styles.csv` format goes back to SD 1.5 / SDXL days, and most style packs floating around
still write prompts the CLIP way: comma-separated tag soup in the positive ("masterpiece, best
quality, photography, storytelling") and a wall of quality-shaming tags in the negative ("bad
anatomy, worst quality, deformed, ugly"). CLIP has no grammar, so tags were the only lever
available.

Krea-2's text encoder is a real language model (Qwen3-VL). It reads sentences, understands
relationships between clauses, and gets *less* out of keyword stuffing than a well-written
description. Feeding it CLIP-style tags is mostly inert at best, and actively confusing at worst.

This pack drops the tag-soup approach entirely:

- **Positives are full sentences.** Every style reads roughly as "The photo is in the style
  of / lit with / composed with / taken from / graded with **X**: [concrete, physical
  description of what that produces]." No "masterpiece," no "best quality," no "trending on
  artstation."
- **Negatives are optional and mostly for RAW.** Krea-2 Turbo is a distilled model running at
  CFG ~1, meaning the negative-prompt branch is never evaluated — negatives are pure dead weight
  there. On Krea-2 RAW (or any non-distilled model that does use CFG), the negatives describe the
  *specific opposite failure mode* for that style, not a generic "worst quality, low quality"
  incantation.
- **Named references are reinforced, not assumed.** "Kodachrome," "chiaroscuro," "Dutch angle"
  etc. are real terms the encoder likely already has some association with. Rather than relying
  on the name alone (which is how CLIP-era styles worked, and inconsistently at that), every style
  restates what that name actually looks like in plain language, so the description carries the
  look even if the name-anchor doesn't land.

## How the categories work together

The categories are designed as **independent, combinable axes**, not a single choice of "pick one
style." A film stock, a lighting setup, a composition rule, a camera position, a color grade, and
an atmosphere can all be layered onto the same subject at once, because each one only touches its
own axis and leaves the others alone:

```
{subject}. The photo is in the style of Kodak Portra 400 [...]. The photo is lit with
strong rim lighting [...]. The photo is composed with vast negative space [...]. The photo
is taken from a low angle looking up [...].
```

A few categories are *not* fully independent and will fight each other if combined carelessly —
those conflicts are called out per-category below (mainly: things that fix the camera's position
already, like a dashcam or CCTV shot, don't mix with the Camera Position & Optics axis; a film
stock already implies a color grade, so stacking a Color & Tonality style on top of a stock is
usually redundant rather than harmful).

## Categories

| Section | What it controls |
|---|---|
| **Quality** | Baseline image fidelity, no posing/staging implications |
| **Film Stocks** | Specific analog/instant film emulsions |
| **Decades** | Photographic eras and their dominant visual conventions, 1920s-2020s |
| **Digital Snapshots** | Amateur phone photography: candid, night-flash, selfies (arm's-length and fisheye), mirror, tourist, food |
| **Flash Photography** | Direct/on-camera flash as a lighting event, from clean to chaotic to startled |
| **The Misc Bin** | Found-footage devices, toy/optical cameras, and 19th-century processes |
| **Lighting** | Direction and quality of light only, independent of genre or setting |
| **Composition** | Where things sit in the frame |
| **Camera Position & Optics** | Where the camera is and what lens is implied |
| **Color & Tonality** | Grade/palette, independent of film stock or era |
| **Atmosphere** | Weather and air conditions in the scene itself |
| **Moment & Motion** | How time and movement are frozen or smeared |
| **Focus & Depth** | Depth of field, independent of lens or stock |


### Category notes

- **Quality** avoids "professional" and "studio" on purpose — both terms drag in posing and
  staging connotations in training data. These two styles describe only sensor/optical
  characteristics (dynamic range, noise, bokeh from a large aperture), so they layer cleanly onto
  any subject or scene without implying anyone was photographed on purpose.
- **Film Stocks** name real stocks (Portra, Velvia, Tri-X, Instax, etc.) and restate each one's
  actual characteristics rather than trusting the name alone. Instant-film entries explicitly
  instruct a full-bleed frame with no white border, since that's the opposite of most instant-photo
  training data.
- **Decades** covers both the classic film eras (1920s-1990s) and the digital-native eras
  (2000s-2020s) in one continuous timeline, since visually they're the same kind of category —
  "what did photography commonly look like in this period."
- **Lighting** and **Camera Position & Optics** deliberately open with "The photo is lit with..."
  / "The photo is taken from..." instead of "in the style of...", since lighting and optics aren't
  a genre — this phrasing keeps them acting as physical properties rather than pulling in a whole
  aesthetic movement, and is what makes them stack safely under any other style.
- **Composition** is the axis with the weakest, most probabilistic adherence — theory-named rules
  like "rule of thirds" are gentle nudges rather than hard constraints in current models, unlike
  the harder physical/optical axes.
- **The Misc Bin** and **Camera Position & Optics** overlap: several Misc entries (CCTV, dashcam,
  trail camera, GoPro) already fix the camera's viewpoint as part of the style, so stacking a
  Camera Position style on top of those is redundant rather than additive.

## Naming convention

```
Style Name [Short Look Descriptor]
```

- **Style Name** is what a person would recognize and search for: a real film stock (`Kodak
  Portra 400`), a technique (`Chiaroscuro`), a device (`CCTV Security`), or a decade + genre
  (`1940s Film Noir`).
- **`[Short Look Descriptor]`** is 1-3 words in square brackets giving an at-a-glance sense of the
  result, since the name alone doesn't always tell you what to expect (does `Ektachrome` skew warm
  or cool? does `Delta 3200` mean grainy daylight or grainy night?). This bracket is only ever
  shown to the person browsing styles — it is never part of the actual prompt sent to the model.
  - `&` is used inside the bracket instead of the word "and" purely to keep these tags short and
    scannable in a dropdown list.
  - Where a look is explicitly amateur/consumer-grade rather than the professional-grade version
    of the same stock, that's called out directly in the bracket (`Kodak Gold 200 [Warm Amateur]`)
    rather than left ambiguous.
  - Black-and-white stocks are tagged `B&W` in the bracket so they're identifiable without reading
    the full prompt.
- **`[ Section Name ]`** rows (with empty prompt/negative fields) are pure visual separators for
  organizing the section headers inside a styles dropdown — they carry no prompt content and
  render as blank entries if selected by mistake.

## Format

Standard A1111/Forge `styles.csv`: `name,prompt,negative_prompt`, with `{prompt}` as the
placeholder for whatever subject text the person enters. File is plain ASCII/UTF-8 with no
Windows-1252 dash characters (`0x96`/`0x97`), which are known to break the WebUI CSV parser.

## Wildcard version

`styles_wildcards.yaml` is the same 138 styles converted for the
[sd-dynamic-prompts](https://github.com/adieyal/sd-dynamic-prompts) extension, for people who
want to insert a random style inline rather than pick one from the Forge/A1111 style dropdown.

Each CSV category becomes a wildcard accessed as `__styles/<category>__` or `__/film/styles/<category>__`
if you placed it into the `film` subfolder, e.g.:

```
a red vintage bicycle leaning against a brick wall, __styles/film-stocks__, __styles/lighting__
```
or
```
a red vintage bicycle leaning against a brick wall, __film/styles/film-stocks__, __film/styles/lighting__
```



The `{prompt}` wrapper is stripped from every entry (wildcards get inserted into your own prompt
text rather than wrapping it), and category names are slugified (`Camera position & optics` ->
`camera-position-optics`). Negatives are cleaned out.

## Known limitations

- Name-anchor strength varies. Famous stocks and looks (Kodachrome, Tri-X, Velvia, Portra,
  Polaroid, film noir, Y2K, indie sleaze) are heavily represented in training data and should
  render reliably from the name alone; rarer ones (Ektar, Provia, Acros, Autochrome) lean more
  on the written description to carry the look.
- Some styles necessarily imply scene content, not just treatment (a drone-aerial style implies
  a top-down landscape; a mall-portrait style implies a backdrop). These are noted per-category
  above where relevant, and are the tradeoff for a style being distinctive at all.
- Negative prompts are written but functionally inert on distilled/Turbo-class models. Keep them
  if you also run non-distilled checkpoints; they're harmless dead weight otherwise.
