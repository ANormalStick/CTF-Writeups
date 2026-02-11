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

# House Of Pain - Writeup

## Summary
The binary provides a menu that lets you enter a message. The small-message path contains a classic stack overflow in `small_message(int)` because it reads up to 0x100 bytes into a 0x18-byte stack buffer. A throw/catch path in `big_message(int)` returns cleanly to the menu and avoids stack-canary checks, enabling a controlled stack pivot and a final return-address overwrite to jump to `win()`.

## Files
- `chall` (ELF64, no PIE, Full RELRO, Canary, NX)
- `libc.so.6`, `ld-linux-x86-64.so.2`

## Key Findings
1. **Leak primitive** (small path)
   - `small_message(int)` does `read(0, buf, 0x100)` and then `puts(buf)`.
   - If we choose size 24, the buffer has no null terminator inside the first 0x18 bytes, so `puts` leaks adjacent stack data.
   - The 6 leaked bytes immediately after our 24 `A`'s are the saved `rbp` of `enter_message`.

2. **Bypass canary / control flow**
   - Overflowing `small_message` would normally trip the stack canary on return.
   - There is a C++ exception path from `big_message` and the top-level `try/catch` in `main` prints "Error occurred try again!" and then loops the menu. This path returns to the loop without rechecking the corrupted canary state, letting us continue with a modified stack frame.

3. **Stack pivot to overwrite scanf return address**
   - `main` uses `scanf("%d", &choice)` each loop. The return address for `scanf` is stored at `[rbp - 0x1c]` in the `main` frame.
   - If we pivot `rbp` to point into the `small_message` buffer, then `scanf` will write into our buffer, letting us overwrite its own return address.
   - From offline analysis, `main.rbp = enter_message.rbp + 0x24`.

## Exploit Strategy
1. **Leak `enter_message` saved rbp**
   - Choose `Enter message`, size `24`.
   - Send 24 `A`'s; read output until the menu prints again.
   - Parse the 6 leaked bytes after the `A`s to get `enter_message.rbp`.

2. **Overflow `small_message` to pivot `main.rbp`**
   - Choose `Enter message`, size `1` (still uses `small_message`).
   - Send a crafted 0x60-byte overflow that:
     - Restores the `small_message` saved `rbp` and saved `rip` (avoid crashing).
     - Overwrites `enter_message` saved `rbp` with `enter_message.rbp + 0x24`.
     - Restores `enter_message` saved `rip` so execution returns to the loop.

3. **Overwrite `scanf` return address with `win()`**
   - After the exception message, the menu appears again.
   - Enter a number equal to `win()` address (`0x401773`).
   - Because `main.rbp` is pivoted, `scanf` writes to `[rbp - 0x1c]` in our buffer, replacing its own return address.
   - Control returns to `win()`, which prints the flag from `flag.txt`.

## Offsets Used
- `small_message` buffer start: `rbp - 0x30`
- `small_message` saved `rbp`: `+0x30`
- `small_message` saved `rip`: `+0x38`
- `enter_message` saved `rbp`: `+0x50`
- `enter_message` saved `rip`: `+0x58`
- `main.rbp = enter_message.rbp + 0x24`

## Exploit Script
See `solve.py` for a full working exploit that:
- Leaks the stack frame pointer.
- Pivots `rbp` via the exception path.
- Overwrites the `scanf` return address with `win()`.

## Flag
`0xL4ugh{ch0p_ch0p_fr33_th3_p41nful_sp1r1t_ch0p_11a6464501114e55}`
