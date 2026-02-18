<style> :root { color-scheme: dark; --bg: #020617; --fg: #e5e7eb; --fg-muted: #9ca3af; --accent: #38bdf8; --border-subtle: #1f2937; } /* Base page */ html, body { margin: 0; padding: 0; background: radial-gradient(circle at top left, #020617, #020617 40%, #020617 100%); color: var(--fg); font-family: system-ui, -apple-system, BlinkMacSystemFont, "SF Pro Text", "Segoe UI", sans-serif; } /* GitHub Pages wrappers */ .page-content, .wrapper, article, .post { background: transparent !important; max-width: 960px; margin: 0 auto; padding: 2.5rem 1.5rem 4rem; } /* Headings */ .post h1, .page-content h1, article h1 { font-size: 1.6rem; margin-bottom: 0.6rem; } .post h2, .page-content h2, article h2 { font-size: 1.1rem; margin-top: 1.8rem; margin-bottom: 0.6rem; color: var(--fg-muted); } /* Tables (optional, if you use them) */ table { border-collapse: collapse; width: 100%; font-size: 0.85rem; margin: 0.4rem 0 0.8rem; border-radius: 0.6rem; overflow: hidden; } th, td { padding: 0.5rem 0.75rem; border-bottom: 1px solid var(--border-subtle); background-color: rgba(15, 23, 42, 0.9); } th { text-align: left; font-weight: 500; color: var(--fg-muted); } tbody tr:nth-child(even) td { background-color: rgba(15, 23, 42, 0.75); } tbody tr:last-child td { border-bottom: none; } /* Links */ a { color: var(--accent); } a:hover { text-decoration: none; } /* Code blocks */ pre, code, pre code, .highlight, .highlight pre, .highlight code { background-color: rgba(15, 23, 42, 0.96) !important; color: var(--fg); } pre { border: 1px solid var(--border-subtle); padding: 0.85rem 1rem; border-radius: 0.5rem; overflow-x: auto; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } code { padding: 0.1rem 0.25rem; border-radius: 0.25rem; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } </style>

# ANormalJourney - CTF Challenge

**Category:** Minecraft / Forensics / OSINT  
**Difficulty:** Medium  
**Flag:** `0xfun{m3m0r135_hur7_s0mt1m3s}`

## 📥 Download Challenge

**[Download ANormalStick's Let's Play #1.rar](https://media.githubusercontent.com/media/ANormalStick/CTF-Writeups/main/ChallangesIMade/ANormalJourney/ANormalSticks%20Let%27s%20Play%20%231.rar)** - Try to solve it yourself before reading the writeup!

---

## Challenge Summary

You're given a seemingly "empty" Minecraft world: paths are faint, loot is gone, and obvious routes dead-end. The trick is to ignore where the *story begins* and instead follow where the *creator stopped*.

The solution chain:

1. Recover the creator's last logout position from NBT data.
2. Use that position to find a book containing hidden coordinates.
3. Follow a second book that points to an image.
4. Use the bedrock pattern in that image to recover the exact location of the final stash.
5. Read the chests to obtain the flag.

---

## Tools

- **NBTExplorer** (or any NBT viewer) — to inspect player `.dat` files and recover the creator's last logout coordinates.
- **Base64 decoder** — any online decoder, CyberChef, or CLI `base64`.
- **Bedrock pattern → coordinate locator** — e.g. **PatternLocatorX** (seed + bedrock pattern search)
  - Repo: https://github.com/ICshX/PatternLocatorX

---

## 1. Find the Creator's Last Logout Coordinates (NBT)

In a Minecraft world save, the last known player position is stored in the player NBT data. Open the world folder and locate the player data file:

- `world/playerdata/<uuid>.dat`

Open it in **NBTExplorer** and look for the player position list:

- `Pos: [x, y, z]`

In this challenge, the creator's last logout position is:

- **`(-948, 107, 190)`**

## 2. Travel to the Logout Position and Read "My Story"

Load the world in **Minecraft 1.20.1** and go to:

- **X = -948, Y = 107, Z = 190**

At/near these coordinates you'll find a written book named **`My Story`**.

On **page 36**, there's a suspicious string:

- `Njc2NzY3Ly02NzY3Njc=`

## 3. Decode the Base64 to Get the Next Coordinates

```bash
echo 'Njc2NzY3Ly02NzY3Njc=' | base64 -d
```

It decodes to: `676767/-676767`

Interpret this as the next **X/Z** pair:

- **X = 676767**
- **Z = -676767**

## 4. Find the Book "Life" and Extract the Image Hint

At the new location you'll find another book: **`Life`**

Reading it reveals an image link:

- https://postimg.cc/yDnYqVyW/d3b0c680

The image contains a **bedrock pattern** at the bottom of the world, plus some "marker" blocks.

## 5. Determine Facing Direction from Block Textures

Because the screenshot doesn't include an F3 overlay, you need orientation. Use the *non-bedrock blocks* in the image to infer which direction the camera/player was facing. This matters because bedrock pattern matchers typically need to know how the captured pattern is rotated.

## 6. Use a Bedrock Pattern Locator

Now that you have:

- the **world seed** (from `level.dat` or in-game `/seed`)
- the **bedrock pattern** from the screenshot
- the **facing direction / orientation**

Feed it into **PatternLocatorX**, and it returns the matching coordinates.

## 7. Go to the Final Coordinates and Collect the Flag

Travel to:

- **X = 6964, Y = -57, Z = -6971**

There you'll find chests arranged to spell out the flag:

- `0xfun{m3m0r135_hur7_s0mt1m3s}`

---

[← Back to Blog](../)
