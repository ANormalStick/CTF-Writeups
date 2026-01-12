<style> :root { color-scheme: dark; --bg: #020617; --fg: #e5e7eb; --fg-muted: #9ca3af; --accent: #38bdf8; --border-subtle: #1f2937; } /* Base page */ html, body { margin: 0; padding: 0; background: radial-gradient(circle at top left, #020617, #020617 40%, #020617 100%); color: var(--fg); font-family: system-ui, -apple-system, BlinkMacSystemFont, "SF Pro Text", "Segoe UI", sans-serif; } /* GitHub Pages wrappers */ .page-content, .wrapper, article, .post { background: transparent !important; max-width: 960px; margin: 0 auto; padding: 2.5rem 1.5rem 4rem; } /* Headings */ .post h1, .page-content h1, article h1 { font-size: 1.6rem; margin-bottom: 0.6rem; } .post h2, .page-content h2, article h2 { font-size: 1.1rem; margin-top: 1.8rem; margin-bottom: 0.6rem; color: var(--fg-muted); } /* Tables (optional, if you use them) */ table { border-collapse: collapse; width: 100%; font-size: 0.85rem; margin: 0.4rem 0 0.8rem; border-radius: 0.6rem; overflow: hidden; } th, td { padding: 0.5rem 0.75rem; border-bottom: 1px solid var(--border-subtle); background-color: rgba(15, 23, 42, 0.9); } th { text-align: left; font-weight: 500; color: var(--fg-muted); } tbody tr:nth-child(even) td { background-color: rgba(15, 23, 42, 0.75); } tbody tr:last-child td { border-bottom: none; } /* Links */ a { color: var(--accent); } a:hover { text-decoration: none; } /* Code blocks */ pre, code, pre code, .highlight, .highlight pre, .highlight code { background-color: rgba(15, 23, 42, 0.96) !important; color: var(--fg); } pre { border: 1px solid var(--border-subtle); padding: 0.85rem 1rem; border-radius: 0.5rem; overflow-x: auto; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } code { padding: 0.1rem 0.25rem; border-radius: 0.25rem; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } </style>

# Coloring Fraud

## Summary
The interactive proof is a standard 3-coloring commit-and-open protocol, but the
commitment hash is weak for short messages. For messages of length <= 48 bytes,
`permute_fast` is used, which is linear and ignores part of the state. This makes
the hash an affine linear map from message bits to 256 output bits, so collisions
are easy to find. With a collision whose first byte differs, one commitment can
open to two different colors, letting us answer any edge query.

## Protocol recap
- Commit: send 4 hashes, one per vertex (hash of `color||nonce` bytes).
- Challenge: server picks a directed edge `(u, v)` and asks to open both vertices.
- Verify: hashes match commitments and colors differ.
- Repeat 128 times.

## Vulnerability
Short messages (<= 48 bytes) take the `permute_fast` path:
- No chi step, only linear rotations and XORs.
- Absorption is XOR, so the prefinal state is an affine linear function of the
  message bits.
- `permute_fast` never touches state words 8..11, so only the first 8 words
  (256 bits) depend on the message.

For a fixed length `L <= 48`, the mapping is:
```
message bits (8*L)  ->  256-bit prefinal state prefix
```
With `L = 47`, there are 376 input bits and only 256 output bits, so the kernel
has dimension at least 120. Any nonzero kernel vector gives two distinct
messages with the same prefinal state, hence the same hash after the final
`permute_full`.

## Constructing a collision
1. Fix length `L = 47` and a base message of all zeros.
2. For each input bit `i`, flip that bit, recompute the prefinal state, and store
   the 256-bit delta as a column vector.
3. Run Gaussian elimination over GF(2) to find a nonzero vector in the kernel.
4. Pick a kernel vector where the first byte is in `{1,2,3}` so it can represent
   a valid color change.

The resulting colliding messages are:
```
m0 = 00 * 47
m1 = 0284444140001000000100040021408002044400481010000100000000000000000000000000000001400000000000
```
Both hash to:
```
adc8a73bc22853b42d29343ac3e1f2a721a1e6ea19b842f8f5beff597b93c3f4
```

`m0[0] = 0` and `m1[0] = 2`, so they open as different colors but share the same
commitment.

## Attack
1. Send the same commitment for all 4 vertices (the hash above).
2. For every edge challenge, respond with `m0` and `m1` in any order. The server
   only checks that colors differ and hashes match the commitments.
3. Repeat for 128 rounds and receive the flag.

