---
name: instagram-carousel
description: >
  Creates high-quality Instagram carousels as swipeable HTML previews with
  export-ready slides (1080×1350px PNG). Handles the full workflow: brand
  setup, slide copy, visual design system (colors, fonts, components), HTML
  generation, and Playwright-based export. Use this skill whenever the user
  asks to create, design, or generate an Instagram carousel, carrossel,
  slides para Instagram, or any Instagram multi-image post — even if they
  don't explicitly say "carousel" or "skill". Also trigger for requests to
  "create a post with multiple slides", "fazer carrossel", or "exportar slides
  para o Instagram".
---

# Instagram Carousel Generator

Generates fully self-contained, swipeable HTML carousels where every slide is
designed to be exported as an individual 1080×1350px PNG for Instagram.

---

## Step 1: Collect Brand Details

Before generating, ask the user for the following (if not already provided):

1. **Brand name** — displayed on first and last slides
2. **Instagram handle** — shown in the IG frame header
3. **Primary brand color** — hex code, or describe and Claude picks one
4. **Logo** — SVG path, brand initial, or skip
5. **Font preference** — see typography table below, or specific Google Fonts
6. **Tone** — professional, casual, playful, bold, minimal, etc.
7. **Images** — profile photo, screenshots, product images, etc.
8. **Idioma dos slides** — default: **Português (BR)** unless specified otherwise
9. **Carousel format** — standard (7 slides) or alternate sequence (see sequences section)

If the user provides a website URL or brand assets, derive colors and style from those.

If the user says "make me a carousel about X" without brand details, ask before generating. Don't assume defaults.

---

## Handling User-Provided Images

**This section applies from the very first HTML generation — not only during export.**

When the user provides an image file path (e.g., `/home/user/gestante.png`, `/mnt/user-data/uploads/foto.jpg`):

### ⚠️ Critical Rules

1. **NEVER use relative paths** (`gestante.png`) — they break in every browser context except the exact folder the HTML lives in.
2. **NEVER use `background: url(filepath)`** — leads to 1.5MB+ base64 inline strings that crash the browser parser.
3. **ALWAYS embed as base64 `data:` URI** — works in preview, export, and any environment.
4. **ALWAYS generate the HTML via Python** (`Path.write_text()`) — shell heredocs interpolate `$` and backticks, corrupting base64 strings.

### Step-by-step: embed an image

```bash
# 1. Check the actual file format (extension may lie)
file /path/to/image.png
```

```python
import base64
from pathlib import Path

# 2. Read and encode
img_path = Path("/path/to/image.png")
# Use "image/jpeg" if `file` command says JPEG, else "image/png"
mime = "image/jpeg"  # or "image/png"
b64 = base64.b64encode(img_path.read_bytes()).decode()
data_uri = f"data:{mime};base64,{b64}"

# 3. Inject into HTML template as a Python variable — never via shell
html = f"""
<div style="position:relative;width:100%;height:100%;">
  <img src="{data_uri}"
       style="position:absolute;inset:0;width:100%;height:100%;object-fit:cover;z-index:0;">
  <div style="position:absolute;inset:0;background:rgba(255,255,255,0.35);z-index:1;"></div>
  <!-- slide content goes here, z-index:2 -->
</div>
"""

Path("/home/claude/carousel.html").write_text(html, encoding="utf-8")
```

### Image as slide background (most common use)

```html
<!-- Inside the slide div, before any content -->
<img src="{data_uri}"
     style="position:absolute;inset:0;width:100%;height:100%;object-fit:cover;z-index:0;">
<!-- Semi-transparent overlay so text stays readable -->
<div style="position:absolute;inset:0;background:rgba(255,255,255,0.35);z-index:1;"></div>
<!-- All slide content must have z-index:2 or higher -->
```

For dark slides, use `rgba(0,0,0,0.45)` as the overlay instead.

### Common image mistakes to avoid

| Mistake | What goes wrong | Fix |
|---------|----------------|-----|
| `<img src="gestante.png">` | Broken image — relative path only works if HTML and image share the same folder | Always use base64 `data:` URI |
| `background: url('data:...')` inline with 1.5MB base64 | Browser parser crash, 1.3M token context | Use `<img>` tag with `object-fit:cover` |
| Generating HTML via shell `echo` or heredoc | `$` and backtick characters in base64 get interpolated and corrupt the string | Always use Python `Path.write_text()` |
| Assuming `.png` extension = PNG format | File may actually be JPEG; wrong MIME type breaks rendering | Run `file` command to detect actual format |

---

## Step 2: Derive the Full Color System

From the user's **single primary brand color**, generate the full 6-token palette:

```
BRAND_PRIMARY   = {user's color}                    // Main accent — progress bar, icons, tags
BRAND_LIGHT     = {primary lightened ~20%}           // Secondary accent — tags on dark, pills
BRAND_DARK      = {primary darkened ~30%}            // CTA text, gradient anchor
LIGHT_BG        = {warm or cool off-white}           // Light slide background (never pure #fff)
LIGHT_BORDER    = {slightly darker than LIGHT_BG}    // Dividers on light slides
DARK_BG         = {near-black with brand tint}       // Dark slide background
```

**Rules for deriving colors:**
- LIGHT_BG: tinted off-white complementing the primary (warm → warm cream, cool → cool gray-white)
- DARK_BG: near-black with subtle brand tint (warm → #1A1918, cool → #0F172A)
- LIGHT_BORDER: always ~1 shade darker than LIGHT_BG
- Brand gradient: `linear-gradient(165deg, BRAND_DARK 0%, BRAND_PRIMARY 50%, BRAND_LIGHT 100%)`

---

## Step 3: Set Up Typography

Based on the user's font preference, pick a **heading font** and **body font** from Google Fonts.

| Style | Heading Font | Body Font |
|-------|-------------|-----------|
| Editorial / premium | Playfair Display | DM Sans |
| Modern / clean | Plus Jakarta Sans (700) | Plus Jakarta Sans (400) |
| Warm / approachable | Lora | Nunito Sans |
| Technical / sharp | Space Grotesk | Space Grotesk |
| Bold / expressive | Fraunces | Outfit |
| Classic / trustworthy | Libre Baskerville | Work Sans |
| Rounded / friendly | Bricolage Grotesque | Bricolage Grotesque |

**Font size scale (fixed across all brands):**
- Headings: 28–34px, weight 600, letter-spacing -0.3 to -0.5px, line-height 1.1–1.15
- Body: 14px, weight 400, line-height 1.5–1.55
- Tags/labels: 10px, weight 600, letter-spacing 2px, uppercase
- Step numbers: heading font, 26px, weight 300
- Small text: 11–12px

Apply via CSS classes `.serif` (heading font) and `.sans` (body font) throughout all slides.

---

## Slide 1 — Hook Rules

The first slide must stop the scroll in under 1 second. Prioritize these formats:

| Hook format | Example |
|---|---|
| Afirmação polêmica | "Você está usando IA errado" |
| Número + benefício | "7 ferramentas que substituem seu designer" |
| Pergunta que dói | "Por que seus carrosséis têm 0 salvamentos?" |
| Resultado concreto | "Esse post gerou 4.200 seguidores em 3 dias" |
| Inversão de expectativa | "Mais esforço no design = menos alcance" |

**Rules:**
- Never start with the brand name as headline
- Visual proof on Slide 1 whenever possible (screenshot, result, real number)
- Hook must promise value that the following slides deliver

---

## Slide Sequences

### Standard (7 slides — default)

| # | Type | Background | Purpose |
|---|------|------------|---------|
| 1 | Hero | LIGHT_BG | Hook — bold statement, logo lockup, optional watermark |
| 2 | Problem | DARK_BG | Pain point — what's broken, frustrating, or outdated |
| 3 | Solution | Brand gradient | The answer — what solves it, optional quote/prompt box |
| 4 | Features | LIGHT_BG | What you get — feature list with icons |
| 5 | Details | DARK_BG | Depth — customization, specs, differentiators |
| 6 | How-to | LIGHT_BG | Steps — numbered workflow or process |
| 7 | CTA | Brand gradient | Call to action — logo, tagline, CTA button. **No arrow. Full progress bar.** |

### Listicle (5–10 slides)

| # | Type | Background |
|---|------|------------|
| 1 | Hero | LIGHT_BG |
| 2–N | Item N | Alternating LIGHT/DARK |
| Last | CTA | Brand gradient |

Use for: "X ferramentas", "X erros", "X dicas"

### Tutorial (7 slides)

| # | Type | Background |
|---|------|------------|
| 1 | Hero | LIGHT_BG |
| 2 | Contexto / Por quê | DARK_BG |
| 3–5 | Passo 1, 2, 3 | Alternating |
| 6 | Resultado esperado | DARK_BG |
| 7 | CTA | Brand gradient |

### Comparação (5 slides)

| # | Type | Background |
|---|------|------------|
| 1 | Hero (o que será comparado) | LIGHT_BG |
| 2 | Opção A | LIGHT_BG |
| 3 | Opção B | DARK_BG |
| 4 | Veredicto | Brand gradient |
| 5 | CTA | DARK_BG |

**General rules for all sequences:**
- Start with a hook — first slide must stop the scroll
- End CTA on brand gradient — no swipe arrow, progress bar at 100%
- Alternate light and dark backgrounds for visual rhythm
- Adapt sequence to topic — not every carousel needs all slides

---

## Slide Architecture

### Format
- Aspect ratio: **4:5** (Instagram carousel standard)
- Each slide is self-contained — all UI elements baked into the image
- Alternate LIGHT_BG and DARK_BG backgrounds for visual rhythm

### Required Elements on Every Slide

#### 1. Progress Bar (bottom of every slide)

Shows position in the carousel. Fills as user swipes.

- Position: absolute bottom, full width, 28px horizontal padding, 20px bottom padding
- Track: 3px height, rounded corners
- Fill width: `((slideIndex + 1) / totalSlides) * 100%`
- Light slides: `rgba(0,0,0,0.08)` track, `BRAND_PRIMARY` fill, `rgba(0,0,0,0.3)` counter
- Dark slides: `rgba(255,255,255,0.12)` track, `#fff` fill, `rgba(255,255,255,0.4)` counter
- Counter label beside the bar: "1/7" format, 11px, weight 500

```javascript
function progressBar(index, total, isLightSlide) {
  const pct = ((index + 1) / total) * 100;
  const trackColor = isLightSlide ? 'rgba(0,0,0,0.08)' : 'rgba(255,255,255,0.12)';
  const fillColor = isLightSlide ? BRAND_PRIMARY : '#fff'; // use actual BRAND_PRIMARY value
  const labelColor = isLightSlide ? 'rgba(0,0,0,0.3)' : 'rgba(255,255,255,0.4)';
  return `<div style="position:absolute;bottom:0;left:0;right:0;padding:16px 28px 20px;z-index:10;display:flex;align-items:center;gap:10px;">
    <div style="flex:1;height:3px;background:${trackColor};border-radius:2px;overflow:hidden;">
      <div style="height:100%;width:${pct}%;background:${fillColor};border-radius:2px;"></div>
    </div>
    <span style="font-size:11px;color:${labelColor};font-weight:500;">${index + 1}/${total}</span>
  </div>`;
}
```

⚠️ **Important:** Always replace `BRAND_PRIMARY` with the actual hex value before rendering. Never leave it as a variable name in the HTML output.

#### 2. Swipe Arrow (right edge — every slide EXCEPT the last)

Subtle chevron guiding the user to keep swiping. Removed on the last slide.

- Position: absolute right, full height, 48px wide
- Background: gradient fade transparent → subtle tint
- Chevron: 24×24 SVG, rounded strokes
- Light slides: `rgba(0,0,0,0.06)` bg, `rgba(0,0,0,0.25)` stroke
- Dark slides: `rgba(255,255,255,0.08)` bg, `rgba(255,255,255,0.35)` stroke

```javascript
function swipeArrow(isLightSlide) {
  const bg = isLightSlide ? 'rgba(0,0,0,0.06)' : 'rgba(255,255,255,0.08)';
  const stroke = isLightSlide ? 'rgba(0,0,0,0.25)' : 'rgba(255,255,255,0.35)';
  return `<div style="position:absolute;right:0;top:0;bottom:0;width:48px;z-index:9;display:flex;align-items:center;justify-content:center;background:linear-gradient(to right,transparent,${bg});">
    <svg width="24" height="24" viewBox="0 0 24 24" fill="none">
      <path d="M9 6l6 6-6 6" stroke="${stroke}" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"/>
    </svg>
  </div>`;
}
```

---

## Reusable Components

### Strikethrough pills
```html
<span style="font-size:11px;padding:5px 12px;border:1px solid rgba(255,255,255,0.1);border-radius:20px;color:#6B6560;text-decoration:line-through;">{Old tool}</span>
```

### Tag pills
```html
<span style="font-size:11px;padding:5px 12px;background:rgba(255,255,255,0.06);border-radius:20px;color:{BRAND_LIGHT};">{Label}</span>
```

### Prompt / quote box
```html
<div style="padding:16px;background:rgba(0,0,0,0.15);border-radius:12px;border:1px solid rgba(255,255,255,0.08);">
  <p class="sans" style="font-size:13px;color:rgba(255,255,255,0.5);margin-bottom:6px;">{Label}</p>
  <p class="serif" style="font-size:15px;color:#fff;font-style:italic;line-height:1.4;">"{Quote text}"</p>
</div>
```

### Feature list
```html
<div style="display:flex;align-items:flex-start;gap:14px;padding:10px 0;border-bottom:1px solid {LIGHT_BORDER};">
  <span style="color:{BRAND_PRIMARY};font-size:15px;width:18px;text-align:center;">{icon}</span>
  <div>
    <span class="sans" style="font-size:14px;font-weight:600;color:{DARK_BG};">{Label}</span>
    <span class="sans" style="font-size:12px;color:#8A8580;">{Description}</span>
  </div>
</div>
```

### Numbered steps
```html
<div style="display:flex;align-items:flex-start;gap:16px;padding:14px 0;border-bottom:1px solid {LIGHT_BORDER};">
  <span class="serif" style="font-size:26px;font-weight:300;color:{BRAND_PRIMARY};min-width:34px;line-height:1;">01</span>
  <div>
    <span class="sans" style="font-size:14px;font-weight:600;color:{DARK_BG};">{Step title}</span>
    <span class="sans" style="font-size:12px;color:#8A8580;">{Step description}</span>
  </div>
</div>
```

### Color swatches
```html
<div style="width:32px;height:32px;border-radius:8px;background:{color};border:1px solid rgba(255,255,255,0.08);"></div>
```

### CTA button (final slide only)
```html
<div style="display:inline-flex;align-items:center;gap:8px;padding:12px 28px;background:{LIGHT_BG};color:{BRAND_DARK};font-family:'{BODY_FONT}',sans-serif;font-weight:600;font-size:14px;border-radius:28px;">
  {CTA text}
</div>
```

### Tag / Category Label
```html
<span class="sans" style="display:inline-block;font-size:10px;font-weight:600;letter-spacing:2px;color:{color};margin-bottom:16px;">{TAG TEXT}</span>
```
- Light slides: `BRAND_PRIMARY`
- Dark slides: `BRAND_LIGHT`
- Brand gradient slides: `rgba(255,255,255,0.6)`

### Logo Lockup (first and last slides)
- If logo icon: 40px circle (BRAND_PRIMARY bg) + icon centered + brand name beside
- If initials: 40px circle with first letter in white
- Brand name: 13px, weight 600, letter-spacing 0.5px

---

## Layout Rules

- Content padding: `0 36px` standard
- Bottom-aligned slides with progress bar: `0 36px 52px` to clear the bar
- **Hero/CTA slides:** `justify-content: center`
- **Content-heavy slides:** `justify-content: flex-end`
- **Content must never overlap the progress bar** — use `padding-bottom: 52px`

---

## Instagram Frame (Preview Wrapper)

When displaying in chat, wrap in an Instagram-style frame:

- **Header:** Avatar (BRAND_PRIMARY circle + logo) + handle + subtitle
- **Viewport:** 4:5 aspect ratio, swipeable/draggable track with all slides
- **Dots:** Small dot indicators below the viewport
- **Actions:** Heart, comment, share, bookmark SVG icons
- **Caption:** Handle + short description + "2 HOURS AGO" timestamp

Include pointer-based swipe/drag interaction for preview. Slides are still standalone export-ready images.

**Important:** `.ig-frame` must be exactly **420px wide**. The carousel viewport is 420×525px. Do NOT change this width — export depends on it.

---

## Review Flow

**Always follow this flow. Never skip to export without approval.**

1. Generate the HTML preview first — never jump directly to export
2. Show the preview and ask: **"Quais slides precisam de ajuste antes de exportar?"**
3. Fix only the mentioned slides — never regenerate the entire carousel unless the direction fundamentally changes
4. Only proceed to export when the user explicitly confirms approval (e.g., "pode exportar", "aprovado", "ok")

---

## Exporting Slides as Instagram-Ready PNGs

After the user approves the carousel preview, export each slide as an individual **1080×1350px PNG**.

### Critical Export Rules

1. **Use Python for HTML generation** — never use shell scripts with variable interpolation. Always use `Path.write_text()` or `open().write()`.

2. **Embed images as base64** — all user-uploaded images must be base64-encoded as `data:image/jpeg;base64,...` URIs. Check actual file format with the `file` command — a `.png` extension may contain a JPEG.

3. **Keep the 420px layout width** — use Playwright's `device_scale_factor` to scale up to 1080px output WITHOUT changing the layout viewport.

### Export Script

```python
import asyncio
from pathlib import Path
from playwright.async_api import async_playwright

INPUT_HTML = Path("/path/to/carousel.html")
OUTPUT_DIR = Path("/path/to/output/slides")
OUTPUT_DIR.mkdir(exist_ok=True)

TOTAL_SLIDES = 7  # Update to match your carousel

VIEW_W = 420
VIEW_H = 525
SCALE = 1080 / 420  # = 2.5714...

async def export_slides():
    async with async_playwright() as p:
        browser = await p.chromium.launch()
        page = await browser.new_page(
            viewport={"width": VIEW_W, "height": VIEW_H},
            device_scale_factor=SCALE,
        )

        html_content = INPUT_HTML.read_text(encoding="utf-8")
        await page.set_content(html_content, wait_until="networkidle")
        await page.wait_for_timeout(3000)  # Wait for Google Fonts to load

        # Hide IG frame chrome, show only the slide viewport
        await page.evaluate("""() => {
            document.querySelectorAll('.ig-header,.ig-dots,.ig-actions,.ig-caption')
                .forEach(el => el.style.display='none');

            const frame = document.querySelector('.ig-frame');
            frame.style.cssText = 'width:420px;height:525px;max-width:none;border-radius:0;box-shadow:none;overflow:hidden;margin:0;';

            const viewport = document.querySelector('.carousel-viewport');
            viewport.style.cssText = 'width:420px;height:525px;aspect-ratio:unset;overflow:hidden;cursor:default;';

            document.body.style.cssText = 'padding:0;margin:0;display:block;overflow:hidden;';
        }""")
        await page.wait_for_timeout(500)

        for i in range(TOTAL_SLIDES):
            await page.evaluate("""(idx) => {
                const track = document.querySelector('.carousel-track');
                track.style.transition = 'none';
                track.style.transform = 'translateX(' + (-idx * 420) + 'px)';
            }""", i)
            await page.wait_for_timeout(400)

            await page.screenshot(
                path=str(OUTPUT_DIR / f"slide_{i+1}.png"),
                clip={"x": 0, "y": 0, "width": VIEW_W, "height": VIEW_H}
            )
            print(f"Exported slide {i+1}/{TOTAL_SLIDES}")

        await browser.close()

asyncio.run(export_slides())
```

### Why This Works

- **`device_scale_factor=2.5714`** renders at high DPI — a 420px element becomes 1080px in the output. Layout stays at 420px.
- **`clip`** captures only the carousel viewport, not browser chrome.
- **`wait_for_timeout(3000)`** gives Google Fonts time to load.
- **`track.style.transition = 'none'`** disables swipe animation so slides snap instantly.

### Common Export Mistakes to Avoid

| Mistake | What goes wrong | Fix |
|---------|----------------|-----|
| Setting viewport to 1080×1350 | Layout reflows — fonts tiny, spacing breaks | Keep viewport at 420×525, use `device_scale_factor` |
| Using shell scripts to generate HTML | `$` signs and backticks get interpolated | Always use Python for HTML generation |
| Not waiting for fonts | Headings render in fallback system fonts | `wait_for_timeout(3000)` after page load |
| Not hiding IG frame chrome | Export includes header, dots, caption | Hide `.ig-header,.ig-dots,.ig-actions,.ig-caption` |
| Changing `.ig-frame` width | Entire layout shifts | Always keep at exactly 420px |
| Leaving `BRAND_PRIMARY` as variable name in CSS | Color renders as invalid / invisible | Always interpolate actual hex values into HTML |

---

## Design Principles

1. **Every slide is export-ready** — arrow and progress bar are part of the slide image
2. **Light/dark alternation** — creates visual rhythm across swipes
3. **Heading + body font pairing** — display font for impact, body for readability
4. **Brand-derived palette** — all colors stem from one primary, keeping everything cohesive
5. **Progressive disclosure** — progress bar fills and arrow guides forward
6. **Last slide is special** — no arrow, full progress bar, clear CTA
7. **Consistent components** — same tag style, list style, spacing across all slides
8. **Content padding clears UI** — body text never overlaps progress bar or arrow
9. **Hook-first copy** — Slide 1 exists to stop the scroll, not to introduce the brand
10. **Iterate fast** — show preview, fix specific slides, don't rebuild from scratch
