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

# RustRoll CTF Challenge Writeup

**Challenge:** RustRoll  
**Category:** Crypto/Blockchain  

## Challenge Description

> A Rust-based rollup node is running as a public TCP service.
> To save block space, we optimized the address format. We have a Vault account with a huge balance. Can you drain the magic amount from the Vault?

## Initial Reconnaissance

Connecting to the service and sending `HELP`:

```
Commands:
HELP              - show this help
INFO              - show public parameters
LIST              - show known accounts (debug)
TX <hex>          - submit transaction

Transaction layout (hex, 120 bytes):
from_addr: u32 LE
to_addr:   u32 LE
amount:    u64 LE
nonce:     u64 LE
pubkey:    [32 bytes] (Ed25519)
signature: [64 bytes] (Ed25519)
```

The `INFO` command reveals:
```
vault_addr: 622488
magic_amount: 13371337
address_bits: 20
```

And `LIST` shows existing accounts:
```
addr=622488 balance=999919771978 nonce=6
addr=240940 balance=53485348 nonce=0
...
```

## Vulnerability Analysis

The key hint is in the challenge description: **"To save block space, we optimized the address format"** combined with **`address_bits: 20`**.

This tells us:
- Addresses are only 20 bits (values 0 to 1,048,575)
- But the TX format uses `u32` (32 bits) for addresses
- There must be truncation happening somewhere

The vulnerability is a **hash collision attack** due to address truncation:

1. Addresses are derived from public keys using a hash function
2. The hash output is truncated to 20 bits
3. Since Ed25519 public keys are 32 bytes (256 bits), but addresses are only 20 bits, there are approximately 2^236 keys that map to each address
4. We can generate keypairs until we find one whose derived address collides with the vault's address

## Finding the Hash Function

After extensive testing of various hash functions (SHA256, Blake2s, Blake2b, SHA3, etc.), I discovered the system uses **Blake3**:

```python
import blake3

h = blake3.blake3(pubkey).digest()
addr = struct.unpack('<I', h[:4])[0] & 0xFFFFF  # 20-bit mask
```

## Exploit

The attack is straightforward once the hash function is known:

1. Generate random Ed25519 keypairs
2. For each keypair, compute `blake3(pubkey)[:4] & 0xFFFFF`
3. Find a keypair where this equals the vault address (622488)
4. Sign a transaction transferring the magic amount from the vault
5. Since our pubkey hashes to the same truncated address as the vault's, the system accepts our signature!

```python
import blake3
from nacl.signing import SigningKey
import struct

target = 622488  # vault address
mask = 0xFFFFF   # 20-bit mask

# Search for collision
for i in range(5000000):
    sk = SigningKey.generate()
    pk = sk.verify_key.encode()
    
    h = blake3.blake3(pk).digest()
    addr = struct.unpack('<I', h[:4])[0] & mask
    
    if addr == target:
        print(f"Found collision! PK={pk.hex()}")
        
        # Craft exploit transaction
        from_addr = target
        to_addr = 999
        amount = 13371337
        nonce = 9  # current vault nonce
        
        msg = struct.pack('<IIqq', from_addr, to_addr, amount, nonce)
        signed = sk.sign(msg)
        tx = msg + pk + signed.signature
        
        # Submit TX
        # ... send to server ...
        break
```

The collision is found after approximately 500,000-1,000,000 iterations (expected ~2^19 = 524,288 on average for a 20-bit collision).

## Flag

```
nexus{Th1s-Is_A-Hard#Fl4g!}
```

