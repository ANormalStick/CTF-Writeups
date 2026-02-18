<style> :root { color-scheme: dark; --bg: #020617; --fg: #e5e7eb; --fg-muted: #9ca3af; --accent: #38bdf8; --border-subtle: #1f2937; } /* Base page */ html, body { margin: 0; padding: 0; background: radial-gradient(circle at top left, #020617, #020617 40%, #020617 100%); color: var(--fg); font-family: system-ui, -apple-system, BlinkMacSystemFont, "SF Pro Text", "Segoe UI", sans-serif; } /* GitHub Pages wrappers */ .page-content, .wrapper, article, .post { background: transparent !important; max-width: 960px; margin: 0 auto; padding: 2.5rem 1.5rem 4rem; } /* Headings */ .post h1, .page-content h1, article h1 { font-size: 1.6rem; margin-bottom: 0.6rem; } .post h2, .page-content h2, article h2 { font-size: 1.1rem; margin-top: 1.8rem; margin-bottom: 0.6rem; color: var(--fg-muted); } /* Tables (optional, if you use them) */ table { border-collapse: collapse; width: 100%; font-size: 0.85rem; margin: 0.4rem 0 0.8rem; border-radius: 0.6rem; overflow: hidden; } th, td { padding: 0.5rem 0.75rem; border-bottom: 1px solid var(--border-subtle); background-color: rgba(15, 23, 42, 0.9); } th { text-align: left; font-weight: 500; color: var(--fg-muted); } tbody tr:nth-child(even) td { background-color: rgba(15, 23, 42, 0.75); } tbody tr:last-child td { border-bottom: none; } /* Links */ a { color: var(--accent); } a:hover { text-decoration: none; } /* Code blocks */ pre, code, pre code, .highlight, .highlight pre, .highlight code { background-color: rgba(15, 23, 42, 0.96) !important; color: var(--fg); } pre { border: 1px solid var(--border-subtle); padding: 0.85rem 1rem; border-radius: 0.5rem; overflow-x: auto; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } code { padding: 0.1rem 0.25rem; border-radius: 0.25rem; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } </style>

# Where's Franklin? - CTF Challenge

**Category:** OSINT / GTA V  
**Difficulty:** Easy  
**Answer:** Mariowe Drive

## 📥 Download Challenge

**[Download Challenge Image](https://media.githubusercontent.com/media/ANormalStick/CTF-Writeups/main/ChallangesIMade/WheresFranklin/e0652c78-acce-48d5-86e2-5106bb6e6248.jpg)** - Try to solve it yourself before reading the writeup!

---

## Challenge Overview

Players are given a screenshot of **Franklin** standing next to a road in *Grand Theft Auto V*. The objective is to determine the exact in-game location where the image was taken.

The challenge can be solved in two ways:
1. Database lookup (OSINT-style approach)
2. Manual map exploration (game knowledge approach)

---

## Step 1 — Image Analysis

The provided image showed:

- Franklin (GTA V character)
- A roadside environment
- Surrounding terrain and environmental details

There was no obvious text or map marker visible, so players had to rely on environmental clues such as:

- Road type (urban vs rural)
- Road shape (curves, intersections)
- Nearby terrain (hills, buildings, coastline, etc.)
- Vegetation and lighting

---

## Method 1 — Database Lookup (Intended Easier Route)

The image was originally sourced from `gtaguessr.com` — a game similar to GeoGuessr but based entirely on GTA V locations.

### Steps

1. Recognize that the screenshot resembles a GTA Guessr-style image.
2. Search for relevant terms such as: `GTA Guessr Franklin road`
3. Browse the GTA Guessr database and locate the exact matching image.
4. Identify the in-game location from the database entry.

This reveals the location: **Mariowe Drive**

---

## Method 2 — Manual Exploration (Hard Mode)

For players who did not recognize the source, the alternative was manual exploration.

### Approach

1. Load GTA V in free roam mode.
2. Explore the map while comparing environmental details to the screenshot.
3. Pay close attention to:
   - Road curvature
   - Elevation changes
   - Surrounding landscape
   - Nearby landmarks

After careful comparison and exploration, the location can be identified as: **Mariowe Drive**

---

## Final Answer

**Mariowe Drive**

---

[← Back to Blog](../)
