<style> :root { color-scheme: dark; --bg: #020617; --fg: #e5e7eb; --fg-muted: #9ca3af; --accent: #38bdf8; --border-subtle: #1f2937; } /* Base page */ html, body { margin: 0; padding: 0; background: radial-gradient(circle at top left, #020617, #020617 40%, #020617 100%); color: var(--fg); font-family: system-ui, -apple-system, BlinkMacSystemFont, "SF Pro Text", "Segoe UI", sans-serif; } /* GitHub Pages wrappers */ .page-content, .wrapper, article, .post { background: transparent !important; max-width: 960px; margin: 0 auto; padding: 2.5rem 1.5rem 4rem; } /* Headings */ .post h1, .page-content h1, article h1 { font-size: 1.6rem; margin-bottom: 0.6rem; } .post h2, .page-content h2, article h2 { font-size: 1.1rem; margin-top: 1.8rem; margin-bottom: 0.6rem; color: var(--fg-muted); } /* Tables (optional, if you use them) */ table { border-collapse: collapse; width: 100%; font-size: 0.85rem; margin: 0.4rem 0 0.8rem; border-radius: 0.6rem; overflow: hidden; } th, td { padding: 0.5rem 0.75rem; border-bottom: 1px solid var(--border-subtle); background-color: rgba(15, 23, 42, 0.9); } th { text-align: left; font-weight: 500; color: var(--fg-muted); } tbody tr:nth-child(even) td { background-color: rgba(15, 23, 42, 0.75); } tbody tr:last-child td { border-bottom: none; } /* Links */ a { color: var(--accent); } a:hover { text-decoration: none; } /* Code blocks */ pre, code, pre code, .highlight, .highlight pre, .highlight code { background-color: rgba(15, 23, 42, 0.96) !important; color: var(--fg); } pre { border: 1px solid var(--border-subtle); padding: 0.85rem 1rem; border-radius: 0.5rem; overflow-x: auto; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } code { padding: 0.1rem 0.25rem; border-radius: 0.25rem; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } </style>

# Bring Your Own Program

## Summary
The VM exposes a capability environment (`caps`) that restricts filesystem reads to `/data/public` via key `0x0a`. A stale inline cache in opcode `0x21` (lookup) can be abused after opcode `0x70` (freeze/reorder) to remap the cached slot for key `0x0a` to slot `0`, which holds the absolute read function (key `0x00`). Using this, the program reads `/flag.txt`.

## VM Input Format
The program input is a hex string that decodes to:
- `nr` (1 byte): number of registers
- `const_count` (1 byte)
- constants (typed)
- bytecode

String constant format:
```
0x02 [u16 length little-endian] [bytes...]
```

## Relevant Opcodes
- `0x02 (op b)`: load capability object by name
- `0x20 (op c)`: get property from env by key
- `0x21 (op d)`: lookup in env chain with inline cache
- `0x70 (op k)`: freeze env (convert to dictionary, reorder slots)
- `0x60 (op h)`: relative jump (signed 16-bit)
- `0x30 (op e)`: call function
- `0x31 (op f)`: return

## Bug: Inline Cache Staleness
`op d` caches the slot index of a key in an environment. `op k` converts a slot-based env to a dictionary and reorders keys by numeric sort, but it does **not** bump the cache version. The cached slot index is then stale and can point to a different key after reordering.

In the root env:
- key `0x0a` => relative read (`/data/public`)
- key `0x00` => absolute read (normally unreachable)

If the cache was created for key `0x0a` and then the env is reordered, the cached slot can resolve to key `0x00`.

## Exploit Plan
1. Load `caps` and fetch `caps[3]` (io env).
2. Execute `op d` on key `0x0a` once to populate the inline cache.
3. Execute `op k` on the same receiver/key to reorder slots.
4. Jump back to the cached `op d` to retrieve key `0x00` (absolute read).
5. Call it with `"/flag.txt"` and return.

## Payload
Hex input (paste into `nc`):
```
0703020400636170730209002f666c61672e7478740201003102000020010003210203010a6104090070010a01040260eeff0105013006020301053106
```

## Result
```
uoftctf{c4ch3_m3_1n11n3_h0w_80u7_d4h??}
```
