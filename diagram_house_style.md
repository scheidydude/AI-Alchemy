# Architecture Diagram House Style

A reusable visual standard for all our architecture diagrams. Paste this whole
file into a new conversation (or attach it) and say *"build the diagram using
this house style."* It defines a fixed **grammar** (layout, typography, icons,
connectors, quality bar) plus a **theme system** so diagrams across every
product look like one family while adapting to mode (light/dark/print),
domain (infra/software/business/...), brand, and output density.

---

## How to use this in a new conversation

> Use the attached house style for this diagram. Declare a theme, e.g.
> `theme: dark x software, density: slide`. Deliver an editable vector SVG
> plus a 3x PNG, built by a reusable generator. If you can run code, render the
> SVG with headless Chromium and visually inspect and fix your own output before
> delivering. Use the real component names I give you, apply the theme's
> surface tokens and category map and the icon catalog consistently, and run
> the self-check list.

---

## 0. Theme system

A theme is composed of four independent layers. Only colors and sizes change
between themes — layout grammar, glyphs, typography scale, and connector
meanings never do.

```
theme: <mode> x <domain> [+ <skin>] [, density: <preset>] [, colorblind-safe]
```

| Layer | Options | Default |
|---|---|---|
| **Mode** (surface palette, §2) | `light`, `dark`, `print`, `whiteboard` | `light` |
| **Domain** (category map, §3) | `infrastructure`, `software`, `business`, `data`, `security` | `infrastructure` |
| **Skin** (brand override, §3.7) | any named client/brand skin | none |
| **Density** (size preset, §4.1) | `poster`, `slide`, `inline` | `slide` |

If no theme is declared, use `light x infrastructure, density: slide`.

---

## 1. Canvas & layout grammar (all themes)

- **Orientation:** left-to-right along the system's real data/control flow.
- **Title:** centered at top. Product name in heavy title color, the rest in
  medium ink.
- **Zones:** group components into labeled regions, each introduced by a small
  numbered badge (`1. ...`, `2. ...`) in the title color.
- **Bands:** give cross-cutting concerns (shared platform services, governance,
  observability) their own full-width horizontal band, not a column.
- **Lanes:** when a zone has parallel categories, render them as side-by-side
  lanes, each light-tinted with its accent color and a matching colored header.
- **Legend:** always present, bottom corner. Include the theme name in small
  sub-gray text under the legend (e.g. `dark x software`).
- **Fit:** size every container to its contents. No large dead space, no
  cramped overlaps. Balance whitespace across the canvas.
- **Reference canvas:** ~1700 x 1170 viewBox works well for a dense platform
  diagram at `slide` density; scale to content.

---

## 2. Modes — surface palettes

Structural / neutral tokens per mode. Same token names everywhere so the
generator swaps a single dict.

### 2.1 `light` (default)

| Token       | Hex       | Use                                  |
|-------------|-----------|--------------------------------------|
| `title`     | `#16315C` | Title, zone badges, container titles |
| `badge`     | `#1B3A6B` | Badge fill                           |
| `ink`       | `#1F2937` | Primary label text                   |
| `sub`       | `#5B6675` | Sublabel / secondary text            |
| `line`      | `#9AA7B8` | Light internal connectors            |
| `card_str`  | `#D9E0EA` | Card / panel borders                 |
| `card_fill` | `#FFFFFF` | Card background                      |
| `page`      | `#FFFFFF` | Canvas background                    |
| `flow`      | `#5B6675` | Solid connector (request/data)       |
| `control`   | `#8A93A3` | Dashed connector (deploy/control)    |
| `private`   | `#3E8E7E` | Dash-dot connector (private/IAM)     |

Drop shadow: dy 1.5, blur ~2.2, title-navy @ 12% opacity.

### 2.2 `dark`

For docs sites, terminals, dark slide decks.

| Token       | Hex       | Notes                                |
|-------------|-----------|--------------------------------------|
| `title`     | `#A8C7F0` | Light steel blue                     |
| `badge`     | `#2E4A7A` |                                      |
| `ink`       | `#E2E8F0` |                                      |
| `sub`       | `#94A3B8` |                                      |
| `line`      | `#64748B` |                                      |
| `card_str`  | `#334155` |                                      |
| `card_fill` | `#1E293B` |                                      |
| `page`      | `#0F172A` |                                      |
| `flow`      | `#94A3B8` |                                      |
| `control`   | `#6B7684` |                                      |
| `private`   | `#4FB39F` |                                      |

Accent handling in dark mode:
- **Lift accents** ~15% lightness so tiles don't go muddy (per-domain lifted
  values are listed alongside each domain map in §3; if not listed, compute:
  same hue, +15 lightness, -10 saturation).
- **Tints become overlays:** lane/zone fills are the *accent at 14% opacity*
  over `card_fill`, not the light-mode tint hexes.
- No drop shadows; instead give cards a 1px `card_str` border and, for emphasis
  panels, a subtle 1px inner top highlight (`#FFFFFF` @ 6%).

### 2.3 `print` (grayscale-safe)

For PDFs, patents, formal docs — must survive B&W photocopying. Categories are
distinguished by **luminance ladder + border treatment**, not hue.

| Token       | Hex       |
|-------------|-----------|
| `title`     | `#111111` |
| `badge`     | `#222222` |
| `ink`       | `#1A1A1A` |
| `sub`       | `#555555` |
| `line`      | `#999999` |
| `card_str`  | `#BBBBBB` |
| `card_fill` | `#FFFFFF` |
| `page`      | `#FFFFFF` |
| `flow`      | `#333333` |
| `control`   | `#777777` |
| `private`   | `#555555` |

Category accents are replaced by this 6-step luminance ladder (assign in the
same order as the domain map's rows):

| Step | Accent    | Tint      | Extra cue                |
|------|-----------|-----------|--------------------------|
| 1    | `#1F2937` | `#EDEFF2` | solid border             |
| 2    | `#414D5C` | `#E4E8ED` | solid border             |
| 3    | `#5F6E80` | `#DDE2E8` | dashed border            |
| 4    | `#7E8DA0` | `#D5DBE3` | dashed border            |
| 5    | `#9CAABA` | `#CDD5DE` | dotted border            |
| 6    | `#BAC5D1` | `#C5CED9` | dotted border            |

Because hue is gone, the **glyph carries category identity** — never omit
glyphs in print mode, and label every lane header with its category name.
Connector styles (solid/dashed/dash-dot) already survive grayscale; keep
stroke widths ≥1.6.

### 2.4 `whiteboard` (sketch / draft)

Signals "draft, not committed architecture" for early design reviews.

| Token       | Hex       | Notes                          |
|-------------|-----------|--------------------------------|
| `title`     | `#3D4A5C` |                                |
| `badge`     | `#4A5A70` |                                |
| `ink`       | `#33404F` |                                |
| `sub`       | `#7A8694` |                                |
| `line`      | `#AEB8C4` |                                |
| `card_str`  | `#B8C2CE` | **dashed** card borders (`5 4`)|
| `card_fill` | `#FCFCFA` | faint warm off-white           |
| `page`      | `#F7F6F2` | paper tone                     |
| `flow`      | `#6E7A88` |                                |
| `control`   | `#98A2AE` |                                |
| `private`   | `#5E9A8C` |                                |

- **No drop shadows.** Corner radius bumped to ~30% on tiles for a softer feel.
- Desaturate the active domain's accents by ~40% and lift lightness ~10%
  (e.g. infra Compute `#E8740C` → `#D9985C`); tints likewise washed out.
- Add a small `DRAFT` chip (sub-gray outline pill) next to the title.

---

## 3. Domains — category color maps

Every component is colored by **category**, not by vendor. The same kind of
thing is always the same color across all diagrams *within a domain*. Each
category has a solid accent (icon tile) and a light tint (lane/zone fill).
**Rule:** ≤6 accents per diagram plus brand accents, or the color map stops
carrying meaning.

### 3.1 `infrastructure` (default)

| Category                          | Accent    | Tint      | Dark accent |
|-----------------------------------|-----------|-----------|-------------|
| Compute (containers, services)    | `#E8740C` | `#FBEBDB` | `#F19442`   |
| Storage / files / catalog         | `#4F9A2E` | `#E7F2DD` | `#72B653`   |
| ML / AI / models                  | `#1AA391` | `#DCF1EC` | `#3EC0AE`   |
| Database / identity               | `#2D6FE0` | `#E3ECFB` | `#5C92EA`   |
| Networking / app integration      | `#7B3FF2` | `#ECE3FE` | `#9B6BF6`   |
| Management / governance / observ. | `#E0257B` | `#FBE0EE` | `#EA5799`   |

### 3.2 `software` (application architecture)

Cooler set — blues/teals dominant; the single warm accent is reserved for
messaging/async so queues and events pop.

| Category                              | Accent    | Tint      | Dark accent |
|---------------------------------------|-----------|-----------|-------------|
| UI / frontend / clients               | `#2D9CDB` | `#E0F1FB` | `#5BB4E5`   |
| API / edge / service interface        | `#2D6FE0` | `#E3ECFB` | `#5C92EA`   |
| Domain logic / core services          | `#1AA391` | `#DCF1EC` | `#3EC0AE`   |
| Data layer / persistence              | `#5561C9` | `#E4E6F8` | `#7A85D8`   |
| Messaging / async / events            | `#E8740C` | `#FBEBDB` | `#F19442`   |
| Cross-cutting (auth, logging, config) | `#7B3FF2` | `#ECE3FE` | `#9B6BF6`   |

### 3.3 `business` (workflow / process)

Warmer, softer set for non-technical audiences. Use `poster` or `slide`
density with the larger end of tile/label sizes; prefer sublabels in plain
language. Adds the **diamond** glyph for decisions (§5).

| Category                       | Accent    | Tint      | Dark accent |
|--------------------------------|-----------|-----------|-------------|
| People / roles / teams         | `#2D6FE0` | `#E3ECFB` | `#5C92EA`   |
| Process step / activity        | `#4F9A2E` | `#E7F2DD` | `#72B653`   |
| Decision / gate                | `#E8A20C` | `#FCF0D6` | `#F0BA42`   |
| System of record               | `#7B3FF2` | `#ECE3FE` | `#9B6BF6`   |
| Document / artifact            | `#1AA391` | `#DCF1EC` | `#3EC0AE`   |
| External party / customer      | `#E0257B` | `#FBE0EE` | `#EA5799`   |

### 3.4 `data` (data platform / pipeline)

Sequential cool→warm hue ramp, left to right, reinforcing pipeline direction:
sources are coolest, serving is warmest, governance sits apart in magenta.

| Category                     | Accent    | Tint      | Dark accent |
|------------------------------|-----------|-----------|-------------|
| Sources / producers          | `#2D6FE0` | `#E3ECFB` | `#5C92EA`   |
| Ingestion / streaming        | `#1AA391` | `#DCF1EC` | `#3EC0AE`   |
| Storage / lake / warehouse   | `#4F9A2E` | `#E7F2DD` | `#72B653`   |
| Transform / processing       | `#D9A00B` | `#FAF0D2` | `#E5B93F`   |
| Serving / BI / consumers     | `#E8740C` | `#FBEBDB` | `#F19442`   |
| Governance / quality / lineage | `#E0257B` | `#FBE0EE` | `#EA5799` |

### 3.5 `security` (threat models, trust boundaries)

Leverages instinctive color meaning: **red is reserved exclusively for
threats/risks** and **green exclusively for controls** — never reuse them.

| Category                    | Accent    | Tint      | Dark accent |
|-----------------------------|-----------|-----------|-------------|
| Trust zones / boundaries    | `#2D6FE0` | `#E3ECFB` | `#5C92EA`   |
| Identity / principals       | `#7B3FF2` | `#ECE3FE` | `#9B6BF6`   |
| Controls / mitigations      | `#2E8B4F` | `#DFF2E5` | `#55AC74`   |
| Threats / risks / findings  | `#D93025` | `#FBE3E1` | `#E45C52`   |
| Monitoring / detection      | `#E8740C` | `#FBEBDB` | `#F19442`   |
| Assets / data / crown jewels| `#5B6675` | `#E8EBEF` | `#8391A0`   |

Trust boundaries are drawn as zone containers with a 2px accent border in the
Trust-zones blue, dashed (`7 5`).

### 3.6 Colorblind-safe variant (`colorblind-safe`)

Any domain can be requested colorblind-safe. Substitute this Okabe–Ito-derived
ladder for the domain's six accents, **in the domain table's row order**
(tints = accent at 15% over white; dark accents = +12 lightness):

| Row | Accent    |
|-----|-----------|
| 1   | `#0072B2` |
| 2   | `#56B4E9` |
| 3   | `#009E73` |
| 4   | `#E69F00` |
| 5   | `#D55E00` |
| 6   | `#CC79A7` |

Exception for `security`: keep Threats on `#D55E00` and Controls on `#009E73`
regardless of row order, then assign the rest. As in print mode, glyphs and
lane header text carry identity — never rely on hue alone.

### 3.7 Brand / client skins

A skin is a thin override applied *on top of* any mode x domain combo. It may
override **only**: `title`, `badge`, and at most **one** category accent
(usually the domain's most prominent category), plus tint recomputed at 12%
over `card_fill`. Everything else stays house-standard so the family
resemblance holds. Define skins as named token dicts, e.g.:

```
skin "acme": title=#7A1F2B, badge=#8C2433, accent[Compute]=#B03040
```

Brand-specific accents for named tools (all themes, icon tile only):

| Thing   | Accent    |
|---------|-----------|
| GitHub  | `#24292F` (light/print) / `#C9D1D9` tile with dark glyph (dark) |
| Jenkins | `#CE3B30` |

### 3.8 Lane header tints (light mode; pattern shared by all domains)

| Lane kind | Fill      | Border    | Header text |
|-----------|-----------|-----------|-------------|
| Lane A    | `#EDF3FC` | `#BBD2F0` | `#1F5FB5`   |
| Lane B    | `#EEF7E8` | `#C6E2B5` | `#4E8A2E`   |
| Lane C    | `#FDF1E3` | `#F2D3AB` | `#C16A12`   |

In dark mode, lane fills are the lane border color at 12% opacity; header text
uses the lane's light-mode border color.

---

## 4. Typography (all themes)

- Font stack: `'Segoe UI', 'Helvetica Neue', Helvetica, Arial, sans-serif`.
  (`whiteboard` mode may substitute `'Comic Neue', 'Segoe Print'` ahead of the
  stack if available; otherwise same stack.)
- Title: ~27px, weight 800 (product name) / ~22px weight 600 (the rest).
- Zone badge: ~15px, weight 700, white (or `page` color in dark mode).
- Container title: ~13–14px, weight 700, `title` color.
- Component label: ~11–12px, weight 700, `ink`.
- Sublabel: ~9px, weight 400, `sub`.
- Never let text overflow its box. Wrap to two lines before shrinking below ~8px.

### 4.1 Density presets

Multiply the base sizes above (and tile sizes in §5) by:

| Preset   | Scale | Use                                            |
|----------|-------|------------------------------------------------|
| `poster` | 1.25x | Wall prints, all-hands screens, business audiences |
| `slide`  | 1.0x  | Decks, docs pages (default)                    |
| `inline` | 0.8x  | Embedded in READMEs / wikis at narrow width; drop sublabels in dense lanes |

Density scales sizes only — never the palette or grammar.

---

## 5. Icon tiles (all themes)

Each component is a tile, not clip art:

- Rounded **colored square** filled with the category accent (corner radius
  ~22% of size; ~30% in `whiteboard`), containing a **simple glyph** drawn
  with 2px strokes.
- **Glyph color by luminance:** white on dark accents, `ink` on light accents.
  Compute relative luminance of the accent (0.2126 R + 0.7152 G + 0.0722 B);
  if it exceeds ~0.55, draw the glyph in `ink` instead of white. This matters
  most for print mode's lighter ladder steps (5–6) and for pale skin/brand
  accents — white glyphs disappear on them.
- To the right: a **bold label** and a smaller **gray sublabel** (what it is /
  does). For tiles inside dense lanes, label only.
- Tile sizes (at `slide` density): ~34px in feature cards, ~26–30px in lanes,
  ~16px inline in lists.
- Card/panel shadow per mode (§2); dark and whiteboard modes use borders, not
  shadows.

### Glyph catalog

Keep glyphs simple and consistent. Reuse these so the same concept always looks
the same. (Glyph = line-art inside the colored tile.)

| Component / concept        | Typical category   | Glyph                                  |
|----------------------------|--------------------|----------------------------------------|
| Container service / cluster| Compute            | Cube / box outline                     |
| Container registry         | Compute            | Stacked boxes with a lid               |
| Object storage / bucket    | Storage            | Bucket with elliptical rim             |
| Data catalog / ETL         | Storage            | Funnel                                 |
| Knowledge base / docs      | ML/AI              | Open book                              |
| Model / inference / agent  | ML/AI              | Chip square with radiating pins        |
| Query engine               | Networking *or* ML | Magnifier                              |
| Database / table / sessions| Database           | Cylinder (stacked disks)               |
| Identity provider (Azure)  | Database           | Two-triangle "A" mark                  |
| Load balancer              | Networking         | One node fanning to three nodes        |
| API gateway                | Networking         | Rectangle split with port dots         |
| VPC endpoint / PrivateLink | Networking         | Two nodes joined by a line             |
| Service discovery          | Networking         | Concentric circles (radar)             |
| Event bus / scheduler      | Mgmt/Govern        | Branching arrows from a node           |
| Logs / monitoring          | Mgmt/Govern        | Eye / gauge                            |
| Tracing                    | Networking         | Hexagon with center dot                |
| Secrets manager            | Mgmt/Govern        | Padlock                                |
| Config / systems manager   | Mgmt/Govern        | Gear                                   |
| Key management             | Mgmt/Govern        | Key                                    |
| Shield / WAF / SCP         | varies             | Shield (with check for protection)     |
| IAM role / access          | Compute/red        | ID card                                |
| Cost / budgets             | Storage            | Bar chart with axes                    |
| Org / account hierarchy    | Mgmt/Govern        | Tree of boxes                          |
| Policy as code / document  | Database           | Document with folded corner            |
| Tag / allocation           | Storage            | Tag with hole                          |
| Service catalog / blueprint| Database           | Framed grid with a node                |
| Users (human)              | Mgmt/Govern        | Two people                             |
| Agents / machines          | Database           | Robot head with antenna                |
| Generic file / manifest    | sub-gray           | Document                               |
| More / ellipsis            | any                | Three dots                             |

Domain-specific additions:

| Component / concept        | Domain     | Glyph                                  |
|----------------------------|------------|----------------------------------------|
| Decision / gate            | business   | Diamond outline                        |
| Handoff / approval         | business   | Hand with document                     |
| External party             | business   | Building with flag                     |
| Stream / topic             | data       | Three parallel wavy lines              |
| Transform / job            | data       | Two gears meshing                      |
| Dashboard / BI             | data       | Framed line chart                      |
| Threat actor               | security   | Mask                                   |
| Vulnerability / finding    | security   | Warning triangle with `!`              |
| Trust boundary crossing    | security   | Broken-line gate                       |

The "typical category" column names the **infrastructure** map; in other
domains, use the domain category that matches the concept's role. Add new
glyphs in the same spirit: a single recognizable line-art shape on the
category accent. When a concept isn't listed, pick the closest category color
and the simplest distinguishing shape.

---

## 6. Connector grammar (all themes)

Exactly **three** line styles, all defined in the legend. Meanings are fixed;
colors come from the mode tokens (§2: `flow`, `control`, `private`).

| Style    | Dash pattern | Meaning                  | Token     |
|----------|--------------|--------------------------|-----------|
| Solid    | none         | Request / data flow      | `flow`    |
| Dashed   | `7 5`        | Deployment / control     | `control` |
| Dash-dot | `6 3 1 3`    | Private / IAM / internal | `private` |

Domain reading of the same three styles (don't invent a fourth):

| Domain     | Solid            | Dashed             | Dash-dot                 |
|------------|------------------|--------------------|--------------------------|
| infra      | request/data     | deploy/control     | private/IAM              |
| software   | sync call        | async/event        | internal/infra concern   |
| business   | main flow        | conditional path   | information-only         |
| data       | data movement    | orchestration      | governance/lineage       |
| security   | data flow        | control applied    | privileged access        |

- Arrowheads are **colored to match their line**.
- Route with clean horizontal/vertical **elbows**, never diagonals across the
  canvas (short diagonal connectors between adjacent boxes are fine).
- Use double-headed arrows for bidirectional relationships (e.g. app <-> data).
- Stroke width ~1.6–1.8 for flows, lighter (~1.2–1.6) for internal links
  (scaled by density preset).

---

## 7. Build method

- Build the SVG with a **reusable generator** (Python recommended) exposing
  helpers: `rect`, `text`, `line`/`path`, `icon(x,y,size,color,glyph)`,
  `card(...)`, `panel(...)`, `zone_badge(...)`.
- **Theme as data:** the generator takes a `theme` object composed of
  `{mode_tokens, domain_map, skin_overrides, density_scale}` merged in that
  order. No hex literals in drawing code — tokens only — so any
  mode x domain x skin combination is a one-line change.
- Define an SVG `<defs>` block with: the mode's shadow filter (or none), an
  arrow marker using `fill="context-stroke"` (so heads inherit line color),
  and a reversed marker for double-headed arrows.
- Use plain ASCII separators (`/`, `|`) in labels — avoid middots and other
  special characters that turn into mojibake.
- **Render and inspect:** if code execution is available, render the SVG to PNG
  with headless Chromium (Playwright, `device_scale_factor` 2–3), view the
  output, fix defects, and re-render until clean. For dark mode, inspect on the
  dark page background, not a transparent one. Export the final 3x PNG
  alongside the SVG.

---

## 8. Self-check before delivery

Fix every item before sending:

1. No overlapping or clipped text — including the title and any rotated labels.
2. Icon tiles uniform in size within each tier.
3. Legend present and accurate; every line style used appears in it; theme
   name noted under the legend.
4. Clean text encoding — no mojibake; ASCII separators only.
5. Whitespace balanced; every container fits its contents.
6. Colors consistent with the declared theme's category map; ≤6 accents plus
   brand/skin accents; no light-mode tints leaking into dark/print renders.
7. Real component names used throughout — no `Service A` placeholders.
8. Connectors are elbow-routed with matching-colored arrowheads.
9. Contrast: labels readable on their backgrounds in the chosen mode (ink on
   card ≥ 7:1; glyphs on accents ≥ 3:1 — use the §5 luminance rule to pick
   white vs. `ink` glyphs). In print mode, verify the
   diagram still reads when converted to pure grayscale.
10. Red used only for threats (security domain) — never decoratively.

---

## 9. Deliverables

- `*_architecture.svg` — editable vector (every label is selectable text).
- `*_architecture.png` — 3x raster for decks and docs.
- Filenames may carry the theme when multiple variants are produced, e.g.
  `platform_architecture.dark.svg`.
- Both placed in the outputs folder and presented.
