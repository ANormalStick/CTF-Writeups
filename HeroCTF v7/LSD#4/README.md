<style> :root { color-scheme: dark; --bg: #020617; --fg: #e5e7eb; --fg-muted: #9ca3af; --accent: #38bdf8; --border-subtle: #1f2937; } /* Base page */ html, body { margin: 0; padding: 0; background: radial-gradient(circle at top left, #020617, #020617 40%, #020617 100%); color: var(--fg); font-family: system-ui, -apple-system, BlinkMacSystemFont, "SF Pro Text", "Segoe UI", sans-serif; } /* GitHub Pages wrappers */ .page-content, .wrapper, article, .post { background: transparent !important; max-width: 960px; margin: 0 auto; padding: 2.5rem 1.5rem 4rem; } /* Headings */ .post h1, .page-content h1, article h1 { font-size: 1.6rem; margin-bottom: 0.6rem; } .post h2, .page-content h2, article h2 { font-size: 1.1rem; margin-top: 1.8rem; margin-bottom: 0.6rem; color: var(--fg-muted); } /* Tables (optional, if you use them) */ table { border-collapse: collapse; width: 100%; font-size: 0.85rem; margin: 0.4rem 0 0.8rem; border-radius: 0.6rem; overflow: hidden; } th, td { padding: 0.5rem 0.75rem; border-bottom: 1px solid var(--border-subtle); background-color: rgba(15, 23, 42, 0.9); } th { text-align: left; font-weight: 500; color: var(--fg-muted); } tbody tr:nth-child(even) td { background-color: rgba(15, 23, 42, 0.75); } tbody tr:last-child td { border-bottom: none; } /* Links */ a { color: var(--accent); } a:hover { text-decoration: none; } /* Code blocks */ pre, code, pre code, .highlight, .highlight pre, .highlight code { background-color: rgba(15, 23, 42, 0.96) !important; color: var(--fg); } pre { border: 1px solid var(--border-subtle); padding: 0.85rem 1rem; border-radius: 0.5rem; overflow-x: auto; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } code { padding: 0.1rem 0.25rem; border-radius: 0.25rem; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } </style>

# LSD#4 – Medium LSB Steganography

Author: Thib  
Category: Forensics / Steganography  
Flag format: `Hero{...}`

---

## Challenge Description

We’re given a trippy, colorful 1500×1500 PNG image and the following hints:

- `LSD#4`
- Difficulty: *medium*
- “medium lsb steganography”
- “The square measures **100×100** and starts at coordinates **1000:1000**”
- Format: `^Hero{\S+}$`

The task is to extract the hidden flag from the image using basic LSB steganography techniques.

---

## Intuition

The description strongly hints at:

- **LSB (Least Significant Bit) steganography**
- The hidden data being confined to a **100×100** region starting at **(x=1000, y=1000)**.
- No fancy tools needed – just basic image processing and bit manipulation.

So the plan:

1. Load the image.
2. Crop the 100×100 region at (1000, 1000).
3. Inspect the least significant bits of each color channel.
4. Reassemble bits into bytes and check for readable ASCII.
5. Extract the flag from the recovered message.

---

## Requirements

- Python 3
- Pillow (`pip install pillow`)
- NumPy (`pip install numpy`)

---

## Usage

1. Save the challenge image as `lsd4.png` in the same folder as the script.
2. Run the script:

   ```bash
   python solve_lsd4.py
   ```

3. The script prints the hidden message and the flag.

---

## Solution Script (`solve_lsd4.py`)

```python
from PIL import Image
import numpy as np

IMAGE_PATH = "lsd4.png"

def bits_to_bytes(bits):
    out = bytearray()
    # convert each group of 8 bits into a byte
    for i in range(0, len(bits) - len(bits) % 8, 8):
        b = 0
        for j in range(8):
            b = (b << 1) | bits[i + j]
        out.append(b)
    return bytes(out)

def extract_lsb_region(image_path, x, y, size=100, channel_index=0, bit_index=0):
    """
    Extracts data from a square region of an image using LSB steganography.

    - x, y: top-left coordinates of the square
    - size: width/height of the square (square is size x size)
    - channel_index: 0=R, 1=G, 2=B
    - bit_index: which bit to extract (0 = least significant)
    """
    img = Image.open(image_path).convert("RGB")
    crop = img.crop((x, y, x + size, y + size))
    arr = np.array(crop)

    # Select the color channel (R/G/B)
    ch = arr[:, :, channel_index]

    # Take the chosen bit plane
    bits = ((ch >> bit_index) & 1).flatten().tolist()

    # Turn bits into bytes
    return bits_to_bytes(bits)

if __name__ == "__main__":
    # Given by the challenge
    X, Y = 1000, 1000
    SIZE = 100

    # After testing, we know: red channel (0), bit 0 (LSB)
    data = extract_lsb_region(IMAGE_PATH, X, Y, SIZE, channel_index=0, bit_index=0)

    # Decode as ASCII; ignore any trailing garbage
    message = data.decode("ascii", errors="ignore")
    print("Recovered message:\n")
    print(message)
```

---

## Step-by-Step Explanation

### 1. Cropping the Hidden Area

The hint tells us where the hidden data lives:

- top-left corner: (1000, 1000)
- size: 100×100

So we crop:

```python
crop = img.crop((1000, 1000, 1100, 1100))
```

### 2. Inspecting Bit Planes

We suspect LSB stego, so we look at different combinations:

- Channels: R, G, B → indices 0, 1, 2
- Bits: bit 0 (LSB), bit 1, bit 2

We try each combination, group bits into bytes, and see if the result looks like ASCII text.

For the **red channel, bit 0**, we get a perfectly readable message:

> Steganography is the practice of concealing information. It invo...

All other combinations produce mostly `0xFF` / garbage.

### 3. Reassembling the Message

We flatten the 100×100 red-channel pixels, take their LSBs, and turn them into bytes:

```python
ch = arr[:, :, 0]               # red channel
bits = ((ch >> 0) & 1).flatten()
data = bits_to_bytes(bits)
message = data.decode("ascii", "ignore")
```

The recovered message is a paragraph describing steganography and ends with:

```text
Here is your flag: Hero{M4YB3_TH3_L4ST_LSB?}
```

---

## Flag

```text
Hero{M4YB3_TH3_L4ST_LSB?}
```
