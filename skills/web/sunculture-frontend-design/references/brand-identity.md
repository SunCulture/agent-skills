# SunCulture Brand Identity

Source: `docs/SunCulture_Brand_Guidelines.pdf` (Version 1.0, April 2019)

---

## Brand foundation

| Element         | Statement                                                                                    |
| --------------- | -------------------------------------------------------------------------------------------- |
| **Purpose**     | We create wonder in everyday lives.                                                          |
| **Mission**     | By designing affordable life-changing technology.                                            |
| **Objective**   | Which solves the biggest daily challenges for millions of people.                            |
| **Vision**      | Build a world where people take control of their environment in rewarding, sustainable ways. |
| **Positioning** | Life-changing technology for everyday challenges.                                            |

### Brand personality — four pillars

| Pillar            | Meaning                                                                         |
| ----------------- | ------------------------------------------------------------------------------- |
| **Idealistic**    | Passionate and enthusiastic. We create wonder and feel it too.                  |
| **Pragmatic**     | Products that understand and solve real-world problems.                         |
| **Visionary**     | Emboldened by technological expertise — a radical approach to what is possible. |
| **Collaborative** | We work alongside our customers as equal partners, not leaders.                 |

### Tone: pragmatic idealism

The brand holds two paradoxes:

1. We are both **idealistic** and **pragmatic**.
2. Even though we are **visionary** leaders, we **collaborate** as equal partners.

In practice:

- Aspirational language is always anchored in concrete facts and results.
- We refer to facts, statistics, and practical reasons whenever we aspire.
- We make concrete references to nearby success stories.
- We anchor "imagine…" language with tangible outcomes: higher yields, lower
  costs, water at the touch of a button.
- We keep language grounded — never use a longer word when a shorter will do.

### Basic copy principles

- **Be uplifting and benefit-led.** Sound life-changing. Bold, confident messages
  that lead with benefits. No exclamation marks — just confident delivery.
- **Speak from a position of excellence.** Premium product, no compromise on
  quality — but make this clear with certainty, not arrogance.
- **Be supportive.** Accessible via pay-as-you-grow plans. Not exclusive.
- **Champion the customer.** We partner alongside customers; we don't lead them.
  Always use "you" — reinforce their ownership.

---

## Colour

### Solid palette

| Colour      | Hex       | Token                      |
| ----------- | --------- | -------------------------- |
| Yellow      | `#F3D500` | `brandPrimitives.yellow`   |
| Green       | `#62C628` | `brandPrimitives.green`    |
| Blue (teal) | `#00CCC8` | `brandPrimitives.blue`     |
| Warm grey   | `#A0918A` | `brandPrimitives.warmGrey` |
| Orange      | `#FF8202` | `brandPrimitives.orange`   |
| Red         | `#E23D26` | `brandPrimitives.red`      |
| Purple      | `#9D2AAD` | `brandPrimitives.purple`   |
| Grey        | `#636569` | `brandPrimitives.grey`     |
| White       | `#FFFFFF` | `brandPrimitives.white`    |

Supporting colours were chosen based on the core gradient colours within the
three gradient sweeps.

### Gradient system

Gradients create clean, zingy visual energy. They can be used as:

- **Solid colour blocks** on brochure covers, print ads, web sections.
- **Multiplied over photography** to add warmth and brand energy to hero images.

**Three canonical gradient sweeps:**

| Name       | From             | Via                  | To               |
| ---------- | ---------------- | -------------------- | ---------------- |
| Warm       | Yellow `#F3D500` | Orange `#FF5C00`     | Red `#DF180D`    |
| Core brand | Yellow `#F3D500` | Lime green `#55CC2E` | Blue `#00C7F7`   |
| Vibrant    | Yellow `#F3D500` | Pink `#E83A7B`       | Purple `#C800FF` |

All three start from yellow — this is the brand's unifying warm energy point.

**Digital gradient construction:**

```css
/* Core brand gradient */
background: linear-gradient(135deg, #f3d500, #55cc2e, #00c7f7);

/* Warm gradient */
background: linear-gradient(135deg, #f3d500, #ff5c00, #df180d);

/* Vibrant gradient */
background: linear-gradient(135deg, #f3d500, #e83a7b, #c800ff);
```

**Gradient rules:**

- Choose the variation that best suits the image or context.
- Do not apply gradients to body text.
- Supporting photography (no direct product interaction shown) does not require
  a gradient overlay.
- Gradient blocks can stand alone without photography — this creates variation
  and visual pace across communications.

### Design tokens

Implement brand colours as CSS custom properties in your project — never
hardcode raw hex values. Full token documentation: `references/design-tokens.md`.

Primary interactive colour: `#07B282` (teal) — maps to `--color-accent`.

---

## Typography

### Primary typeface: Circular Std

Circular Std is the primary typeface for all SunCulture communications. Its
round shapes reflect and complement the circle in the logo. It is friendly
while being clean and professional.

**Available weights:**

| Weight | Name   | Use                                   |
| ------ | ------ | ------------------------------------- |
| 400    | Book   | Body text, form fields, table content |
| 500    | Medium | Subheadings, labels, navigation       |
| 700    | Bold   | Display headings, hero copy           |

**Not available as a web font by default** — load via `next/font/local` from
brand assets, or reference via a licensed font service.

### System fallback: Arial

When Circular Std is unavailable (legacy environments, printed materials
without embedded fonts), use Arial Book and Arial Bold as the fallback.

### Typography rules

- Never use Inter, Roboto, Space Grotesk, or system-ui as the primary body
  font on a SunCulture-branded surface.
- Headlines: Circular Std Medium (500) or Bold (700).
- Body: Circular Std Book (400).
- Use `font-variant-numeric: tabular-nums` for number columns in tables.
- Heading text benefits from `text-wrap: balance` to avoid awkward wraps.
- Use ellipsis `…` (U+2026), not three dots `...`.
- Use curly quotes `"` `"`, not straight quotes `"`.

---

## Logo

### Variations

The logo has three colour variations, each with stacked and horizontal layouts:

| Variation                                      | When to use                             |
| ---------------------------------------------- | --------------------------------------- |
| Primary (green–teal gradient)                  | Default — digital, print, most contexts |
| Secondary (pink–purple or orange–red gradient) | Alternative brand moments               |
| Monochrome (black on white, or white on dark)  | Black-and-white printing only           |

**Stacked vs horizontal:**

- Use the stacked (circle above wordmark) layout by default.
- Use the horizontal (circle beside wordmark) only when the stacked layout
  does not fit the available space.

### Exclusion zone

Clear space = at least half the height of the circle on all four sides of
the complete logo (mark + wordmark). No text, images, or other graphics should
appear within this zone.

### Minimum digital sizes

| Layout     | Minimum width |
| ---------- | ------------- |
| Stacked    | 100px         |
| Horizontal | 57px          |

### Logo don'ts

- Do not stretch or distort proportions.
- Do not rotate the logo.
- Do not recolour the logo (e.g. no yellow wordmark).
- Do not place on a mid-grey or otherwise unsuitable background.
- Do not alter, rearrange, or combine any element of the logo file.

---

## Iconography

The SunCulture icon set uses a consistent outline style that echoes the logo:

- **Shape language:** based on circles and rounded terminals.
- **Stroke:** thin, consistent weight.
- **Default colour:** brand teal `#00CCC8`.
- **Topics:** water pump, solar panel, light bulb, location pin, water drop,
  lightning bolt, sun, people (community), finance (hand + coin).

When adding functional UI icons (not marketing assets):

- Use your project's icon library for standard UI actions (e.g. Ant Design icons,
  Lucide, Heroicons) — choose based on project conventions.
- Ensure icon style is consistent with surrounding UI — thin outline icons work
  best; avoid filled/solid styles that clash with the brand outline system.
- Icons in the SunCulture brand style should use teal `#07B282` (`--color-accent`).

---

## Photography

### Hero photography

Shows real people — farmers and customers — **using and benefiting** from
SunCulture products in real environments.

Characteristics:

- Genuine emotion: joy, productivity, abundance.
- Active subjects — harvesting, working, laughing, thriving.
- Natural outdoor light; warm, golden-hour tones preferred.
- Apply a brand gradient multiplied over the image, choosing the variation
  that best suits the image's colour temperature.

### Supporting photography

Shows close-up detail: hands, produce, products, animals, land.

- Products should be photographed professionally (clean environment or
  in-field with careful composition).
- No gradient overlay required on supporting images.
- People in supporting shots do not need to interact with a product.

### Photography don'ts

- No stock-photo aesthetic — images must feel specific and real.
- No staged corporate group shots.
- No images centred on hardship without the solution visible or implied.
- Products must look premium — re-shoot rather than use low-quality product shots.
