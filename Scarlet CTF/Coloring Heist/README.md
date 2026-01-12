<style> :root { color-scheme: dark; --bg: #020617; --fg: #e5e7eb; --fg-muted: #9ca3af; --accent: #38bdf8; --border-subtle: #1f2937; } /* Base page */ html, body { margin: 0; padding: 0; background: radial-gradient(circle at top left, #020617, #020617 40%, #020617 100%); color: var(--fg); font-family: system-ui, -apple-system, BlinkMacSystemFont, "SF Pro Text", "Segoe UI", sans-serif; } /* GitHub Pages wrappers */ .page-content, .wrapper, article, .post { background: transparent !important; max-width: 960px; margin: 0 auto; padding: 2.5rem 1.5rem 4rem; } /* Headings */ .post h1, .page-content h1, article h1 { font-size: 1.6rem; margin-bottom: 0.6rem; } .post h2, .page-content h2, article h2 { font-size: 1.1rem; margin-top: 1.8rem; margin-bottom: 0.6rem; color: var(--fg-muted); } /* Tables (optional, if you use them) */ table { border-collapse: collapse; width: 100%; font-size: 0.85rem; margin: 0.4rem 0 0.8rem; border-radius: 0.6rem; overflow: hidden; } th, td { padding: 0.5rem 0.75rem; border-bottom: 1px solid var(--border-subtle); background-color: rgba(15, 23, 42, 0.9); } th { text-align: left; font-weight: 500; color: var(--fg-muted); } tbody tr:nth-child(even) td { background-color: rgba(15, 23, 42, 0.75); } tbody tr:last-child td { border-bottom: none; } /* Links */ a { color: var(--accent); } a:hover { text-decoration: none; } /* Code blocks */ pre, code, pre code, .highlight, .highlight pre, .highlight code { background-color: rgba(15, 23, 42, 0.96) !important; color: var(--fg); } pre { border: 1px solid var(--border-subtle); padding: 0.85rem 1rem; border-radius: 0.5rem; overflow-x: auto; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } code { padding: 0.1rem 0.25rem; border-radius: 0.25rem; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } </style>

# Coloring Heist (Crypto) - Writeup

## Summary
The service implements the standard graph 3-coloring zero-knowledge protocol, but the "guess" endpoint accepts any coloring that is a permutation of the server's secret coloring. Since the provided graph is 3-colorable, we can compute any valid 3-coloring locally and submit it to get the flag.

## Files
- `E:\ctf\chal.py`
- `E:\ctf\graph.txt`

## Key Observation
The `guess` logic compares the submitted coloring to the secret `coloring` under any permutation of the three colors:

```
for perm in permutations(colors):
    if all(a == perm[b] for a, b in zip(coloring, guess_coloring)):
        return {'flag': FLAG}
```

So any valid 3-coloring of the graph works, because it must equal the secret coloring up to a color permutation if the graph's 3-coloring is unique (which this instance is).

## Approach
1. Parse `graph.txt` to build the graph.
2. Find a valid 3-coloring using DSATUR (greedy backtracking with saturation degree).
3. Connect to the server, solve its PoW, and submit the coloring via `option=guess`.

The graph has 1000 nodes and ~20k edges; DSATUR solves this quickly.

## Solver (Python)
This script computes a 3-coloring and submits it. It also handles the PoW using the `redpwnpow` binary (Windows build).

```python
import json, socket, re, subprocess, sys
from collections import defaultdict

HOST, PORT = "challs.ctf.rusec.club", 2751
POW_EXE = "E:/ctf/redpwnpow.exe"  # download from https://github.com/redpwn/pow/releases

# load graph
edges = defaultdict(set)
N = 0
for line in open("E:/ctf/graph.txt"):
    u, v = map(int, line.split())
    edges[u].add(v); edges[v].add(u)
    N = max(N, u, v)
N += 1
adj = [edges[i] for i in range(N)]
deg = [len(adj[i]) for i in range(N)]

# DSATUR 3-coloring
colors = [-1] * N
sat = [set() for _ in range(N)]
uncolored = set(range(N))
sys.setrecursionlimit(10000)

def dsatur():
    if not uncolored:
        return True
    node = max(uncolored, key=lambda i: (len(sat[i]), deg[i]))
    used = sat[node]
    for c in range(3):
        if c in used:
            continue
        colors[node] = c
        uncolored.remove(node)
        updated = []
        for nb in adj[node]:
            if colors[nb] == -1 and c not in sat[nb]:
                sat[nb].add(c)
                updated.append(nb)
        if dsatur():
            return True
        for nb in updated:
            sat[nb].remove(c)
        uncolored.add(node)
        colors[node] = -1
    return False

assert dsatur()

req = json.dumps({"option": "guess", "coloring": colors}).encode() + b"\n"

with socket.create_connection((HOST, PORT), timeout=10) as s:
    data = b""
    while b"solution:" not in data:
        data += s.recv(4096)
    chal = re.search(r"sh -s ([A-Za-z0-9+/=._-]+)", data.decode()).group(1)
    sol = subprocess.check_output([POW_EXE, chal]).strip()
    s.sendall(sol + b"\n")
    s.sendall(req)
    print(s.recv(4096).decode())
```

## Flag
`RUSEC{t0uhou_fum0_b4urs4k_orz0city_fn1x9fk3mdj1}`

