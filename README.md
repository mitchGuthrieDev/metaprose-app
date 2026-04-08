# Scaline Selects — Web App

A Svelte-based web clone inspired by musicforprogramming.net, built with a fully responsive 3-column layout, dynamic audio playback, an advanced ASCII audio visualizer, and a flexible color system with theme toggles.

---

## Features

- Episode browser & player with Markdown support for tracklist descriptions.
- Custom ASCII visualizer with color-coded amplitude display.
- Dynamic theming with:
  - **Invert mode** (light-on-dark / dark-on-light)
  - **Grayscale mode** (monochrome aesthetic)
- Audio seeking (+30/-30 sec controls).
- Fully responsive flexbox grid.

---

## Project Structure

### Main Components

- `+page.svelte` — core layout & logic
- `AsciiVisualizer.svelte` — advanced real-time audio visualizer
- `style.css` — global utility-based design system

### Supporting Files

- `episodes.json` — metadata for episodes
- `/pages/about.md` & `/pages/credits.md` — static Markdown content

---

## Design System (Utilities)

### Layout Utilities
| Class | Description |
|--|--|
| `full` | width 100% |
| `flex`, `row`, `col`, `wrap` | flexbox layout |
| `relative` | position relative |
| `noselect` | disable text selection |
| `hidden` | hide element |

### Spacing Utilities
| Class | Description |
|--|--|
| `pad` | padding: 1rem |
| `pad-topless` | padding: 0 top, 1rem sides & bottom |
| `px`, `pl-1`, `pt`, `pb-0`, `py`, `mb`, `y-space` | various shorthand padding/margin |

### Responsive Utilities
| Class | Breakpoint |
|--|--|
| `md:half` | >=768px: width 50% |
| `lg:quarter` | >=1024px: width 25% |
| `lg:screen-v-scroll` | full height, scrollable |

### Color Classes
| Class | Reference |
|--|--|
| `.black`, `.white`, `.grey`, `.purple`, `.fuschia`, `.blue`, `.green`, `.orange`, `.yellow` | Maps to CSS variables in palette |

### Theme Toggles
| Class | Mode |
|--|--|
| `body.invert` | inverted colors |

---

## Audio Visualizer Logic
- Uses `AnalyserNode` FFT with smoothing.
- Applies Fletcher-Munson equal-loudness curve for perceptual amplitude mapping.
- Converts amplitudes into vertical ASCII character stacks.
- Color-coded rows based on amplitude height.
- Resizes column count on window resize.

---

## Global Theme Palette

CSS variables automatically handle all color modes:

```css
body {
  --c-black, --c-white, --c-grey, --c-purple, --c-fuschia, --c-blue, --c-green, --c-orange, --c-yellow;
}
body.invert { ... }
body.grayscale { ... }
```

Color classes like `.green` dynamically follow active palette.

---

## Quick Start

```bash
npm install
npm run dev
```

---

## Future Ideas
- Episode shuffle/randomizer
- Track seeking via visualizer interaction
- Export visualizer as animated ASCII art
- Add persistent user preferences for theme modes

---

**Built using Svelte + pure CSS utilities.**
