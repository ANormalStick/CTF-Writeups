<style> :root { color-scheme: dark; --bg: #020617; --fg: #e5e7eb; --fg-muted: #9ca3af; --accent: #38bdf8; --border-subtle: #1f2937; } /* Base page */ html, body { margin: 0; padding: 0; background: radial-gradient(circle at top left, #020617, #020617 40%, #020617 100%); color: var(--fg); font-family: system-ui, -apple-system, BlinkMacSystemFont, "SF Pro Text", "Segoe UI", sans-serif; } /* GitHub Pages wrappers */ .page-content, .wrapper, article, .post { background: transparent !important; max-width: 960px; margin: 0 auto; padding: 2.5rem 1.5rem 4rem; } /* Headings */ .post h1, .page-content h1, article h1 { font-size: 1.6rem; margin-bottom: 0.6rem; } .post h2, .page-content h2, article h2 { font-size: 1.1rem; margin-top: 1.8rem; margin-bottom: 0.6rem; color: var(--fg-muted); } /* Tables (optional, if you use them) */ table { border-collapse: collapse; width: 100%; font-size: 0.85rem; margin: 0.4rem 0 0.8rem; border-radius: 0.6rem; overflow: hidden; } th, td { padding: 0.5rem 0.75rem; border-bottom: 1px solid var(--border-subtle); background-color: rgba(15, 23, 42, 0.9); } th { text-align: left; font-weight: 500; color: var(--fg-muted); } tbody tr:nth-child(even) td { background-color: rgba(15, 23, 42, 0.75); } tbody tr:last-child td { border-bottom: none; } /* Links */ a { color: var(--accent); } a:hover { text-decoration: none; } /* Code blocks */ pre, code, pre code, .highlight, .highlight pre, .highlight code { background-color: rgba(15, 23, 42, 0.96) !important; color: var(--fg); } pre { border: 1px solid var(--border-subtle); padding: 0.85rem 1rem; border-radius: 0.5rem; overflow-x: auto; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } code { padding: 0.1rem 0.25rem; border-radius: 0.25rem; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } </style>

# Music Box v2 - CTF Challenge

**Category:** Misc / Steganography  
**Difficulty:** Medium-Hard  
**Flag:** `KIB{aes_k3y_sl1ngs_christm4s_tun3}`  

## 📥 Download Challenge

**[Download Music Box v2.7z](./Music%20Box%20v2.7z)** - Try to solve it yourself before reading the writeup!

---

## Challenge Summary

You’re given a 7z archive:

```bash
Music Box v2.7z
```

- Two encrypted presents, one real and one fake.
- Music and Morse code: “listen closely”.
- “Where the ear fails, the eye reveals” → spectrograms / visual analysis.
- “Salted, padded, and blocky” → classical block cipher with salt (AES/CBC).
- Elves were messy with the wrapping → data is hidden in containers (archive, media files).

The solution chain:

1. Find a hidden hex blob inside the 7z and decode it.
2. Use that decoded message to identify which song to inspect.
3. Open that song in a spectrogram and read Morse code to get the passphrase.
4. Fix XOR-corrupted image files to get valid WebP images.
5. Carve attached AES ciphertext from those WebP images.
6. Decrypt both presents with AES-128-CBC + PBKDF2-MD5 using the passphrase.
7. Identify which decrypted present contains the real flag.

---

## 1. Inspecting the 7z Archive

Instead of just extracting, we inspect the archive content as raw bytes:

```bash
strings "Music Box v2.7z" | grep -E "[0-9A-Fa-f]{32,}"
```

Among the output, we find a long hex string, e.g.:

```text
XOR1_HEX:3027242730272c21271d363023212978620430232c2962...
```

We copy only the hex part (no `XOR1_HEX:` label).  
This looks like an obfuscated message: long, clean hex, not random noise.

---

## 2. Decoding the XOR’d Hex Hint

We suspect this hex is XOR-encoded with a single-byte key.  
Using Python:

```bash
python3 - << 'EOF'
import binascii

hex_str = "3027242730272c21271d363023212978620430232c2962112b2c23363023626f62112b2e272c36620c2b252a366c352334"

data = binascii.unhexlify(hex_str)

key = 0x42  # XOR key used by the elves
decoded = bytes(b ^ key for b in data)

print("As text:", decoded.decode())
EOF
```

Output (example):

```text
As text: reference_track: <name_of_song>.wav
```

This tells us exactly which track to analyze for Morse in the next step.

---

## 3. Spectrogram & Morse – Getting the Passphrase

Extract the archive and list the contents:

```bash
7z x "Music Box v2.7z" -oMusicBox_v2
cd MusicBox_v2
ls
```

Among the audio files, we locate the `reference_track` mentioned above.

We open it in **Audacity** (or any audio editor), switch the track view to **Spectrogram**, and zoom in.  
We see a clear dot–dash pattern: **Morse code** rendered as tones in the spectrogram.

Decoding the Morse (visually or using a decoder) gives us a passphrase:

```text
holly!jolly!xmas
```

This will be used later as the AES passphrase.

---

## 4. Fixing the Corrupted Images (Global XOR)

The archive also contains two image files representing the presents, for example:

```text
elf.png
santa.png
```

Initially, they appear broken. When we inspect them:

```bash
file elf.png santa.png
```

After reversing the XOR (see below), they turn out to actually be **WebP (RIFF) images** with `.png` extensions.

The challenge used a **global XOR** with key `0x42` over the entire image files.  
We undo this:

```bash
python3 - << 'EOF'
key = 0x67
names = ["elf.png", "santa.png"]

for name in names:
    with open(name, "rb") as f:
        data = f.read()
    fixed = bytes(b ^ key for b in data)
    out = name.replace(".png", "_fixed.png")
    with open(out, "wb") as f_out:
        f_out.write(fixed)
    print(f"{name} -> {out}")
EOF
```

Now we check:

```bash
file elf_fixed.png santa_fixed.png
```

Example result:

```text
elf_fixed.png: RIFF (little-endian) data, Web/P image, ...
santa_fixed.png: RIFF (little-endian) data, Web/P image, ...
```

Opening them shows valid images (Santa vs Grinch).

---

## 5. Carving Attached Presents from WebP (RIFF) Images

The presents are not separate files; they are **appended to the image files**.  
Because they’re raw AES ciphertext with no magic header, `binwalk` shows nothing useful.

WebP files are wrapped in a **RIFF** container:

- Bytes `0–3`: `"RIFF"`
- Bytes `4–7`: a 32-bit little-endian size value = `total_file_size - 8`
- Bytes `8–11`: `"WEBP"`
- Then WebP chunks follow.

Therefore, the **expected** length of the WebP file is:

```text
expected_len = size_val + 8
```

Anything beyond that is extra data → our attached present.

We parse and extract that tail with Python:

```bash
python3 - << 'EOF'
import struct

for name in ["elf_fixed.png", "santa_fixed.png"]:
    with open(name, "rb") as f:
        data = f.read()

    if data[:4] != b"RIFF":
        print(f"{name}: not RIFF, magic =", data[:4])
        continue

    size_val = struct.unpack("<I", data[4:8])[0]
    expected_len = size_val + 8
    actual_len = len(data)
    extra_len = actual_len - expected_len

    print(f"{name}: actual={actual_len}, expected={expected_len}, extra={extra_len}")

    if extra_len > 0:
        extra = data[expected_len:]
        out = name.replace(".png", "_present.gift")
        with open(out, "wb") as f_out:
            f_out.write(extra)
        print(f"  -> extracted attached data to {out}")
    else:
        print("  -> no attached data found")
EOF
```

We end up with:

- `elf_fixed_present.gift`
- `santa_fixed_present.gift`

These are small, high-entropy binary blobs - our encrypted presents.

---

## 6. Crypto: AES-128-CBC + PBKDF2-MD5

From the challenge design:

- Cipher: **AES-128-CBC**
- Key derivation: **PBKDF2** using **MD5**
- Supported by a salt stored with the ciphertext
- File layout for each `.gift`:

  ```text
  salt (16 bytes) | iv (16 bytes) | ciphertext (...)
  ```

- Passphrase: `holly!jolly!xmas` (from the Morse code)
- Iterations: `100000` (example value; must match the encryption script)

We decrypt both `*_present.gift` files and see which one contains the real flag.

Using a virtual environment and `pycryptodome`:

```bash
python3 -m venv ctfenv
source ctfenv/bin/activate
pip install pycryptodome
```

Then:

```bash
nano decrypt_presents.py
```

```python
from Crypto.Cipher import AES
from Crypto.Protocol.KDF import PBKDF2
from Crypto.Hash import MD5

ITERATIONS = 100000   # Must match the encrypt-side setting
KEY_LEN = 16          # AES-128


def pkcs7_unpad(data):
    pad_len = data[-1]
    if pad_len < 1 or pad_len > 16:
        raise ValueError("Invalid padding")
    if data[-pad_len:] != bytes([pad_len]) * pad_len:
        raise ValueError("Invalid padding pattern")
    return data[:-pad_len]


def decrypt_file(fname, password):
    with open(fname, "rb") as f:
        blob = f.read()

    if len(blob) < 32:
        raise ValueError(f"{fname}: blob too small for salt+iv")

    salt = blob[0:16]
    iv = blob[16:32]
    ciphertext = blob[32:]

    # PBKDF2-HMAC-MD5 → 16-byte key for AES-128
    key = PBKDF2(password.encode(), salt,
                 dkLen=KEY_LEN,
                 count=ITERATIONS,
                 hmac_hash_module=MD5)

    cipher = AES.new(key, AES.MODE_CBC, iv)
    plaintext = cipher.decrypt(ciphertext)
    plaintext = pkcs7_unpad(plaintext)
    return plaintext


if __name__ == "__main__":
    password = "holly!jolly!xmas"

    for name in ["elf_fixed_present.gift", "santa_fixed_present.gift"]:
        print(f"=== {name} ===")
        try:
            pt = decrypt_file(name, password)
            try:
                print(pt.decode())
            except UnicodeDecodeError:
                print(repr(pt))
        except Exception as e:
            print(f"Error decrypting {name}: {e}")
        print()
```

Run it:

```bash
python decrypt_presents.py
```

One present is a decoy (Grinch), the other contains the real flag.

---

## 7. Flag

The correct decrypted present yields the final flag:

```text
KIB{aes_k3y_sl1ngs_christm4s_tun3}
```

---

## Takeaways

This challenge is a nice multi-stage chain involving:

- Archive forensics (`strings` on a 7z container).
- Simple obfuscation (hex + XOR).
- Audio forensics (spectrogram-based Morse).
- Image/container forensics (XOR-corrupted WebP, RIFF parsing).
- Symmetric crypto (AES-128-CBC, PBKDF2-MD5, salt+IV layout, PKCS#7 padding).
- Red herrings (two presents; only one true flag).
