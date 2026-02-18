<style> :root { color-scheme: dark; --bg: #020617; --fg: #e5e7eb; --fg-muted: #9ca3af; --accent: #38bdf8; --border-subtle: #1f2937; } /* Base page */ html, body { margin: 0; padding: 0; background: radial-gradient(circle at top left, #020617, #020617 40%, #020617 100%); color: var(--fg); font-family: system-ui, -apple-system, BlinkMacSystemFont, "SF Pro Text", "Segoe UI", sans-serif; } /* GitHub Pages wrappers */ .page-content, .wrapper, article, .post { background: transparent !important; max-width: 960px; margin: 0 auto; padding: 2.5rem 1.5rem 4rem; } /* Headings */ .post h1, .page-content h1, article h1 { font-size: 1.6rem; margin-bottom: 0.6rem; } .post h2, .page-content h2, article h2 { font-size: 1.1rem; margin-top: 1.8rem; margin-bottom: 0.6rem; color: var(--fg-muted); } /* Tables (optional, if you use them) */ table { border-collapse: collapse; width: 100%; font-size: 0.85rem; margin: 0.4rem 0 0.8rem; border-radius: 0.6rem; overflow: hidden; } th, td { padding: 0.5rem 0.75rem; border-bottom: 1px solid var(--border-subtle); background-color: rgba(15, 23, 42, 0.9); } th { text-align: left; font-weight: 500; color: var(--fg-muted); } tbody tr:nth-child(even) td { background-color: rgba(15, 23, 42, 0.75); } tbody tr:last-child td { border-bottom: none; } /* Links */ a { color: var(--accent); } a:hover { text-decoration: none; } /* Code blocks */ pre, code, pre code, .highlight, .highlight pre, .highlight code { background-color: rgba(15, 23, 42, 0.96) !important; color: var(--fg); } pre { border: 1px solid var(--border-subtle); padding: 0.85rem 1rem; border-radius: 0.5rem; overflow-x: auto; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } code { padding: 0.1rem 0.25rem; border-radius: 0.25rem; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } </style>

# Temptation. Stone. Silence. - CTF Challenge

**Category:** OSINT  
**Difficulty:** Medium  
**Flag:** `0xfun{Mālpils_Ērgļi_Lube}`

## 📥 Download Challenge

**[Download Temptation. Stone. Silence..zip](https://media.githubusercontent.com/media/ANormalStick/CTF-Writeups/main/ChallangesIMade/TemptationStoneSilence/Temptation.%20Stone.%20Silence..zip)** - Try to solve it yourself before reading the writeup!

---

## Challenge Summary

Three images, three fragments — each pointing to a Latvian place. Identify the Latvian place name for each image and submit:

`0xfun{City1_City2_City3}`

Latvian spelling **and diacritics** are required.

---

## Overview / Approach

Each image is a "fragment" that points to a location. Three separate OSINT mini-tasks:

1. **Temptation** → find a distinctive object via reverse image search + confirm exact location.
2. **Stone** → identify the carved face/person first, then locate the matching monument/grave.
3. **Silence** → use the legend clue (deal with the devil) + the "boat" shape to locate the site.

Main techniques:
- Reverse image search (Google Images / Lens)
- Visual clue extraction (subject, environment, timestamps)
- Targeted keyword searching (person name / legend name + Latvia)

---

## 1. Temptation (Golden Pig) → Mālpils

### What I Saw
A large **golden pig statue** on grass, with flowers and a small landscaped structure nearby.

### Steps
1. **Reverse image search** the pig photo.
2. Results pointed to two candidate locations: **Rīga** and **Mālpils**.
3. The reverse search alone didn't fully disambiguate, so I looked for **the exact photo** and any metadata.
4. The image had a **date stamp**, and with that I found a matching photo set in the **LETA** photo archive, where the location is explicitly listed as Mālpils.

**Reference (LETA album):**
```
https://leta.lv/photo/album/A9CD3C76-C38D-44E6-928B-0D160A6E3101
```

**City1 = Mālpils**

---

## 2. Stone (Boulder with Carved Face) → Ērgļi

### What I Saw
A large boulder in a forest with a carved **portrait medallion** (face in relief) embedded into the stone.

### Steps
1. **Reverse image search** was not immediately helpful.
2. Shifted to **identifying the face**: searching for "Latvian writer portrait relief stone" and similar terms.
3. The face matches **Rūdolfs Blaumanis** (Latvian writer / playwright).
4. Searching for **Rūdolfs Blaumanis** + **grave** / **memorial** leads to a grave site in **Ērgļi**, where photos show the same distinctive stone with the portrait medallion.

**City2 = Ērgļi**

---

## 3. Silence (Stone "Boat" in Forest) → Lube

### What I Saw
A quiet forest clearing with a low, elongated stone formation resembling a **boat**.

### Steps
1. **Reverse image search** and generic keyword attempts didn't return a clean match.
2. Used the narrative clue: "The silence of a legend" and "They say the region's elder once made a deal with the devil…"
3. Combining **Latvian devil folklore** + **boat**, searched for legends/monuments with that motif.
4. This led to a known site/monument referred to as a "devil's boat" in a forest next to **Lube**.
5. Comparing images of that site with the challenge image confirmed the match.

**City3 = Lube**

---

## Final Flag

`0xfun{Mālpils_Ērgļi_Lube}`

---

[← Back to Blog](../)
