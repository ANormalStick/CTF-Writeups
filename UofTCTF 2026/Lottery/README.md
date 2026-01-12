<style> :root { color-scheme: dark; --bg: #020617; --fg: #e5e7eb; --fg-muted: #9ca3af; --accent: #38bdf8; --border-subtle: #1f2937; } /* Base page */ html, body { margin: 0; padding: 0; background: radial-gradient(circle at top left, #020617, #020617 40%, #020617 100%); color: var(--fg); font-family: system-ui, -apple-system, BlinkMacSystemFont, "SF Pro Text", "Segoe UI", sans-serif; } /* GitHub Pages wrappers */ .page-content, .wrapper, article, .post { background: transparent !important; max-width: 960px; margin: 0 auto; padding: 2.5rem 1.5rem 4rem; } /* Headings */ .post h1, .page-content h1, article h1 { font-size: 1.6rem; margin-bottom: 0.6rem; } .post h2, .page-content h2, article h2 { font-size: 1.1rem; margin-top: 1.8rem; margin-bottom: 0.6rem; color: var(--fg-muted); } /* Tables (optional, if you use them) */ table { border-collapse: collapse; width: 100%; font-size: 0.85rem; margin: 0.4rem 0 0.8rem; border-radius: 0.6rem; overflow: hidden; } th, td { padding: 0.5rem 0.75rem; border-bottom: 1px solid var(--border-subtle); background-color: rgba(15, 23, 42, 0.9); } th { text-align: left; font-weight: 500; color: var(--fg-muted); } tbody tr:nth-child(even) td { background-color: rgba(15, 23, 42, 0.75); } tbody tr:last-child td { border-bottom: none; } /* Links */ a { color: var(--accent); } a:hover { text-decoration: none; } /* Code blocks */ pre, code, pre code, .highlight, .highlight pre, .highlight code { background-color: rgba(15, 23, 42, 0.96) !important; color: var(--fg); } pre { border: 1px solid var(--border-subtle); padding: 0.85rem 1rem; border-radius: 0.5rem; overflow-x: auto; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } code { padding: 0.1rem 0.25rem; border-radius: 0.25rem; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } </style>

# Lottery 

## Overview
`lottery.sh` uses `let "g = 0x$guess"` with a regex that only checks the *prefix* is hex. That means we can inject extra arithmetic expressions. Bash also exposes its command hash table via the `BASH_CMDS` associative array. If we poison the hash for `md5sum`, bash will try to execute that bogus path and *won't* fall back to `PATH`, so the pipeline produces no output and the ticket becomes 0.

## Key Bug
- Input is only prefix-validated (`^[0-9a-fA-F]+`), so `guess` can contain additional arithmetic.
- `BASH_CMDS[cmd]=...` lets us override the hashed path for `cmd`.
- If `md5sum` fails to exec, `ticket` is empty and `let "t = 0x$ticket"` yields `t=0`.

## Exploit
Send a guess that:
- sets `g` to 0, and
- poisons the md5sum hash entry.

Payload:

```
0,BASH_CMDS[md5sum]=0
```

This makes `md5sum` execute path `0` (which fails), so the ticket is empty and `t=0`. Since `g=0`, the comparison succeeds and the flag is printed.

## Solve Steps
1. Connect and solve the PoW prompt.
2. Send the payload above as the guess.
3. Read the flag.

Example:

```
nc 35.245.30.212 5000
# solve PoW ...
# then send:
0,BASH_CMDS[md5sum]=0
```

## Flag

```
uoftctf{you_won_the_LETtery_(hahahaha_get_it???)}
```
