<style> :root { color-scheme: dark; --bg: #020617; --fg: #e5e7eb; --fg-muted: #9ca3af; --accent: #38bdf8; --border-subtle: #1f2937; } /* Base page */ html, body { margin: 0; padding: 0; background: radial-gradient(circle at top left, #020617, #020617 40%, #020617 100%); color: var(--fg); font-family: system-ui, -apple-system, BlinkMacSystemFont, "SF Pro Text", "Segoe UI", sans-serif; } /* GitHub Pages wrappers */ .page-content, .wrapper, article, .post { background: transparent !important; max-width: 960px; margin: 0 auto; padding: 2.5rem 1.5rem 4rem; } /* Headings */ .post h1, .page-content h1, article h1 { font-size: 1.6rem; margin-bottom: 0.6rem; } .post h2, .page-content h2, article h2 { font-size: 1.1rem; margin-top: 1.8rem; margin-bottom: 0.6rem; color: var(--fg-muted); } /* Tables (optional, if you use them) */ table { border-collapse: collapse; width: 100%; font-size: 0.85rem; margin: 0.4rem 0 0.8rem; border-radius: 0.6rem; overflow: hidden; } th, td { padding: 0.5rem 0.75rem; border-bottom: 1px solid var(--border-subtle); background-color: rgba(15, 23, 42, 0.9); } th { text-align: left; font-weight: 500; color: var(--fg-muted); } tbody tr:nth-child(even) td { background-color: rgba(15, 23, 42, 0.75); } tbody tr:last-child td { border-bottom: none; } /* Links */ a { color: var(--accent); } a:hover { text-decoration: none; } /* Code blocks */ pre, code, pre code, .highlight, .highlight pre, .highlight code { background-color: rgba(15, 23, 42, 0.96) !important; color: var(--fg); } pre { border: 1px solid var(--border-subtle); padding: 0.85rem 1rem; border-radius: 0.5rem; overflow-x: auto; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } code { padding: 0.1rem 0.25rem; border-radius: 0.25rem; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } </style>

# Gambler's Fallacy

## Summary
The game uses Python's MT19937 (`random`) to generate a 32-bit `server_seed` for each roll, and it prints that seed directly. After collecting 624 outputs, we can reconstruct the MT internal state via untempering, predict all future `server_seed` values, compute exact rolls from the HMAC formula, and bet the full balance on guaranteed wins until the balance exceeds 10000 to buy the flag.

## Key Observations
- `server_seed = random.getrandbits(32)` is revealed every round.
- MT19937 has a 624-word internal state; 624 outputs fully recover it.
- The roll is deterministic from `(server_seed, client_seed, nonce)`:
  - `sig = HMAC_SHA256(key=str(server_seed), msg=f"{client_seed}-{nonce}")`
  - Take 5-hex-digit chunks until `< 1e6`
  - `roll = round((lucky % 10000) * 1e-2)` (integer 0..99)

## Attack Plan
1. Play 624 games with minimal wager to collect 624 `server_seed` values.
2. Untemper each output to recover the MT state.
3. Predict future `server_seed` values.
4. For each predicted seed, compute the exact roll using the HMAC formula.
5. If roll is in `[2..98]`, set `greed = roll` and wager the full balance to win for sure.
6. Otherwise, place a minimum bet to advance the nonce and consume the RNG output.
7. Repeat until balance >= 10000, then buy the flag.

## Untemper (MT19937)
Temper in Python is:
```
y ^= (y >> 11)
y ^= (y << 7)  & 0x9d2c5680
y ^= (y << 15) & 0xefc60000
y ^= (y >> 18)
```
Each step is reversible with iterative xor-unshifting, yielding the original state word.

## Result
Flag: `uoftctf{ez_m3rs3nne_untwisting!!}`
