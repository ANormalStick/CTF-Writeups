<style> :root { color-scheme: dark; --bg: #020617; --fg: #e5e7eb; --fg-muted: #9ca3af; --accent: #38bdf8; --border-subtle: #1f2937; } /* Base page */ html, body { margin: 0; padding: 0; background: radial-gradient(circle at top left, #020617, #020617 40%, #020617 100%); color: var(--fg); font-family: system-ui, -apple-system, BlinkMacSystemFont, "SF Pro Text", "Segoe UI", sans-serif; } /* GitHub Pages wrappers */ .page-content, .wrapper, article, .post { background: transparent !important; max-width: 960px; margin: 0 auto; padding: 2.5rem 1.5rem 4rem; } /* Headings */ .post h1, .page-content h1, article h1 { font-size: 1.6rem; margin-bottom: 0.6rem; } .post h2, .page-content h2, article h2 { font-size: 1.1rem; margin-top: 1.8rem; margin-bottom: 0.6rem; color: var(--fg-muted); } /* Tables (optional, if you use them) */ table { border-collapse: collapse; width: 100%; font-size: 0.85rem; margin: 0.4rem 0 0.8rem; border-radius: 0.6rem; overflow: hidden; } th, td { padding: 0.5rem 0.75rem; border-bottom: 1px solid var(--border-subtle); background-color: rgba(15, 23, 42, 0.9); } th { text-align: left; font-weight: 500; color: var(--fg-muted); } tbody tr:nth-child(even) td { background-color: rgba(15, 23, 42, 0.75); } tbody tr:last-child td { border-bottom: none; } /* Links */ a { color: var(--accent); } a:hover { text-decoration: none; } /* Code blocks */ pre, code, pre code, .highlight, .highlight pre, .highlight code { background-color: rgba(15, 23, 42, 0.96) !important; color: var(--fg); } pre { border: 1px solid var(--border-subtle); padding: 0.85rem 1rem; border-radius: 0.5rem; overflow-x: auto; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } code { padding: 0.1rem 0.25rem; border-radius: 0.25rem; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } </style>

# extended-eBPF

## Summary

The service runs a custom kernel with an "extended" eBPF verifier bug. A BPF program can build an arbitrary 64-bit offset using a non-constant shift, then add it to a map value pointer without a proper bounds check. This gives out-of-bounds read/write relative to the map value and lets us locate and overwrite the current task's `cred` struct to become root and read `/flag`.

## Vulnerability

The program lets a user provide a list of `(shift, enable)` slots. The BPF program computes:

```
offset = sum_i ( (1ULL << (shift_i & 63)) * (enable_i & 1) )
```

The verifier accepts the non-constant shift and the accumulated scalar, then allows adding that scalar to a map value pointer. With 64 slots this builds any 64-bit value, so we can point outside the map value and read/write arbitrary kernel memory near the map allocation. This is the "non-constant shift is too big of an extension" bug hinted by the flag.

## Exploit

1) Create a BPF array map with a large value size. The map value is a control structure:

- `op` selects read or write.
- `write_val` is the 64-bit value to store.
- `slots[64]` encodes the offset bits.
- `out[0x400]` receives read data.

2) Load a BPF program that:

- Looks up the map value.
- Computes `offset` from the slots.
- Sets `ptr = map_value + offset`.
- If `op == 1`, writes `write_val` to `*ptr`.
- Always copies 0x400 bytes from `ptr` into `out`.

3) Use the read primitive to scan memory around the map value for a `cred` struct. The scan checks:

- `usage` is a small refcount.
- UID/GID fields match the current user.
- `securebits == 0`.
- Capabilities match values from `capget`, `PR_CAPBSET_READ`, and `PR_CAP_AMBIENT`.

4) When a candidate matches, overwrite its fields:

- Set uid/gid fields to 0.
- Set capability masks to all ones.

If `getuid()` becomes 0, spawn a shell and read the flag. If not, restore the original fields and keep scanning.

## Reproduction

From the challenge directory:

```bash
gcc -O2 -s exploit.c -o exploit
python run_exploit.py
```

The script solves the PoW, logs in as `ctf`, uploads the binary, and runs it. You should see `uid=0(root)` and the flag.

## Flag

```
uoftctf{n0n_c0ns74n7_shif7_is_700_big_0f_4n_3x73nsi0n}
```
