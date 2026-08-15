<div align="center">

# 🕌 Islamic Post Creator

**A Claude Skill for generating source-verified, beautifully typeset Islamic Instagram carousels — automatically.**

*Ayah / Hadith / Topic → verified text → designed slides → rendered PNGs → caption*

[![Claude Skill](https://img.shields.io/badge/Claude-Skill-6B1F2E?style=for-the-badge&logo=anthropic&logoColor=white)](https://claude.ai)
[![Playwright](https://img.shields.io/badge/Rendering-Playwright-2EAD33?style=for-the-badge&logo=playwright&logoColor=white)](https://playwright.dev)
[![License](https://img.shields.io/badge/License-MIT-blue?style=for-the-badge)](#LICENSE)
[![Made for](https://img.shields.io/badge/Made%20for-@ibnesaed-c9a86a?style=for-the-badge)](https://instagram.com/ibnesaed)
[![Made by](https://img.shields.io/badge/Made%20by-@iabdullahahmad-c9a86a?style=for-the-badge&logo=instagram&logoColor=white)](https://instagram.com/iabdullahahmad)

</div>

---

## ✨ What this is

**Islamic Post Creator** is a [Claude Skill](https://www.anthropic.com/news/skills) — a set of instructions and tooling that Claude follows automatically whenever you ask for Islamic Instagram content. Instead of hand-typing hadith text into a design tool and hoping it's accurate, this skill runs a full pipeline:

```
┌─────────────┐     ┌──────────────┐     ┌───────────────┐     ┌────────────┐     ┌─────────┐
│   Intent     │ →   │   Source      │ →   │    Design      │ →   │  Render     │ →   │ Caption │
│  Capture     │     │ Verification  │     │    System      │     │  (PNG)      │     │  + Tags │
└─────────────┘     └──────────────┘     └───────────────┘     └────────────┘     └─────────┘
```

No manual copy-pasting of ayahs. No unverified hadith. No broken Urdu typography. No inconsistent branding between posts.

---

## 🎯 Why it exists

Most AI-generated Islamic content has three recurring problems — this skill was built specifically to solve each one:

| Problem | How this skill solves it |
|---|---|
| **Misquoted or misattributed hadith/ayahs** | Every source is cross-checked live against Quran.com and Sunnah.com before it ever touches a slide |
| **Broken Arabic/Urdu rendering** | Dedicated typography reference covers RTL handling, Nastaliq line-height, font stacks, and common clipping bugs |
| **Inconsistent visuals across posts** | Design tokens are established once per account (`@ibnesaed` by default) and reused across every future post |

---

## 🧩 How it works

### 1️⃣ Capture Intent
Claude figures out (from your message, or one short round of questions):
- **Content type** — Quran ayah, hadith, seerah/story, or general topic
- **Language** — Urdu, English, or bilingual
- **Slide count** — typically 3–6 for a single ayah/hadith, up to ~10 for topical lists
- **Visual style** — reused from prior posts, or picked fresh
- **Account** — defaults to `@ibnesaed`

### 2️⃣ Source Verification *(mandatory — never skipped)*
| Source type | Verified against |
|---|---|
| Quran ayah | [Quran.com](https://quran.com) / [corpus.quran.com](https://corpus.quran.com) — Surah, ayah number, Arabic text, translation |
| Hadith | [Sunnah.com](https://sunnah.com) — Bukhari, Muslim, Abu Dawud, Tirmidhi, Nasa'i, Ibn Majah — wording, narrator chain, authenticity grade |

If a hadith can't be verified, or its grading is weak/disputed, **you're told explicitly** rather than the skill silently presenting it as authentic. Even user-supplied text gets cross-checked — paraphrased hadith circulate widely on social media, and this skill won't propagate them.

### 3️⃣ Design System
A token set (`--bg`, `--accent`, `--text`, fonts) is defined once and reused across every slide in the post — and across future posts for the same account. Three baseline directions are offered when nothing exists yet:

- 🟢 **Reverent / Classic** — deep green or maroon, gold accents, geometric borders
- ⚫ **Modern Minimal** — dark neutral background, single accent, generous whitespace
- 🎨 **Existing brand system** — reused from prior `@ibnesaed` content if one already exists

### 4️⃣ Render (HTML/CSS → PNG)
Each slide is written as HTML/CSS and rendered via a bundled **Playwright** script:

```bash
python3 scripts/render_slide.py input.html output.png --width 1080 --height 1350
```

- Standard Instagram carousel canvas: **1080×1350px** (4:5), or **1080×1080px** square
- **2x device scale factor** for crisp, high-DPI export
- Waits on `document.fonts.ready` before capturing — so Nastaliq/Amiri never silently fall back to a system font

### 5️⃣ Caption
A caption is drafted to match the post's language(s): hook line, reflection, full citation, call-to-action, and 5–15 hashtags mixing broad and niche tags.

### 6️⃣ Deliver
All slide PNGs are handed over together as downloadable files, with the caption presented inline.

---

## 🔤 Typography

Correct Arabic and Urdu rendering is the single most common failure point in AI-generated Islamic graphics. This skill handles it explicitly:

| Text type | Font stack | Notes |
|---|---|---|
| **Arabic (Quran)** | `Amiri`, `KFGQPC Uthmanic Script HAFS` | `direction: rtl` |
| **Urdu (Nastaliq)** | `Jameel Noori Nastaleeq` → falls back to `Noto Nastaliq Urdu` | needs `line-height: ~2.1` — Nastaliq breaks under tight leading |
| **English / Latin** | Whatever the design system specifies | no special handling needed |

> 💡 **Custom fonts:** `Jameel Noori Nastaleeq` isn't on Google Fonts. Drop a local font file into the project and the skill will use it as primary instead of the `Noto Nastaliq Urdu` fallback.

Full pitfalls (clipped diacritics, RTL scoping on mixed content, kashida justification, Eastern vs. Western numerals) are documented in [`references/typography.md`](./references/typography.md).

---

## 🔗 Connected skills

This skill doesn't try to do everything itself — it calls out to other skills for decisions they're better suited for:

| Skill | Used for |
|---|---|
| `ui-ux-pro-max` | Color palette & font pairing selection |
| `design-system` | Token architecture for ongoing multi-post series |
| `brand` | Establishing/checking `@ibnesaed`'s visual & voice identity |
| `banner-design` / `design` | Heavier multi-concept or multi-platform social image workflows |
| `seo-geo-aeo` | Optional — caption/keyword discoverability if content is repurposed for a blog or website |

Not every skill fires on every run — only whichever actually resolves the decision in front of Claude at that step.

---

## 📁 Project structure

```
islamic-post-creator/
├── SKILL.md                          # Core instructions (the skill itself)
├── README.md                         # You are here
├── references/
│   ├── typography.md                 # Arabic/Urdu font stacks, RTL rules, common bugs
│   └── design-system-tokens.md       # Token schema + 3 baseline style directions
└── scripts/
    └── render_slide.py               # Playwright HTML → PNG renderer
```

---

## ⚙️ Settings & defaults

| Setting | Default | Configurable? |
|---|---|---|
| Target account | `@ibnesaed` | ✅ specify a different account in your request |
| Slide canvas | `1080×1350px` (4:5) | ✅ `--width` / `--height` flags, or ask for `1080×1080` square |
| Export scale | `2x` device scale factor | ✅ `--scale` flag |
| Capture mode | Fixed viewport | ✅ `--full-page` flag for scrollable content |
| Source verification | **Mandatory, never skipped** | ❌ by design |
| Slide count | 3–6 (single ayah/hadith), up to ~10 (topical) | ✅ specify any count |
| Language | Urdu / English / bilingual | ✅ specify per post |
| Design system | Reused per account once established | ✅ override anytime by requesting a new style |

### `render_slide.py` CLI reference

```bash
python3 scripts/render_slide.py <input.html> <output.png> [options]

Options:
  --width INT       Viewport width in px       (default: 1080)
  --height INT       Viewport height in px       (default: 1350)
  --scale FLOAT      Device scale factor          (default: 2.0)
  --full-page         Capture full scrollable page instead of fixed viewport
```

---

## 🚀 Installation

1. Download [`islamic-post-creator.skill`](./islamic-post-creator.skill) (or clone this repo)
2. In Claude, open **Settings → Skills** (or drop the `.skill` file into a chat) and click **Save skill**
3. Ask Claude for an Islamic post, carousel, or hadith graphic — the skill triggers automatically, no need to invoke it by name

**Requirements** (already available in Claude's environment — nothing to install yourself):
- Python 3
- [Playwright](https://playwright.dev/python/) with Chromium

---

## 📋 Example prompts that trigger this skill

- *"Make an Islamic carousel from this hadith: [paste text]"*
- *"Create a post about sabr for @ibnesaed"*
- *"Turn Surah Al-Baqarah 2:286 into an Instagram carousel"*
- *"5 duas for anxiety, Urdu carousel"*
- *"Continue the hadith series, part 3"*

---

## 🛡️ A note on accuracy

This skill treats **source verification as non-negotiable**. Every ayah and hadith is checked live against Quran.com and Sunnah.com before any slide is designed — including text you paste in yourself. If something can't be verified or its authenticity grading is weak, you'll be told plainly rather than the post shipping anyway. Islamic content carries real credibility weight; this skill is built to respect that.

## 🤝 Contributors

<div align="center">

| [<img src="https://github.com/iabdullahahmad.png" width="80" style="border-radius:50%"><br><sub><b>Abdullah Ahmad</b></sub>](https://github.com/iabdullahahmad) | [<img src="https://github.com/ibnesaed.png" width="80" style="border-radius:50%"><br><sub><b>ibnesaed</b></sub>](https://github.com/ibnesaed) |
|:---:|:---:|
| Creator & maintainer | Account this skill is built for |

</div>

---

<div>

Made with 🤍 by [**Abdullah Ahmad**](https://instagram.com/iabdullahahmad) ([@iabdullahahmad](https://instagram.com/iabdullahahmad))

</div>
