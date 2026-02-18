<style> :root { color-scheme: dark; --bg: #020617; --fg: #e5e7eb; --fg-muted: #9ca3af; --accent: #38bdf8; --border-subtle: #1f2937; } /* Base page */ html, body { margin: 0; padding: 0; background: radial-gradient(circle at top left, #020617, #020617 40%, #020617 100%); color: var(--fg); font-family: system-ui, -apple-system, BlinkMacSystemFont, "SF Pro Text", "Segoe UI", sans-serif; } /* GitHub Pages wrappers */ .page-content, .wrapper, article, .post { background: transparent !important; max-width: 960px; margin: 0 auto; padding: 2.5rem 1.5rem 4rem; } /* Headings */ .post h1, .page-content h1, article h1 { font-size: 1.6rem; margin-bottom: 0.6rem; } .post h2, .page-content h2, article h2 { font-size: 1.1rem; margin-top: 1.8rem; margin-bottom: 0.6rem; color: var(--fg-muted); } /* Tables (optional, if you use them) */ table { border-collapse: collapse; width: 100%; font-size: 0.85rem; margin: 0.4rem 0 0.8rem; border-radius: 0.6rem; overflow: hidden; } th, td { padding: 0.5rem 0.75rem; border-bottom: 1px solid var(--border-subtle); background-color: rgba(15, 23, 42, 0.9); } th { text-align: left; font-weight: 500; color: var(--fg-muted); } tbody tr:nth-child(even) td { background-color: rgba(15, 23, 42, 0.75); } tbody tr:last-child td { border-bottom: none; } /* Links */ a { color: var(--accent); } a:hover { text-decoration: none; } /* Code blocks */ pre, code, pre code, .highlight, .highlight pre, .highlight code { background-color: rgba(15, 23, 42, 0.96) !important; color: var(--fg); } pre { border: 1px solid var(--border-subtle); padding: 0.85rem 1rem; border-radius: 0.5rem; overflow-x: auto; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } code { padding: 0.1rem 0.25rem; border-radius: 0.25rem; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } </style>

# Pixel Rehab - CTF Challenge

**Category:** Forensics  
**Difficulty:** Medium  
**Flag:** `0xfun{FuN_PN9_f1Le_7z}`

## 📥 Download Challenge

**[Download pixel.fun](https://media.githubusercontent.com/media/ANormalStick/CTF-Writeups/main/ChallangesIMade/PixelRehab/pixel.fun)** - Try to solve it yourself before reading the writeup!

---

## Challenge Summary

A file named `pixel.fun` is presented as a "repaired" image from a design intern. The claim is that the *important part* survived the repair, but the prompt hints the file originally lived inside a compressed archive and the "fix" may be staged.

Goal: determine what was truly archived and recover the flag.

---

## Key Observations

- `pixel.fun` looks like a PNG, but its first signature byte is wrong (`0x88` instead of `0x89`).
- After restoring that byte, the image displays a "Pixel Rehab Clinic" card with a tempting flag string and a cheeky "looks legit" message — strong indicator of misdirection.
- The PNG structure continues normally **until `IEND`**, but the file contains **extra bytes appended after the PNG ends**.
- The appended block begins with bytes resembling a PNG header fragment (`89 50 4E 47 0D 0A`). Swapping those 6 bytes to the **7z magic** (`37 7A BC AF 27 1C`) turns the tail into a valid 7z archive.

---

## What's Actually Inside

Extracting the reconstructed archive yields a file named `real_flag.png`, which is misleadingly named: it's **WEBP**.

That image contains a QR code that's another decoy, but the real flag is printed directly on the image.

---

## Practical Workflow (Manual)

1. Fix the signature byte so PNG parsers can read the file cleanly.
2. Locate the end of the PNG stream (after the `IEND` chunk).
3. Take the trailing bytes after `IEND` and replace the first 6 bytes with the 7z magic.
4. Save as `hidden.7z` and extract:
   - `7z x hidden.7z`
5. Open the extracted image and read the printed flag text.

---

## Solver Script (Chunk-Parsing Approach)

This version parses PNG chunks properly to find the true end of the PNG, then reconstructs the 7z payload from the trailer.

```python
#!/usr/bin/env python3
"""
pixel_rehab_alt.py
Recover a 7z archive appended after a PNG-like file by parsing PNG chunks to IEND,
then swapping the 6-byte header overlay into the 7z magic.
"""

import argparse
import struct
import hashlib
from pathlib import Path

PNG_SIG = b"\x89PNG\r\n\x1a\n"
SEVENZ_MAGIC6 = b"\x37\x7a\xbc\xaf\x27\x1c"  # 7z signature (6 bytes)
PNG_OVERLAY6 = b"\x89PNG\r\n"                   # first 6 bytes of PNG signature


def png_end_offset(buf: bytes) -> int:
    """Return the offset right after the IEND chunk by parsing PNG chunks."""
    if len(buf) < 8:
        raise ValueError("Too small to be PNG-like.")

    if buf[1:8] != PNG_SIG[1:8]:
        raise ValueError("Not PNG-like (signature mismatch beyond the first byte).")

    p = 8  # after signature
    while True:
        if p + 8 > len(buf):
            raise ValueError("EOF while reading chunk header.")

        length = struct.unpack(">I", buf[p:p + 4])[0]
        ctype = buf[p + 4:p + 8]
        p += 8

        if p + length + 4 > len(buf):
            raise ValueError(f"Chunk {ctype!r} overruns file (len={length}).")

        p += length + 4  # data + CRC

        if ctype == b"IEND":
            return p


def rebuild_7z(trailer: bytes) -> bytes:
    """Swap the 6-byte PNG overlay with the 7z magic and return archive bytes."""
    if len(trailer) < 6:
        raise ValueError("No trailer (or too short) to rebuild archive.")
    if trailer[:6] != PNG_OVERLAY6:
        raise ValueError(f"Unexpected trailer prefix: {trailer[:6].hex()}")
    return SEVENZ_MAGIC6 + trailer[6:]


def main():
    ap = argparse.ArgumentParser()
    ap.add_argument("input", type=Path, help="Input file (e.g., pixel.fun)")
    ap.add_argument("-o", "--out", type=Path, default=Path("hidden_payload.7z"))
    args = ap.parse_args()

    raw = args.input.read_bytes()
    end = png_end_offset(raw)
    tail = raw[end:]

    print(f"[i] PNG ends at offset: {end}")
    print(f"[i] Trailer length: {len(tail)} bytes")

    archive = rebuild_7z(tail)
    args.out.write_bytes(archive)

    print(f"[+] Wrote: {args.out} ({len(archive)} bytes)")
    print(f"[i] sha256: {hashlib.sha256(archive).hexdigest()}")
    print("[+] Extract with: 7z x hidden_payload.7z")


if __name__ == "__main__":
    main()
```

---

[← Back to Blog](../)
