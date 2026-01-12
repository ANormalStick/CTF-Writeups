<style> :root { color-scheme: dark; --bg: #020617; --fg: #e5e7eb; --fg-muted: #9ca3af; --accent: #38bdf8; --border-subtle: #1f2937; } /* Base page */ html, body { margin: 0; padding: 0; background: radial-gradient(circle at top left, #020617, #020617 40%, #020617 100%); color: var(--fg); font-family: system-ui, -apple-system, BlinkMacSystemFont, "SF Pro Text", "Segoe UI", sans-serif; } /* GitHub Pages wrappers */ .page-content, .wrapper, article, .post { background: transparent !important; max-width: 960px; margin: 0 auto; padding: 2.5rem 1.5rem 4rem; } /* Headings */ .post h1, .page-content h1, article h1 { font-size: 1.6rem; margin-bottom: 0.6rem; } .post h2, .page-content h2, article h2 { font-size: 1.1rem; margin-top: 1.8rem; margin-bottom: 0.6rem; color: var(--fg-muted); } /* Tables (optional, if you use them) */ table { border-collapse: collapse; width: 100%; font-size: 0.85rem; margin: 0.4rem 0 0.8rem; border-radius: 0.6rem; overflow: hidden; } th, td { padding: 0.5rem 0.75rem; border-bottom: 1px solid var(--border-subtle); background-color: rgba(15, 23, 42, 0.9); } th { text-align: left; font-weight: 500; color: var(--fg-muted); } tbody tr:nth-child(even) td { background-color: rgba(15, 23, 42, 0.75); } tbody tr:last-child td { border-bottom: none; } /* Links */ a { color: var(--accent); } a:hover { text-decoration: none; } /* Code blocks */ pre, code, pre code, .highlight, .highlight pre, .highlight code { background-color: rgba(15, 23, 42, 0.96) !important; color: var(--fg); } pre { border: 1px solid var(--border-subtle); padding: 0.85rem 1rem; border-radius: 0.5rem; overflow-x: auto; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } code { padding: 0.1rem 0.25rem; border-radius: 0.25rem; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } </style>

# :(

**Points:** 498  
**Solves:** 3  
**File:** `sad_face.zip` → `Challenge.evtx`

## What the hint tells us

> “kept sending my machine a payload that made my screen go blue…”

A “blue screen payload” strongly suggests **EternalBlue**, a well-known exploit that targets **SMBv1**.  
So we expect the flag to reference something like `eternal_blu3` and/or `smbv1`.

## 1) Unzip and identify the artifact

The zip only contains a single file:

- `Challenge.evtx` — a Windows **Event Log** file

## 2) Extract suspicious strings

Because `.evtx` is a binary container, the quickest first pass is to pull out printable strings and look for:
- URLs / IPs
- PowerShell
- base64 blobs (very common in logging/artifacts)

Example approaches:

```bash
unzip sad_face.zip
strings -n 12 Challenge.evtx | less
```

While scrolling through the output, several **base64-looking** strings appear, e.g. things ending with `==`.

## 3) Identify & decode base64 blobs

Filter the extracted strings for base64 patterns and attempt decoding them.  
Three of the blobs decode cleanly into readable ASCII:

- `UlVTRUN7M3Rlcm5hbF9ibHUzXw==` → `RUSEC{3ternal_blu3_`
- `c0BkX2ZhYzNfc21idg==` → `s@d_fac3_smbv`
- `MV8zODkwY24yazI5fQ==` → `1_3890cn2k29}`

## 4) Reconstruct the flag

Concatenate the decoded fragments in order:

`RUSEC{3ternal_blu3_` + `s@d_fac3_smbv` + `1_3890cn2k29}`

✅ **Flag:**

```
RUSEC{3ternal_blu3_s@d_fac3_smbv1_3890cn2k29}
```

