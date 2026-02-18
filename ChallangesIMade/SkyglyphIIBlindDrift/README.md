<style> :root { color-scheme: dark; --bg: #020617; --fg: #e5e7eb; --fg-muted: #9ca3af; --accent: #38bdf8; --border-subtle: #1f2937; } /* Base page */ html, body { margin: 0; padding: 0; background: radial-gradient(circle at top left, #020617, #020617 40%, #020617 100%); color: var(--fg); font-family: system-ui, -apple-system, BlinkMacSystemFont, "SF Pro Text", "Segoe UI", sans-serif; } /* GitHub Pages wrappers */ .page-content, .wrapper, article, .post { background: transparent !important; max-width: 960px; margin: 0 auto; padding: 2.5rem 1.5rem 4rem; } /* Headings */ .post h1, .page-content h1, article h1 { font-size: 1.6rem; margin-bottom: 0.6rem; } .post h2, .page-content h2, article h2 { font-size: 1.1rem; margin-top: 1.8rem; margin-bottom: 0.6rem; color: var(--fg-muted); } /* Tables (optional, if you use them) */ table { border-collapse: collapse; width: 100%; font-size: 0.85rem; margin: 0.4rem 0 0.8rem; border-radius: 0.6rem; overflow: hidden; } th, td { padding: 0.5rem 0.75rem; border-bottom: 1px solid var(--border-subtle); background-color: rgba(15, 23, 42, 0.9); } th { text-align: left; font-weight: 500; color: var(--fg-muted); } tbody tr:nth-child(even) td { background-color: rgba(15, 23, 42, 0.75); } tbody tr:last-child td { border-bottom: none; } /* Links */ a { color: var(--accent); } a:hover { text-decoration: none; } /* Code blocks */ pre, code, pre code, .highlight, .highlight pre, .highlight code { background-color: rgba(15, 23, 42, 0.96) !important; color: var(--fg); } pre { border: 1px solid var(--border-subtle); padding: 0.85rem 1rem; border-radius: 0.5rem; overflow-x: auto; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } code { padding: 0.1rem 0.25rem; border-radius: 0.25rem; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } </style>

# Skyglyph II: Blind Drift - CTF Challenge

**Category:** Misc / Crypto / Forensics  
**Difficulty:** Very Hard  
**Flag:** `0xfun{w0w_Y0u_4R3_G0oD_4t_Th1s_ST4r_Th1N9}`

## 📥 Download Challenge

**[Download Skyglyph II Blind Drift.zip](https://media.githubusercontent.com/media/ANormalStick/CTF-Writeups/main/ChallangesIMade/SkyglyphIIBlindDrift/Skyglyph%20II%20Blind%20Drift.zip)** - Try to solve it yourself before reading the writeup!

---

## Challenge Summary

This edition is designed to reduce overwhelm by giving **progress feedback**: each frame decrypts **one part** of the flag independently.

Solve frame1 → decrypt `cipher1.bin` → you get `PART 1/4`. Repeat for frames 2–4, then concatenate the parts.

---

## What You're Given

- `frame1.csv … frame4.csv`: detections (`x_px,y_px,flux,sigma_px`)
- `catalog.csv`: stars (`star_id,ra_deg,dec_deg,mag`)
- `seed.json`: approximate pointing center for **frame1 only** (`ra0_deg, dec0_deg`)
- `cipher1.bin..cipher4.bin` + `nonce1.bin..nonce4.bin`
- `crypto.txt`: AEAD details (ChaCha20-Poly1305)

---

## Per-Frame Key Rule (The Oracle)

For one frame:

1. Plate-solve: match detections ↔ catalog stars
2. Keep matches where:
   - catalog `mag < 6.0`
   - detection `sigma_px < 1.2`
3. Sort remaining matches by detection `flux` descending
4. Take the first **64** `star_id` values

Encode the 64 IDs as little-endian u32 bytes and compute:

`key = SHA256(bytes)`

Decrypt the frame's ciphertext with:
- AEAD: ChaCha20-Poly1305
- nonce: `nonce{i}.bin`
- ciphertext: `cipher{i}.bin`
- AAD: `PlateSolve++|frame={i}`

If any ID is wrong, AEAD authentication fails (perfect correctness check).

---

## Step 1 — Project the Catalog to a 2D Tangent Plane (Gnomonic)

For frame1, use the seed center as `(ra0, dec0)`. Convert degrees → radians and project:

```
Δra = wrap_to_[-π,π](ra - ra0)

cosc = sin(dec0)*sin(dec) + cos(dec0)*cos(dec)*cos(Δra)
u    = (cos(dec) * sin(Δra)) / cosc
v    = (cos(dec0)*sin(dec) - sin(dec0)*cos(dec)*cos(Δra)) / cosc
```

## Step 2 — Bootstrap Pose (Similarity Transform)

Fit: `[x,y] ≈ s * R(θ) * [u,v] + t`

Bootstrap methods: triangle/quad hashing + RANSAC, or coarse roll/scale search + nearest-neighbor scoring (the seed reduces search space).

Filter detections first: keep `sigma_px < ~2.0–2.2`, take top 800–1500 by `flux`.

## Step 3 — Refine Radial Distortion (k1, k2)

```
r² = u² + v²
f  = 1 + k1*r² + k2*r⁴
(u_d, v_d) = (u*f, v*f)
```

Iterate match → least squares → tighten gate.

## Step 4 — Extract 64 IDs and Decrypt One Part

```python
import hashlib, struct
from cryptography.hazmat.primitives.ciphers.aead import ChaCha20Poly1305

ids64 = [...]  # 64 ints for this frame
blob = b"".join(struct.pack("<I", x) for x in ids64)
key = hashlib.sha256(blob).digest()

nonce = open("nonce1.bin","rb").read()
ct    = open("cipher1.bin","rb").read()
aad   = b"PlateSolve++|frame=1"

pt = ChaCha20Poly1305(key).decrypt(nonce, ct, aad)
print(pt.decode())
```

Repeat for frames 2–4. Concatenate the returned `PART i/4:` payloads.

## Final Flag

Concatenating all 4 decrypted parts yields:

`0xfun{w0w_Y0u_4R3_G0oD_4t_Th1s_ST4r_Th1N9}`

---

[← Back to Blog](../)
