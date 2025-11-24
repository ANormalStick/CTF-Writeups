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


# [Cryptography 4] Radical Security Animal – Writeup

**Category:** Crypto  
**Points:** 554  
**Flag:** `MCTF25{r4d1c4l_s3cur1ty}`

---

## 1. Challenge Overview

We are given:

- A TCP service: `10.240.2.11:1337`  
- A Python file `crypto.py` that implements RSA with a small key size  
- A PDF that hints the issue is related to the RSA key length  

From `crypto.py` (simplified):

```python
from Cryptodome.Util.number import getPrime, inverse, bytes_to_long, long_to_bytes

KEY_BITS = 256
E = 65537

p = getPrime(KEY_BITS // 2)
q = getPrime(KEY_BITS // 2)

n = p * q
phi = (p - 1) * (q - 1)
d = inverse(E, phi)
```

So the server is using RSA with:

- Public key: `(n, e)`  
- Private key: `d`  
- `KEY_BITS = 256`, which means `p` and `q` are 128-bit primes and `n` is only 256 bits  

A 256-bit RSA modulus is way too small and can be factored easily. That’s the core vulnerability.

---

## 2. Talking to the Secure Crypto Oracle

We connect to the service:

```bash
nc 10.240.2.11 1337
```

The service responds with:

```text
Connection established to Secure Crypto Oracle [SCO v1.0]
Session ready.

Options:
1. Encrypt message
2. Get Secret Message
3. Get Public Key
```

We need:

- The public key `(n, e)` from option **3**  
- The encrypted secret from option **2**  

First, get the public key:

```text
Enter your choice (1-3): 3
Public Key (n, e): (74453105459613851322187047665994238471320654596065423955454287400990254712301, 65537)
```

Then, get the secret message:

```text
Enter your choice (1-3): 2
57404030761199998498847049987163372359776031729451339753283863067115090061488
```

So we have:

```text
n = 74453105459613851322187047665994238471320654596065423955454287400990254712301
e = 65537
C = 57404030761199998498847049987163372359776031729451339753283863067115090061488
```

The server is doing standard RSA encryption:

```text
C = m^e mod n
```

where `m` is the secret message (the flag) converted to an integer.

---

## 3. Spotting the Vulnerability

From `crypto.py`:

```python
KEY_BITS = 256
p = getPrime(KEY_BITS // 2)
q = getPrime(KEY_BITS // 2)
n = p * q
```

This means:

- `p` is a 128-bit prime  
- `q` is a 128-bit prime  
- `n` is a 256-bit modulus  

Real-world RSA typically uses at least 2048-bit moduli. A 256-bit modulus is trivial to factor with modern tools.

So the attack strategy is:

1. Factor `n` into its prime factors `p` and `q`  
2. Compute `phi = (p - 1) * (q - 1)`  
3. Compute the private exponent `d = e^(-1) mod phi`  
4. Decrypt the ciphertext `C` using `d` to recover `m`  
5. Convert `m` back to bytes to reveal the flag string  

---

## 4. Factoring the Modulus `n`

We paste the 77-digit number `n` into a big integer factorization tool (e.g. FactorDB / WebAssembly factorizer) and obtain:

```text
p = 238310156548665930651417811579815413711
q = 312421033739741558015235161390562658691
```

Both are ~128-bit primes, and indeed:

```text
n = p * q
```

So this is the correct factorization of the RSA modulus.

---

## 5. Reconstructing the Private Key and Decrypting

Given `p`, `q`, `n`, `e`, and the ciphertext `C`, we can perform textbook RSA decryption locally.

### Step 1: Compute φ(n)

```python
phi = (p - 1) * (q - 1)
```

### Step 2: Compute the private exponent `d`

`d` is the modular inverse of `e` modulo `phi`:

```python
d = pow(e, -1, phi)  # e^(-1) mod phi
```

### Step 3: Decrypt the ciphertext

```python
m = pow(C, d, n)  # m = C^d mod n
```

### Step 4: Convert `m` to bytes and decode

Below is the complete Python script used:

```python
# rsa_decrypt.py
from math import gcd

# Prime factors from factorization
p = 238310156548665930651417811579815413711
q = 312421033739741558015235161390562658691

n = p * q
e = 65537
C = 57404030761199998498847049987163372359776031729451339753283863067115090061488

phi = (p - 1) * (q - 1)

# Compute private exponent d
d = pow(e, -1, phi)

# Decrypt ciphertext: m = C^d mod n
m = pow(C, d, n)

# Convert integer m to bytes
def long_to_bytes(x: int) -> bytes:
    if x == 0:
        return b""
    length = (x.bit_length() + 7) // 8
    return x.to_bytes(length, "big")

pt = long_to_bytes(m)
print(pt)
print(pt.decode())
```

Run it:

```bash
python3 rsa_decrypt.py
```

Output:

```text
b'MCTF25{r4d1c4l_s3cur1ty}'
MCTF25{r4d1c4l_s3cur1ty}
```

So the decrypted plaintext string is the flag.

---

## 6. Final Flag

```text
MCTF25{r4d1c4l_s3cur1ty}
```

---

## 7. Summary

- The challenge uses RSA with an extremely small 256-bit modulus, which is insecure.  
- The oracle gives:
  - The public key `(n, e)`  
  - The encrypted secret message (ciphertext `C`)  
- By factoring `n` into `p` and `q`, we can:
  - Compute `phi = (p - 1) * (q - 1)`  
  - Compute the private exponent `d = e^(-1) mod phi`  
  - Decrypt the ciphertext with `m = C^d mod n`  
  - Convert `m` back to bytes to recover the flag  

**In one sentence:**  
Because the RSA key size was only 256 bits, we could factor `n`, reconstruct the private key, decrypt the oracle’s secret, and obtain the flag:

```text
MCTF25{r4d1c4l_s3cur1ty}
```

---
