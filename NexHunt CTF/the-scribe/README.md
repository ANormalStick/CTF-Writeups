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

# the-scribe

> “He wrote everything. You just forgot he was there.”

This challenge is a **keystroke log** disguised as a plain hex dump. The goal is to reconstruct what was typed and extract the flag.

---

## What the file is

`dump.hex` contains **PC keyboard scancodes (Set 1)**:

- **Key press** codes are `< 0x80`
- **Key release** codes are usually `press + 0x80`
  - Example: `0x1E` (press `a`) and `0x9E` (release `a`)

You’ll also see modifier scancodes such as Shift:
- Left Shift press/release: `0x2A` / `0xAA`
- Right Shift press/release: `0x36` / `0xB6`
- Caps Lock: `0x3A` (toggle on press)

So the “scribe” wrote everything — we just need to decode it.

---

## Solution approach

1. Read all hex bytes in order.
2. Track modifier state:
   - Maintain `shift = True/False`
   - Maintain `caps = True/False` (toggled by Caps Lock press)
3. Ignore most **release** events (`>= 0x80`), *except* Shift releases which matter for state.
4. Translate **press** scancodes using a Set 1 lookup table:
   - `unshift` map for normal typing
   - `shifted` map for Shift held (symbols / uppercase)
5. Handle **Backspace** (`0x0E`) by removing the last output character.
6. After reconstructing the typed text, search for `nexus{...}` and pick the real flag (there are decoys).

---

## Reference decoder (Python)

Save as `solve.py` next to `dump.hex` and run with Python 3.

```python
import re
from pathlib import Path

# Scancode Set 1 lookup tables (common keys)
unshift = {
    0x02:'1',0x03:'2',0x04:'3',0x05:'4',0x06:'5',0x07:'6',0x08:'7',0x09:'8',0x0A:'9',0x0B:'0',
    0x0C:'-',0x0D:'=',
    0x0F:'\t',
    0x10:'q',0x11:'w',0x12:'e',0x13:'r',0x14:'t',0x15:'y',0x16:'u',0x17:'i',0x18:'o',0x19:'p',
    0x1A:'[',0x1B:']',0x1C:'\n',
    0x1E:'a',0x1F:'s',0x20:'d',0x21:'f',0x22:'g',0x23:'h',0x24:'j',0x25:'k',0x26:'l',
    0x27:';',0x28:"'",0x29:'`',0x2B:'\\',
    0x2C:'z',0x2D:'x',0x2E:'c',0x2F:'v',0x30:'b',0x31:'n',0x32:'m',
    0x33:',',0x34:'.',0x35:'/',0x39:' '
}

shifted = {
    0x02:'!',0x03:'@',0x04:'#',0x05:'$',0x06:'%',0x07:'^',0x08:'&',0x09:'*',0x0A:'(',0x0B:')',
    0x0C:'_',0x0D:'+',
    0x0F:'\t',
    0x10:'Q',0x11:'W',0x12:'E',0x13:'R',0x14:'T',0x15:'Y',0x16:'U',0x17:'I',0x18:'O',0x19:'P',
    0x1A:'{',0x1B:'}',0x1C:'\n',
    0x1E:'A',0x1F:'S',0x20:'D',0x21:'F',0x22:'G',0x23:'H',0x24:'J',0x25:'K',0x26:'L',
    0x27:':',0x28:'"',0x29:'~',0x2B:'|',
    0x2C:'Z',0x2D:'X',0x2E:'C',0x2F:'V',0x30:'B',0x31:'N',0x32:'M',
    0x33:'<',0x34:'>',0x35:'?',0x39:' '
}

def decode(scancodes):
    out = []
    shift = False
    caps = False

    for b in scancodes:
        # Shift press/release
        if b in (0x2A, 0x36):
            shift = True
            continue
        if b in (0xAA, 0xB6):
            shift = False
            continue

        # Caps Lock toggles on press
        if b == 0x3A:
            caps = not caps
            continue

        # Ignore releases
        if b >= 0x80:
            continue

        # Backspace
        if b == 0x0E:
            if out:
                out.pop()
            continue

        # Choose table by shift state
        table = shifted if shift else unshift
        ch = table.get(b, '')

        # Caps Lock affects letters (caps XOR shift behavior)
        if ch.isalpha():
            ch = ch.upper() if (caps ^ shift) else ch.lower()

        out.append(ch)

    return ''.join(out)

raw = Path("dump.hex").read_text().split()
sc = [int(x, 16) for x in raw]

text = decode(sc)
print(text)

flags = re.findall(r"nexus\{[^}\n\r]+\}", text)
print("\nFound flags:", flags)
```

---

## Extracted flags (decoys included)

Running the solver on the provided dump yields three `nexus{...}` strings:

- `nexus{n0t_th3_f14g_k33p_l00k1ng_babe}` *(decoy)*
- `nexus{f4k3_fl4g_f0r_th3_l0lz_gg_wp}` *(decoy)*
- **Real flag:** `nexus{1_c4ptur3d_k3y5_wh1l3_w4tch1ng_m3551}`

---

## Flag

`nexus{1_c4ptur3d_k3y5_wh1l3_w4tch1ng_m3551}`
