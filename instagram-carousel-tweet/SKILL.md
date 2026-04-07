---
name: instagram-tweet-carousel
description: >
  Creates high-quality Instagram carousels in an X (Twitter) post visual format, delivered as swipeable HTML previews with export-ready slides (1080×1350px PNG). Handles the full workflow: brand setup, slide copy, X-style visual design (profile header, clean background, X engagement bar), HTML generation, and Playwright-based export. Use this skill whenever the user asks to create, design, or generate an Instagram carousel in tweet-style or X-style format (e.g., "tweet-style carousel", "X-style carousel", "carrossel estilo tweet", "carrossel estilo X"), including requests for carrossel, Instagram slides, or any multi-image Instagram post — even if they don't explicitly mention "carousel" or "skill". Also trigger for requests like "create a post with multiple slides", "make a tweet-style/X-style carousel", or "export slides for Instagram" in this format.
---

# Instagram Tweet-style Carousel Generator (X / Twitter Layout)

Generates fully self-contained, swipeable HTML carousels where every slide mimics
an authentic X (Twitter) post — designed to be exported as individual 1080×1350px PNGs for Instagram.

---

## Working Directory Rules

**All file operations MUST use the current working directory (cwd) as the base. Never use hardcoded absolute paths like `C:\Users\marco\`.**

- Detect cwd at the start of every session using `pwd` or `os.getcwd()`
- Save all generated files (HTML, PNGs) relative to cwd
- The `ASSETS/` folder is always `{cwd}/ASSETS/`
- The `.claude/launch.json` is always `{cwd}/.claude/launch.json`
- The export output folder is always `{cwd}/slides/`
- Never scan directories outside of cwd for project files or images

---

## Step 0 — Ask Background Preference First

**Before collecting any other details, always ask:**

> "Você prefere o fundo **claro** (branco, estilo X padrão) ou **escuro** (preto, estilo X dark mode)?"

This choice determines the entire color system and font colors. Do not proceed until the user answers.

---

## Step 1: Collect Brand Details

After background preference is confirmed, ask for:

1. **Display name** — shown in the post header (e.g., "Marco Lang")
2. **X handle** — shown below the name (e.g., @marcolang)
3. **Verified badge** — blue checkmark yes/no (default: yes)
4. **Profile photo** — before asking the user anything about the photo, search in this order:
   1. **`{cwd}/assets/`** — check the project folder first (case-insensitive: `assets/` or `ASSETS/`)
   2. **Skill folder `assets/`** — if not found above, check the skill's own assets folder at `~/.claude/skills/instagram-carousel-tweet/assets/`
   - Use the Glob tool or `ls` to list files in each location
   - If an image file is found (JPG, PNG, WEBP) in either location, use it automatically and tell the user which file was picked — do not ask
   - Only ask the user to provide a photo if no image is found in either location
5. **Content / topic** — what the carousel is about
6. **Idioma dos slides** — default: **Português (BR)** unless specified otherwise
7. **Number of slides** — default: 7

If the user provides a website URL or brand assets, derive name and handle from those.

If the user says "make me a carousel about X" without brand details, ask before generating. Don't assume defaults.

---

## Handling User-Provided Images

**This section applies from the very first HTML generation — not only during export.**

The profile photo will be located inside the `ASSETS/` folder in the working directory. Always:

1. Find the file in `ASSETS/` (e.g., `ASSETS/foto.jpg`, `ASSETS/profile.png`)
2. Check the actual file format with the `file` command — extension may lie
3. Embed as base64 `data:` URI — never use relative paths

### ⚠️ Critical Rules

1. **NEVER use relative paths** (`ASSETS/foto.jpg`) — they break in every browser context except the exact folder the HTML lives in.
2. **NEVER use `background: url(filepath)`** — leads to 1.5MB+ base64 inline strings that crash the browser parser.
3. **ALWAYS embed as base64 `data:` URI** — works in preview, export, and any environment.
4. **ALWAYS generate the HTML via Python** (`Path.write_text()`) — shell heredocs interpolate `$` and backticks, corrupting base64 strings.

### Step-by-step: embed the profile photo

```bash
# 1. Check the actual file format (extension may lie)
file ASSETS/profile.jpg
```

```python
import base64
from pathlib import Path

# 2. Read and encode
img_path = Path("ASSETS/profile.jpg")  # adjust filename
# Use "image/jpeg" if `file` command says JPEG, else "image/png"
mime = "image/jpeg"  # or "image/png"
b64 = base64.b64encode(img_path.read_bytes()).decode()
avatar_uri = f"data:{mime};base64,{b64}"

# 3. Inject into HTML template as a Python variable — never via shell
html = f"""...<img src="{avatar_uri}" style="width:100%;height:100%;object-fit:cover;">..."""

Path("carousel.html").write_text(html, encoding="utf-8")
```

### Common image mistakes to avoid

| Mistake | What goes wrong | Fix |
|---------|----------------|-----|
| `<img src="ASSETS/foto.jpg">` | Broken image in browser | Always use base64 `data:` URI |
| `background: url('data:...')` inline with 1.5MB base64 | Browser parser crash | Use `<img>` tag with `object-fit:cover` |
| Generating HTML via shell heredoc | `$` and backtick chars corrupt base64 | Always use Python `Path.write_text()` |
| Assuming `.png` extension = PNG | Wrong MIME type breaks rendering | Run `file` command first |

---

## Step 2: Color System

Two modes only — no gradients, no brand-derived palettes. The X aesthetic is deliberately minimal.

### Light Mode (fundo claro)

```
SLIDE_BG        = #FFFFFF          // Slide background
TEXT_PRIMARY    = #0F1419          // Display name, tweet body text
TEXT_MUTED      = #536471          // Handle, timestamps, muted labels
DIVIDER         = #EFF3F4          // Separator lines, borders
ICON_COLOR      = #536471          // Engagement icons
X_BLUE          = #1D9BF0          // Verified badge, links, accent
SLIDE_NUM_BG    = rgba(0,0,0,0.06) // Slide counter pill background
SLIDE_NUM_TEXT  = #536471          // Slide counter text
```

### Dark Mode (fundo escuro)

```
SLIDE_BG        = #000000          // Slide background
TEXT_PRIMARY    = #E7E9EA          // Display name, tweet body text
TEXT_MUTED      = #71767B          // Handle, timestamps, muted labels
DIVIDER         = #2F3336          // Separator lines, borders
ICON_COLOR      = #71767B          // Engagement icons
X_BLUE          = #1D9BF0          // Verified badge, links, accent
SLIDE_NUM_BG    = rgba(255,255,255,0.08) // Slide counter pill background
SLIDE_NUM_TEXT  = #71767B          // Slide counter text
```

---

## Step 3: Typography

X uses a system font stack. Use **Inter** from Google Fonts as the closest equivalent.

```html
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
```

**Font size scale:**
- Display name: 16px, weight 700, color `TEXT_PRIMARY`
- Handle: 15px, weight 400, color `TEXT_MUTED`
- Tweet body: 20px, weight 400, line-height 1.55, color `TEXT_PRIMARY`
  _(Large text makes each slide readable at a glance — key for Instagram carousels)_
- Engagement counts: 13px, weight 400, color `TEXT_MUTED`
- Slide counter: 13px, weight 500

Apply via CSS class `.x-font` using `font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif`.

---

## Slide Structure (X Post Layout)

Every slide is an exact replica of an X post. Structure from top to bottom:

```
┌─────────────────────────────────────┐  ← SLIDE_BG background
│  [Avatar] Display Name ✓            │  ← Profile header (always on every slide)
│           @handle                   │
├─────────────────────────────────────┤  ← thin divider (DIVIDER color)
│                                     │
│   Tweet body text goes here.        │  ← Content area (large, readable)
│   It can span multiple lines        │
│   and fill most of the slide.       │
│                                     │
├─────────────────────────────────────┤  ← thin divider
│  💬 Reply  🔁 Repost  ♡ Like  📊   │  ← X engagement bar (decorative)
│                              [1/7]  │  ← Slide counter (right side)
└─────────────────────────────────────┘
```

### Profile Header

```html
<div class="x-header" style="display:flex;align-items:center;gap:12px;padding:20px 20px 14px;">
  <!-- Avatar: circular, 48px, profile photo -->
  <div style="width:48px;height:48px;border-radius:50%;overflow:hidden;flex-shrink:0;">
    <img src="{avatar_uri}" style="width:100%;height:100%;object-fit:cover;">
  </div>
  <!-- Name + handle -->
  <div style="display:flex;flex-direction:column;gap:1px;">
    <div style="display:flex;align-items:center;gap:4px;">
      <span class="x-font" style="font-size:16px;font-weight:700;color:{TEXT_PRIMARY};">{Display Name}</span>
      <!-- Verified badge (if enabled) -->
      <svg width="18" height="18" viewBox="0 0 24 24" fill="{X_BLUE}">
        <path d="M22.25 12c0-1.43-.88-2.67-2.19-3.34.46-1.39.2-2.9-.81-3.91s-2.52-1.27-3.91-.81c-.66-1.31-1.91-2.19-3.34-2.19s-2.67.88-3.33 2.19c-1.4-.46-2.91-.2-3.92.81s-1.26 2.52-.8 3.91c-1.31.67-2.2 1.91-2.2 3.34s.89 2.67 2.2 3.34c-.46 1.39-.21 2.9.8 3.91s2.52 1.26 3.91.81c.67 1.31 1.91 2.19 3.34 2.19s2.68-.88 3.34-2.19c1.39.45 2.9.2 3.91-.81s1.27-2.52.81-3.91c1.31-.67 2.19-1.91 2.19-3.34zm-11.71 4.2L6.8 12.46l1.41-1.42 2.26 2.26 4.8-5.23 1.47 1.36-6.2 6.77z"/>
      </svg>
    </div>
    <span class="x-font" style="font-size:15px;font-weight:400;color:{TEXT_MUTED};">@{handle}</span>
  </div>
</div>
<!-- Thin divider below header -->
<div style="height:1px;background:{DIVIDER};margin:0 20px;"></div>
```

### Content Area

```html
<div class="x-content" style="flex:1;padding:18px 20px 16px;display:flex;align-items:flex-start;">
  <p class="x-font" style="font-size:20px;font-weight:400;line-height:1.55;color:{TEXT_PRIMARY};margin:0;">
    {Tweet text for this slide}
  </p>
</div>
```

**Content rules:**
- Text should feel like a natural tweet — direct, punchy, no corporate speak
- Each slide = one thought / one point
- First slide is the hook (question, bold statement, number)
- Last slide is the CTA (follow, save, share)
- Do NOT repeat the same intro phrase on every slide

### Engagement Bar + Slide Counter

```html
<!-- Thin divider above engagement bar -->
<div style="height:1px;background:{DIVIDER};margin:0 20px;"></div>
<!-- Engagement bar -->
<div class="x-engagement" style="display:flex;align-items:center;padding:12px 20px 20px;gap:0;">
  <!-- Reply -->
  <div style="display:flex;align-items:center;gap:6px;flex:1;">
    <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="{ICON_COLOR}" stroke-width="1.8">
      <path d="M21 15a2 2 0 0 1-2 2H7l-4 4V5a2 2 0 0 1 2-2h14a2 2 0 0 1 2 2z"/>
    </svg>
    <span class="x-font" style="font-size:13px;color:{TEXT_MUTED};">{reply_count}</span>
  </div>
  <!-- Repost -->
  <div style="display:flex;align-items:center;gap:6px;flex:1;">
    <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="{ICON_COLOR}" stroke-width="1.8">
      <path d="M17 1l4 4-4 4"/><path d="M3 11V9a4 4 0 0 1 4-4h14"/><path d="M7 23l-4-4 4-4"/><path d="M21 13v2a4 4 0 0 1-4 4H3"/>
    </svg>
    <span class="x-font" style="font-size:13px;color:{TEXT_MUTED};">{repost_count}</span>
  </div>
  <!-- Like -->
  <div style="display:flex;align-items:center;gap:6px;flex:1;">
    <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="{ICON_COLOR}" stroke-width="1.8">
      <path d="M20.84 4.61a5.5 5.5 0 0 0-7.78 0L12 5.67l-1.06-1.06a5.5 5.5 0 0 0-7.78 7.78l1.06 1.06L12 21.23l7.78-7.78 1.06-1.06a5.5 5.5 0 0 0 0-7.78z"/>
    </svg>
    <span class="x-font" style="font-size:13px;color:{TEXT_MUTED};">{like_count}</span>
  </div>
  <!-- Views -->
  <div style="display:flex;align-items:center;gap:6px;flex:1;">
    <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="{ICON_COLOR}" stroke-width="1.8">
      <path d="M1 12s4-8 11-8 11 8 11 8-4 8-11 8-11-8-11-8z"/><circle cx="12" cy="12" r="3"/>
    </svg>
    <span class="x-font" style="font-size:13px;color:{TEXT_MUTED};">{view_count}</span>
  </div>
  <!-- Slide counter pill (right-aligned) -->
  <div style="margin-left:auto;padding:4px 10px;background:{SLIDE_NUM_BG};border-radius:20px;">
    <span class="x-font" style="font-size:13px;font-weight:500;color:{SLIDE_NUM_TEXT};">{i+1}/{total}</span>
  </div>
</div>
```

**Engagement numbers:** Use realistic-looking but fictional numbers. Example: 847 replies, 2.3K reposts, 18.4K likes, 412K views. Vary per slide.

---

## Slide Sequences

### Standard Thread (7 slides — default)

| # | Type | Content |
|---|------|---------|
| 1 | Hook | Bold statement, question, or surprising number that stops the scroll |
| 2 | Problem | Pain point or context — why this matters |
| 3 | Insight 1 | First key point |
| 4 | Insight 2 | Second key point |
| 5 | Insight 3 | Third key point |
| 6 | Deep dive | Most detailed / surprising revelation |
| 7 | CTA | Follow, save, share — clear action |

### Listicle Thread (5–10 slides)

| # | Content |
|---|---------|
| 1 | Hook — "X coisas que..." |
| 2–N | One item per slide, numbered |
| Last | CTA |

### Tutorial Thread (6 slides)

| # | Content |
|---|---------|
| 1 | Hook — what will be taught |
| 2 | Why this matters |
| 3–5 | Steps 1, 2, 3 |
| 6 | CTA |

**All sequences:**
- Every slide has the exact same X post header (avatar + name + handle)
- Same background throughout (no alternating dark/light)
- Hook on slide 1 is mandatory
- CTA on last slide is mandatory

---

## Slide Architecture

### Format
- Aspect ratio: **4:5** (Instagram carousel standard)
- Canvas: 420×525px in HTML preview (exported at 1080×1350px)
- Every slide: same background color (SLIDE_BG), no gradients, no alternating

### No Progress Bar

X posts don't have progress bars. Replace with the slide counter pill inside the engagement bar (see above). This is the only slide position indicator.

### No Swipe Arrow

X posts don't have decorative arrows. Omit entirely.

---

## Preview Wrapper (X Thread Frame)

When displaying in chat, wrap in an X-style post frame:

```
┌─────────────────────────────────────┐
│  X  [Search...]           🔔 👤     │  ← X app top bar (simplified)
├─────────────────────────────────────┤
│  ← Back    Post                     │  ← Post header bar
├─────────────────────────────────────┤
│  [Swipeable carousel viewport]      │  ← 420×525px
├─────────────────────────────────────┤
│  ● ● ○ ○ ○ ○ ○                     │  ← Dot indicators
└─────────────────────────────────────┘
```

- **Frame width:** exactly **420px** — never change this, export depends on it
- **Carousel viewport:** 420×525px
- **Top bar:** dark or light matching the slide background; X logo (bird/X icon), search bar, bell, profile icons
- **Dots:** small dot indicators below the viewport, styled to match X's minimalism
- **No caption area** — X posts don't have Instagram-style captions
- Include pointer-based swipe/drag interaction for preview

**Class names to use:**
- `.x-frame` — outer wrapper (420px wide)
- `.x-topbar` — app top bar
- `.carousel-viewport` — 420×525px clip area
- `.carousel-track` — flex row of all slides
- `.x-dots` — dot indicators
- Each slide: `.x-slide`

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
2. **Embed images as base64** — profile photo must be base64-encoded as `data:image/jpeg;base64,...` URI. Check actual file format with the `file` command.
3. **Keep the 420px layout width** — use Playwright's `device_scale_factor` to scale up to 1080px output WITHOUT changing the layout viewport.

### Install Playwright (only if needed)

```bash
python3 -c "import playwright" 2>/dev/null || pip3 install playwright
python3 -c "from playwright.sync_api import sync_playwright; sync_playwright().__enter__().chromium" 2>/dev/null || python3 -m playwright install chromium
```

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

        # Hide X frame chrome, show only the slide viewport
        await page.evaluate("""() => {
            document.querySelectorAll('.x-topbar,.x-dots')
                .forEach(el => el.style.display='none');

            const frame = document.querySelector('.x-frame');
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

### Common Export Mistakes to Avoid

| Mistake | What goes wrong | Fix |
|---------|----------------|-----|
| Setting viewport to 1080×1350 | Layout reflows — fonts tiny, spacing breaks | Keep viewport at 420×525, use `device_scale_factor` |
| Using shell scripts to generate HTML | `$` signs and backticks get interpolated | Always use Python for HTML generation |
| Not waiting for fonts | Headings render in fallback system fonts | `wait_for_timeout(3000)` after page load |
| Not hiding X frame chrome | Export includes topbar and dots | Hide `.x-topbar,.x-dots` |
| Changing `.x-frame` width | Entire layout shifts | Always keep at exactly 420px |
| Using relative path for avatar | Broken image in every context | Always embed as base64 `data:` URI |

---

## Design Principles

1. **Authenticity first** — every slide must look like a real X post, not a designed graphic
2. **Same background throughout** — no alternating colors; X threads are visually consistent
3. **Large body text** — 20px for tweet content ensures readability after Instagram export
4. **Profile header on every slide** — users scrolling Instagram must always see who posted
5. **Fictional but realistic engagement numbers** — adds social proof and authenticity
6. **Hook-first copy** — Slide 1 is the scroll-stopper; every subsequent slide must earn the swipe
7. **One thought per slide** — each tweet is a standalone idea, not a continuation mid-sentence
8. **CTA is always last** — follow, save, share — one clear action
9. **Iterate fast** — show preview, fix specific slides, don't rebuild from scratch
10. **Dark or light — never mixed** — respect the user's mode choice across all slides
