# Skyglyph II: Blind Drift

This edition is designed to reduce overwhelm by giving **progress feedback**:
each frame decrypts **one part** of the flag independently.

Solve frame1 → decrypt `cipher1.bin` → you get `PART 1/4`.
Repeat for frames 2–4, then concatenate the parts.

---

## What you’re given

- `frame1.csv … frame4.csv`: detections (`x_px,y_px,flux,sigma_px`)
- `catalog.csv`: stars (`star_id,ra_deg,dec_deg,mag`)
- `seed.json`: approximate pointing center for **frame1 only** (`ra0_deg, dec0_deg`)
- `cipher1.bin..cipher4.bin` + `nonce1.bin..nonce4.bin`
- `crypto.txt`: AEAD details (ChaCha20-Poly1305)

---

## Per-frame key rule (the oracle)

For one frame:

1) plate-solve: match detections ↔ catalog stars  
2) keep matches where:
   - catalog `mag < 6.0`
   - detection `sigma_px < 1.2`  
3) sort remaining matches by detection `flux` descending  
4) take the first **64** `star_id` values  

Encode the 64 IDs as little-endian u32 bytes and compute:

`key = SHA256(bytes)`

Decrypt the frame’s ciphertext with:
- AEAD: ChaCha20-Poly1305
- nonce: `nonce{i}.bin`
- ciphertext: `cipher{i}.bin`
- AAD: `PlateSolve++|frame={i}`

If any ID is wrong, AEAD authentication fails (perfect correctness check).

---

## Step 1 — Project the catalog to a 2D tangent plane (gnomonic)

Pixels are 2D; RA/Dec live on a sphere. Convert catalog coordinates into a local 2D plane.

For frame1, use the seed center as `(ra0, dec0)`.

Convert degrees → radians and project:

Let `Δra = wrap_to_[-π,π](ra - ra0)`.

```
cosc = sin(dec0)*sin(dec) + cos(dec0)*cos(dec)*cos(Δra)
u    = (cos(dec) * sin(Δra)) / cosc
v    = (cos(dec0)*sin(dec) - sin(dec0)*cos(dec)*cos(Δra)) / cosc
```

---

## Step 2 — Bootstrap pose (similarity transform)

Fit:

`[x,y] ≈ s * R(θ) * [u,v] + t`

Bootstrap methods:
- triangle/quad hashing + RANSAC, or
- coarse roll/scale search + nearest-neighbor scoring (the seed reduces search space)

Filter detections first:
- keep `sigma_px < ~2.0–2.2`
- take top 800–1500 by `flux`

---

## Step 3 — Refine radial distortion (k1,k2)

This edition only needs radial distortion:

- `r² = u² + v²`
- `f = 1 + k1*r² + k2*r⁴`
- `(u_d,v_d) = (u*f, v*f)`

Iterate match → least squares → tighten gate.

---

## Step 4 — Extract 64 IDs and decrypt one part

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

---

## Final flag

Concatenating all 4 decrypted parts yields:

`0xfun{w0w_Y0u_4R3_G0oD_4t_Th1s_ST4r_Th1N9}`
