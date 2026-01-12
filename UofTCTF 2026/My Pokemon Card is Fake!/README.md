<style> :root { color-scheme: dark; --bg: #020617; --fg: #e5e7eb; --fg-muted: #9ca3af; --accent: #38bdf8; --border-subtle: #1f2937; } /* Base page */ html, body { margin: 0; padding: 0; background: radial-gradient(circle at top left, #020617, #020617 40%, #020617 100%); color: var(--fg); font-family: system-ui, -apple-system, BlinkMacSystemFont, "SF Pro Text", "Segoe UI", sans-serif; } /* GitHub Pages wrappers */ .page-content, .wrapper, article, .post { background: transparent !important; max-width: 960px; margin: 0 auto; padding: 2.5rem 1.5rem 4rem; } /* Headings */ .post h1, .page-content h1, article h1 { font-size: 1.6rem; margin-bottom: 0.6rem; } .post h2, .page-content h2, article h2 { font-size: 1.1rem; margin-top: 1.8rem; margin-bottom: 0.6rem; color: var(--fg-muted); } /* Tables (optional, if you use them) */ table { border-collapse: collapse; width: 100%; font-size: 0.85rem; margin: 0.4rem 0 0.8rem; border-radius: 0.6rem; overflow: hidden; } th, td { padding: 0.5rem 0.75rem; border-bottom: 1px solid var(--border-subtle); background-color: rgba(15, 23, 42, 0.9); } th { text-align: left; font-weight: 500; color: var(--fg-muted); } tbody tr:nth-child(even) td { background-color: rgba(15, 23, 42, 0.75); } tbody tr:last-child td { border-bottom: none; } /* Links */ a { color: var(--accent); } a:hover { text-decoration: none; } /* Code blocks */ pre, code, pre code, .highlight, .highlight pre, .highlight code { background-color: rgba(15, 23, 42, 0.96) !important; color: var(--fg); } pre { border: 1px solid var(--border-subtle); padding: 0.85rem 1rem; border-radius: 0.5rem; overflow-x: auto; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } code { padding: 0.1rem 0.25rem; border-radius: 0.25rem; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } </style>

# My Pokémon Card is Fake! (Forensics) — Writeup

## TL;DR
This challenge is solved by extracting **printer tracking dots** (a.k.a. **Machine Identification Code / MIC**, “yellow dots”) from the provided card image, identifying the pattern as the **Xerox DocuColor-style** grid, and decoding it to recover the **print timestamp** and **printer serial number**.

**Final flag:** `uoftctf{2024_08_06_21:49_704641508}`

---

## Challenge prompt recap
We’re given a “prototype” Charizard card image and asked to find:

- **Date and time** it was printed (**24-hour clock**, **relative to the printer**)
- **Printer serial number**
- Flag format: `uoftctf{YYYY_MM_DD_HH:MM_SERIALNUM}`

Example: `uoftctf{9999_09_09_23:59_676767676}`

---

## Context / Why this is a forensics problem
Many color laser printers (and some other devices) embed near-invisible **yellow tracking dots** on printed pages. These dots can act like a printer “signature” and may encode:

- printer identification (often including a **serial number**)
- **date/time** of printing (depending on the pattern/printer family)

This is commonly referred to as a **Machine Identification Code (MIC)** and is a known real-world document forensics technique.

---

## Step 1 — Use the highest quality image
Tracking dots are tiny and low-contrast. If the image has heavy compression, resizing artifacts, or a low resolution, the dots can be destroyed or become too noisy.

For this solve, we use the original high-resolution card image:

- Original: https://postimg.cc/k6DW9pBV

---

## Step 2 — Reveal the “yellow dots”
The dots are often invisible at normal zoom and color balance. The trick is to isolate and amplify the **yellow** component.

### Practical approaches (any image editor)
**Option A: Decompose into color channels**
- In GIMP: `Colors → Components → Decompose…` (try **CMYK**) and inspect the **Y** channel.

**Option B: Levels / Curves**
- Increase contrast aggressively to make faint dot patterns visible.

**Option C: Threshold**
- Convert faint yellow specks into solid points (useful before decoding).

After isolating yellow and boosting contrast, the dots become visible as repeated clusters. Here is the processed “dots emphasized” version used for decoding:

- Processed: https://postimg.cc/6TkCJzwz

---

## Step 3 — Identify the pattern family (Xerox DocuColor style)
Once visible, the dots form a repeating grid consistent with the well-known **Xerox DocuColor**-style MIC pattern:

- A repeating **15 × 8** grid pattern across the page (checkerboard-like repetition)
- Each grid block encodes the same metadata

This is important because known public decoders exist for this pattern family.

---

## Step 4 — Decode the dots
With a clean 15×8 dot block extracted/visible, use a Xerox-style MIC decoder.

A working decoder implementation:
- https://cel-hub.art/yelloow-dots-decoder.html

### Decoder workflow
1. Select a clean region with a clearly visible dot grid (avoid shadows, gradients, or textured print areas).
2. Ensure the grid is correctly oriented (rotation matters; if decoding fails, rotate 90°/180° and try again).
3. Mark dots according to the decoder interface.
4. Decode to recover:
   - print date/time (24-hour clock)
   - printer serial number

---

## Results
Decoded values:

- **Printed date:** 2024-08-06  
- **Printed time:** 21:49 (24-hour clock; “relative to the printer”)  
- **Printer serial:** 704641508

---

## Flag
The required format is:
`uoftctf{YYYY_MM_DD_HH:MM_SERIALNUM}`

So the flag is:

`uoftctf{2024_08_06_21:49_704641508}`

---

## Notes / Additional background (optional reading)
This challenge ties into broader community investigation of “prototype/playtest” Pokémon cards that began appearing in auctions in 2024 and are suspected of being modern prints. Tracking dots provide an objective forensic artifact that can indicate modern printing.

Helpful threads/guides:

- Reddit lead:  
  https://www.reddit.com/r/PokeInvesting/comments/1iaxh7z/cgc_is_gonna_be_in_some_deep_water_a_lot_of_these/

- EliteFourum discussion:  
  https://www.elitefourum.com/t/many-of-the-pokemon-playtest-cards-were-likely-printed-in-2024/52421

- DIY guide to finding dots:  
  https://www.elitefourum.com/t/how-to-find-yellow-dots-in-prototypes-diy-guide/52544

- Decoder used:  
  https://cel-hub.art/yelloow-dots-decoder.html

---

## Common pitfalls
- **JPEG compression** can destroy dots: always prefer original/high-res images.
- **Wrong orientation**: rotate the dot grid if the decode output looks nonsensical.
- **Bad sampling region**: pick a uniform background area with the clearest dot visibility.
