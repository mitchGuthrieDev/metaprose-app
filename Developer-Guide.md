# Scaline Selects — Comprehensive Developer Reference Sheet

---

## 🔧 Project Overview

**Project Name:** Scaline Selects Web App  
**Tech Stack:** Svelte + Vanilla CSS + Pure Utility Classes

This project is a fully responsive, single-page web app inspired by the design of musicforprogramming.net, featuring:

- Multi-column grid layout (Left Sidebar, Main Content, Episodes Sidebar)
- Dynamic audio playback and visualization
- Custom ASCII audio visualizer
- Global color palette with Invert and Grayscale modes
- Markdown-based content rendering
- Fully utility-driven CSS architecture

---

## 🏗 Project Architecture

### 🔑 Main Files

| File | Purpose |
|--|--|
| `+page.svelte` | Main UI, routing, state management, audio playback, theme toggles |
| `AsciiVisualizer.svelte` | Advanced ASCII real-time audio visualizer |
| `style.css` | Global utility classes, layout, color system |
| `episodes.json` | Episode metadata |
| `/pages/about.md` / `/pages/credits.md` | Markdown for static content |

---

## 🎨 Global Color System

Color classes dynamically reference CSS variables under multiple modes:

| Class | Default | Invert Mode | Grayscale Mode |
|--|--|--|--|
| `black` | #1a1a1a | #d5d5d5 | #000000 |
| `white` | #e1e1e1 | #2a2830 | #ffffff |
| `grey` | #898989 | #7e7e7e | #999999 |
| `purple` | #b462ff | #6b1fa3 | #777777 |
| `fuschia` | #ff45b4 | #f50777 | #aaaaaa |
| `blue` | #18b6ff | #0b8cff | #888888 |
| `green` | #1beb9e | #00a76c | #bbbbbb |
| `orange` | #ff9528 | #dd7200 | #999999 |
| `yellow` | #dddd25 | #949700 | #cccccc |

### 🌗 Theme Control Classes

- `body` — default theme
- `body.invert` — invert mode
- `body.grayscale` — grayscale mode

These are toggled by UI buttons using simple class toggling on `document.body`.

---

## 🖼 Layout System

### Grid Breakdown

| Section | Width |
|--|--|
| Left Sidebar | `lg:quarter` (25%) |
| Main Content | `md:half` (50%) |
| Episodes Sidebar | `lg:quarter` (25%) |

### Responsive Utilities

| Class | Behavior |
|--|--|
| `md:half` | 50% width at >=768px |
| `lg:quarter` | 25% width at >=1024px |
| `lg:screen-v-scroll` | Full screen vertical scroll container |

### Utility Layout Classes

| Class | Purpose |
|--|--|
| `full` | 100% width |
| `flex` | Flex display |
| `col`, `row`, `wrap` | Flexbox directions |
| `relative` | Positioning |
| `hidden` | Display none |
| `noselect` | Disable text selection |
| `break-anywhere` | Force word breaks |

---

## 📏 Spacing Utilities

| Class | Purpose |
|--|--|
| `pad` | `padding: 1rem` |
| `pad-topless` | `padding: 0 1rem 1rem 1rem` (used for header alignment) |
| `px` | horizontal padding 1rem |
| `pl-1` | padding-left 0.25rem |
| `pt` | padding-top 1rem |
| `pt-0` | padding-top 0 |
| `pb-0` | padding-bottom 0 |
| `py` | padding vertical 1rem |
| `mb`, `mb-1`, `y-space` | margin bottom spacing |

---

## 🎛 Button System

| Class | Purpose |
|--|--|
| `btn` | Base button reset |
| `btn-audio` | Audio control buttons (green) |
| `btn-volume` | Navigation volume group (purple) |
| `btn-nav-about` | Blue about button |
| `btn-nav-credits` | Fuschia credits button |
| `btn-nav-rss` | Orange RSS button |
| `btn-nav-source` | Purple source button |
| `btn-invert` | Invert & grayscale toggle buttons |
| `btn-episode` | Episode list buttons (green + italic + truncation) |

---

## 🔊 Audio Playback Logic

- Controlled entirely via bound `audioEl`
- Seek logic: `seek(seconds)` clamps properly to valid time range
- Audio metadata fetched and file size displayed
- Markdown tracklist rendered via `marked()`
- Full episode switching handled via `loadEpisode(id)`

### Playback Buttons:
| Button | Action |
|--|--|
| `[prev]` | Load previous episode |
| `[-30]` | Seek back 30 sec |
| `[play]/[stop]` | Toggle play/pause |
| `[+30]` | Seek forward 30 sec |
| `[next]` | Load next episode |

---

## 🎹 AsciiVisualizer.svelte — Visualizer Architecture

- Web Audio API `AnalyserNode` w/ smoothing (`smoothingTimeConstant: 0.85`)
- FFT size: `1024` bins
- Fletcher-Munson weighting applied to each bin for perceptual amplitude
- Adaptive number of columns based on screen width (`window.innerWidth / 15`)
- Dynamic vertical rows (8 rows) with distinct ASCII characters:
  - Character Map: `[' ', '.', '·', 'o', '*', '°', '^', '#']`
  - Random peak characters: `['#', '@', '%', '$', '&', '*']`
- Each amplitude row colored using global color classes via mapped index
- Responsive recalculation on window resize
- Fully stateless: driven externally by bound `audioEl`

---

## 📦 Markdown Content Rendering

- About & Credits pages loaded from local Markdown files
- `marked()` parser used for HTML conversion
- Markdown content injected via {@html currentHtml}

---

## 📄 Developer Notes

- No external dependencies except `marked`
- Entire theme logic handled via pure CSS utility class toggles
- Easily extensible utility system for rapid adjustments
- Visualizer fully decoupled and extensible for future effects

---

✅ **Current State: Clean, production-ready build with flexible architecture for future expansion.**
