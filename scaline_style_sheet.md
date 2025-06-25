# SCALINE-STYLE-SHEET

> **Reference Guide for Utility Classes**  
> For the ScalineSelects Svelte Web App Layout

---

## Layout / Sizing

| Class         | Description                                     |
| ------------- | ----------------------------------------------- |
| `full`        | `width: 100%`                                   |
| `h-min-screen`| `min-height: 100vh`                             |
| `relative`    | `position: relative`                            |

---

## Flexbox Helpers

| Class         | Description           |
| ------------- | --------------------- |
| `flex`        | `display: flex`       |
| `col`         | `flex-direction: column` |
| `row`         | `flex-direction: row` |
| `wrap`        | `flex-wrap: wrap`     |
| `center`      | `justify-content: center; align-items: center` |

---

## Padding & Margin Utilities

| Class    | Description                        |
| -------- | ----------------------------------- |
| `pad`    | `padding: 1rem` (all sides)        |
| `px`     | `padding-left/right: 1rem`         |
| `pl-1`   | `padding-left: 0.25rem`            |
| `pr-0`   | `padding-right: 0`                 |
| `pt`     | `padding-top: 1rem`                |
| `pt-0`   | `padding-top: 0`                   |
| `pb-0`   | `padding-bottom: 0`                |
| `py`     | `padding-top/bottom: 1rem`         |
| `mb`     | `margin-bottom: 1rem`              |
| `y-space`| `margin-bottom: 2rem`              |

---

## Responsive Breakpoints

| Class           | Description (Applied on Min Width) |
| --------------- | ----------------------------------- |
| `md:half`        | `width: 50%` (>= 768px)           |
| `lg:quarter`     | `width: 25%` (>= 1024px)          |
| `lg:screen-v-scroll` | `overflow-y: auto; height: 100vh` |
| `md:hidden`      | `display: none` (>= 768px)        |
| `lg:hidden`      | `display: none` (>= 1024px)       |
| `md:flex`        | `display: flex` (>= 768px)        |
| `lg:flex`        | `display: flex` (>= 1024px)       |

---

## Colors

All colors are controlled by CSS variables in `body { ... }`.

| Class     | CSS Variable Used |
| --------- | ------------------ |
| `black`   | `var(--c-black)`   |
| `white`   | `var(--c-white)`   |
| `grey`    | `var(--c-grey)`    |
| `purple`  | `var(--c-purple)`  |
| `fuschia` | `var(--c-fuschia)` |
| `blue`    | `var(--c-blue)`    |
| `green`   | `var(--c-green)`   |
| `orange`  | `var(--c-orange)`  |
| `yellow`  | `var(--c-yellow)`  |

---

## Text / Type Utilities

| Class           | Description                |
| --------------- | -------------------------- |
| `large`         | Enlarged title font size   |
| `title`         | Applied for large titles   |
| `noselect`      | `user-select: none`        |
| `break-anywhere`| Allow text to wrap anywhere |

---

## Hover / Interaction

| Class         | Description                |
| ------------- | -------------------------- |
| `hover`       | Color + box-shadow hover effect |
| `subtle-hover`| Slight opacity fade hover  |

---

## Visibility

| Class    | Description          |
| -------- | -------------------- |
| `hidden` | `display: none`      |

---

## Buttons (Custom Color Buttons)

| Button Class     | Description         |
| ---------------- | ------------------- |
| `btn`            | Base button styling |
| `btn-episode`    | Episode buttons     |
| `btn-audio`      | Play/pause buttons  |
| `btn-volume`     | Volume/random buttons |
| `btn-source`     | Source download link |
| `btn-nav-about`  | About nav button    |
| `btn-nav-credits`| Credits nav button  |
| `btn-nav-rss`    | RSS nav button      |
| `btn-nav-link-1` | Extra nav button    |

---

## Notes:

- This project uses a fully **utility-first CSS system**.
- Classes are composable. Multiple classes are stacked in your markup.
- Responsive classes use Tailwind-style syntax: `md:*` / `lg:*`.
- Colors come entirely from your CSS variable palette.

---

**Always build layout via utility stacking — avoid hardcoded IDs like `#left`, `#content`, `#episodes`.**

This reference will evolve as you add more utilities!

