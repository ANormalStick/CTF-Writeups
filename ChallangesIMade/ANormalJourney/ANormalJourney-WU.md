# ANormalJourney (medium) — ANormalStick

**Category:** Minecraft / Forensics / OSINT-ish  
**Version:** Java 1.20.1 world  
**Flag:** `0xfun{m3m0r135_hur7_s0mt1m3s}`

## Overview

You’re given a seemingly “empty” Minecraft world: paths are faint, loot is gone, and obvious routes dead-end. The trick is to ignore where the *story begins* and instead follow where the *creator stopped*.

The core idea is:

1. Recover the creator’s last logout position from NBT.
2. Use that position to find a book that contains hidden coordinates.
3. Follow a second book that points to an image.
4. Use the bedrock pattern in that image to recover the exact location of the final stash.
5. Read the chests to obtain the flag.

---

## Tools

- **NBTExplorer** (or any NBT viewer) — to inspect player `.dat` files and recover the creator’s last logout coordinates.
- **Base64 decoder** — any online decoder, CyberChef, or CLI `base64`.
- **Bedrock pattern → coordinate locator** — e.g. **PatternLocatorX** (seed + bedrock pattern search)  
  - Repo: https://github.com/ICshX/PatternLocatorX  
  - (It also provides a limited web app in the README.)

---

## Walkthrough

### 1) Find the creator’s last logout coordinates (NBT)

In a Minecraft world save, the last known player position is stored in the player NBT data. Open the world folder and locate the player data file:

- `world/playerdata/<uuid>.dat`

Open it in **NBTExplorer** and look for the player position list:

- `Pos: [x, y, z]`

In this challenge, the creator’s last logout position is:

- **`(-948, 107, 190)`**

### 2) Travel to the logout position and read “My Story”

Load the world in **Minecraft 1.20.1** and go to:

- **X = -948, Y = 107, Z = 190**

At/near these coordinates you’ll find a written book named:

- **`My Story`**

On **page 36**, there’s a suspicious string:

- `Njc2NzY3Ly02NzY3Njc=`

### 3) Decode the Base64 to get the next coordinates

Decode the string from Base64. Example on Linux/macOS:

```bash
echo 'Njc2NzY3Ly02NzY3Njc=' | base64 -d
```

It decodes to:

- `676767/-676767`

Interpret this as the next **X/Z** pair:

- **X = 676767**
- **Z = -676767**

(You can travel there however you like: Nether travel, teleport if allowed, or creative flight in a local solve.)

### 4) Find the book “Life” and extract the image hint

At the new location you’ll find another book:

- **`Life`**

Reading it reveals an image link:

- https://postimg.cc/yDnYqVyW/d3b0c680

The image contains a **bedrock pattern** at the bottom of the world, plus some “marker” blocks (redstone/cobble/etc.).

### 5) Determine facing direction from block textures

Because the screenshot doesn’t include an F3 overlay, you need orientation.

Use the *non-bedrock blocks* in the image (e.g., redstone, cobble, etc.) to infer which direction the camera/player was facing (North/East/South/West). This matters because bedrock pattern matchers typically need to know how the captured pattern is rotated.

### 6) Use a bedrock pattern locator to recover the exact coordinates

Now that you have:

- the **world seed** (from `level.dat` or in-game `/seed`)
- the **bedrock pattern** from the screenshot
- the **facing direction / orientation**

…you can use a bedrock pattern coordinate finder tool to search the seed for a matching bedrock formation.

One working option:

- **PatternLocatorX** — https://github.com/ICshX/PatternLocatorX

Feed it the bedrock pattern (and the correct rotation), and it returns the matching coordinates.


### 7) Go to the final coordinates and collect the flag

Travel to:

- **X = 6964, Y = -57, Z = -6971**

There you’ll find chests arranged to spell out the flag:

- `0xfun{m3m0r135_hur7_s0mt1m3s}`

---
