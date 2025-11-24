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


# Sepolia Heist – Write-up

> **CTF:** MCTF 2025  
> **Challenge name:** Sepolia Heist  
> **Category:** Blockchain / Forensics  
> **Difficulty:** ~Medium  

---

## Challenge description

We’re given a text file:

- `challenge.txt`

It contains an almost complete BIP-39 seed phrase:

```text
thumb ten glide antique ready jaguar gap ______ despair trophy slide weapon
```

We’re told:

- Only one word is missing (the 8th word).  
- The victim used this wallet on **Sepolia**.  
- The flag is hidden in the **first transaction he made** on the blockchain.  
- That transaction happened on **20.11.2025**.  
- The flag format is `MCTF25{...}` and may contain non-alphanumeric characters.

Goal: recover the flag.

---

## High-level idea

1. The given phrase looks like a **12-word BIP-39 mnemonic** with one word missing.  
2. We brute-force the missing word using the official BIP-39 wordlist and keep only **checksum-valid** mnemonics.  
3. For each valid mnemonic, we derive the **Ethereum address** at `m/44'/60'/0'/0/0`.  
4. For each address, we check its transactions on **Sepolia** using the block explorer.  
5. We look for an address whose **first outgoing transaction** is on **2025-11-20**, and inspect its input data for the flag.

---

## Step 1 – Brute-forcing the missing BIP-39 word

First I created a Python virtual environment and installed the needed libraries:

```bash
python3 -m venv venv
source venv/bin/activate

pip install mnemonic eth-account requests
```

Then I wrote a small script to:

- load the official English BIP-39 wordlist via `mnemonic`,
- insert each word at position 7 (0-based index) into the phrase,
- keep only phrases that pass the BIP-39 checksum (`mnemo.check`).

```python
# candidates.py
from mnemonic import Mnemonic

mnemo = Mnemonic("english")
wordlist = mnemo.wordlist

BASE_WORDS = [
    "thumb", "ten", "glide", "antique", "ready", "jaguar",
    "gap",     # index 6
    None,     # index 7 -> missing
    "despair", "trophy", "slide", "weapon"
]

MISSING_INDEX = 7

def build_phrase(w):
    words = BASE_WORDS.copy()
    words[MISSING_INDEX] = w
    return " ".join(words)

candidates = []

for w in wordlist:
    phrase = build_phrase(w)
    if mnemo.check(phrase):
        candidates.append((w, phrase))

print(f"Checksum-valid candidates: {len(candidates)}")
for w, phrase in candidates:
    print(w, "->", phrase)
```

Running this:

```bash
python candidates.py > candidates.txt
```

Result: **134** checksum-valid candidate mnemonics (out of 2048 possible words).

---

## Step 2 – Deriving Ethereum addresses

Next, for each candidate mnemonic I derived the Ethereum address at the standard Metamask path `m/44'/60'/0'/0/0`:

```python
# derive_addresses.py
from mnemonic import Mnemonic
from eth_account import Account

Account.enable_unaudited_hdwallet_features()
mnemo = Mnemonic("english")

BASE_WORDS = [
    "thumb", "ten", "glide", "antique", "ready", "jaguar",
    "gap", None, "despair", "trophy", "slide", "weapon"
]
MISSING_INDEX = 7

def build_phrase(w):
    words = BASE_WORDS.copy()
    words[MISSING_INDEX] = w
    return " ".join(words)

candidates = []
for w in mnemo.wordlist:
    phrase = build_phrase(w)
    if mnemo.check(phrase):
        candidates.append((w, phrase))

for w, phrase in candidates:
    acct = Account.from_mnemonic(phrase, account_path="m/44'/60'/0'/0/0")
    print(w, acct.address)
```

I saved the output:

```bash
python derive_addresses.py > addresses.txt
```

Each line looks like:

```text
person 0xADEceCb8f16670F0E132e6B248cDebA4321416Cc
...
```

So for each potential missing word we now have the corresponding Ethereum address.

---

## Step 3 – Checking Sepolia for the right wallet

I initially tried to automate this using the Sepolia Etherscan API, but the free endpoint kept returning `status=0, message='NOTOK'`.  
So I switched to a **manual** (but still fast) approach using the Etherscan web UI.

From `addresses.txt` I extracted just the addresses:

```bash
awk '{print $2}' addresses.txt > just_addresses.txt
```

Then:

1. Opened `sepolia.etherscan.io` in a browser.  
2. For each address in `just_addresses.txt`, I pasted it into the search bar.
3. For most addresses the explorer showed **no transactions**.  
4. Eventually I found an address that had transactions:

   ```text
   0xADEceCb8f16670F0E132e6B248cDebA4321416Cc
   ```

   This address corresponds to the missing word:

   ```text
   person
   ```

   So the full mnemonic is:

   ```text
   thumb ten glide antique ready jaguar gap person despair trophy slide weapon
   ```

---

## Step 4 – Finding the first **outgoing** tx on 2025-11-20

On the Etherscan address page for  
`0xADEceCb8f16670F0E132e6B248cDebA4321416Cc`:

1. I opened the **Transactions** tab.
2. I identified the **first transaction where the `From` address equals this wallet** (the first tx the owner **made**, not just received).
3. That transaction had timestamp **2025-11-20**, matching the challenge metadata.
4. I opened that transaction’s detail page.

There was an **earlier incoming tx** with empty input data (`0x`), which is just a funding tx and not relevant.  
The **second transaction**, sent *from* the wallet, is the one we want.

---

## Step 5 – Extracting the flag from tx input

On the transaction details page:

1. I scrolled to the **“Input Data”** section.  
2. Switched **“View Input As” → “UTF-8”**.  
3. The decoded text contained the flag in clear:

```text
MCTF25{b1p39_jungl3_tr3@sur3_hunt}
```

Replace the placeholder with the exact flag you saw in the UTF‑8 view.

---

## Final answer

- **Recovered mnemonic (seed phrase)**  
  ```text
  thumb ten glide antique ready jaguar gap person despair trophy slide weapon
  ```

- **Wallet address (m/44'/60'/0'/0/0)**  
  ```text
  0xADEceCb8f16670F0E132e6B248cDebA4321416Cc
  ```

- **Flag**  
  ```text
  MCTF25{b1p39_jungl3_tr3@sur3_hunt}
  ```

---
