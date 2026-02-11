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

/* Tables */
table {
  border-collapse: collapse;
  width: 100%;
  font-size: 0.85rem;
  margin: 0.4rem 0 0.8rem;
  border-radius: 0.6rem;
  overflow: hidden;
}

th, td {
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
pre, code, pre code, .highlight, .highlight pre, .highlight code {
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

# 12 - Writeup

## Challenge
Binary-only CTF challenge. The file is an ELF64 PIE named `12`.

## Summary
The binary validates a UDP response and then unpacks and runs a staged RWX blob. The 56-byte UDP response must have a fixed 8-byte header and a 48-byte payload. The payload is used as a per-stage key; each stage embeds a hash check that can be inverted to recover the next 32-bit word of the key. After 12 iterations, the correct 48-byte payload drops the flag.

Flag:
`0xL4ugh{eNOUgh_OBFuscaT!On_For_tOD@y_I7S_Time_for_tH3_FLag!}`

## Step 1: Identify the expected UDP response
The program sends a 4-byte request to `127.0.0.1:1337` and expects a response. Logging the `recvfrom` buffer and tracing the validation function shows:

- First 4 bytes must equal `8ae1aff5`.
- Bytes 4-5 (network order) must equal total length.
- Bytes 6-7 (network order) must equal `0x000c` (12).
- Constraint: `len - 8 == val6 * 4`.

That forces:
- `len = 56 (0x0038)`,
- `val6 = 12 (0x000c)`,
- payload length = 48 bytes.

Response format (hex):
`8ae1aff50038000c` + 48-byte payload.

## Step 2: Observe staged blob behavior
After validation, the program:
1. Copies 48 bytes into a struct.
2. `mmap`'s RWX memory.
3. Copies a 600+ byte blob into the RWX page and executes it.

Dumping the exec map shows a self-decrypting stub. The stub has two XOR/ADD loops and then a "hash check" function that returns `1` only if a 32-bit input matches a hardcoded target.

## Step 3: Extract the per-stage hash check
When the payload is all zeros, the decrypted blob contains a function that:
- Reads the first 4 bytes of its argument as a 32-bit value.
- Applies a fixed sequence of XOR, add, rotate, and multiply operations.
- Compares the result to a 64-bit immediate.

Because the operations are all invertible modulo 2^64, you can invert the function to recover the required 32-bit input for that stage. That recovered 32-bit word is the next 4 bytes of the UDP payload.

## Step 4: Automate all 12 rounds
Each new payload word changes the decrypted blob and therefore the next hardcoded target. Repeat the process 12 times to recover the full 48-byte payload.

I automated this in `E:\ctf4\derive_key.py` and executed the blob locally using `E:\ctf4\run_exec_blob`:

Key (48-byte payload) hex:
`bb1f051b94fbb62e38aee6ebda3721e2c0e44323fa05bd58a771e7a6fcd1374ae2107871b8d688e8be021fe5b2cbc004`

## Step 5: Send the final response
Final response (hex):
`8ae1aff50038000c` + key hex above.

Running the binary with this response prints the flag.

## Notes / Tooling
- A preload library was used to bypass anti-debug/VM checks and to log `dlsym`, `recvfrom`, `memcmp`, `memcpy`, and RWX `mmap` usage.
- The staged blob was dumped before/after execution to study the decryptor.
- The final key derivation is deterministic and repeatable from the binary alone.

