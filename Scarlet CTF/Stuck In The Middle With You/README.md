<style> :root { color-scheme: dark; --bg: #020617; --fg: #e5e7eb; --fg-muted: #9ca3af; --accent: #38bdf8; --border-subtle: #1f2937; } /* Base page */ html, body { margin: 0; padding: 0; background: radial-gradient(circle at top left, #020617, #020617 40%, #020617 100%); color: var(--fg); font-family: system-ui, -apple-system, BlinkMacSystemFont, "SF Pro Text", "Segoe UI", sans-serif; } /* GitHub Pages wrappers */ .page-content, .wrapper, article, .post { background: transparent !important; max-width: 960px; margin: 0 auto; padding: 2.5rem 1.5rem 4rem; } /* Headings */ .post h1, .page-content h1, article h1 { font-size: 1.6rem; margin-bottom: 0.6rem; } .post h2, .page-content h2, article h2 { font-size: 1.1rem; margin-top: 1.8rem; margin-bottom: 0.6rem; color: var(--fg-muted); } /* Tables (optional, if you use them) */ table { border-collapse: collapse; width: 100%; font-size: 0.85rem; margin: 0.4rem 0 0.8rem; border-radius: 0.6rem; overflow: hidden; } th, td { padding: 0.5rem 0.75rem; border-bottom: 1px solid var(--border-subtle); background-color: rgba(15, 23, 42, 0.9); } th { text-align: left; font-weight: 500; color: var(--fg-muted); } tbody tr:nth-child(even) td { background-color: rgba(15, 23, 42, 0.75); } tbody tr:last-child td { border-bottom: none; } /* Links */ a { color: var(--accent); } a:hover { text-decoration: none; } /* Code blocks */ pre, code, pre code, .highlight, .highlight pre, .highlight code { background-color: rgba(15, 23, 42, 0.96) !important; color: var(--fg); } pre { border: 1px solid var(--border-subtle); padding: 0.85rem 1rem; border-radius: 0.5rem; overflow-x: auto; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } code { padding: 0.1rem 0.25rem; border-radius: 0.25rem; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } </style>

# OSINT: Stuck In The Middle With You

## Challenge Description

> We're trying to figure out how to track this Tor traffic but all we've got is this string, A68097FE97D3065B1A6F4CE7187D753F8B8513F5! We don't know what to do with it. We're looking for someone responsible for hosting multiple nodes. Can you find the IPv4 addresses this node and any of its effective family members?
>
> FLAG FORMAT: RUSEC{family_ip1:family_ip2:...:family_ipX} for X family members
>
> The flag will be the IPs of the node and all the associated family members in order of oldest node to youngest, based on when they were first seen, separated by colons.

## Solution

### Step 1: Identify the String

The provided string `A68097FE97D3065B1A6F4CE7187D753F8B8513F5` is a 40-character hexadecimal string, which is the format of a **Tor relay fingerprint**. Tor relays are identified by their unique RSA SHA1 fingerprint.

### Step 2: Look Up the Relay

We can look up Tor relay information using the [Tor Metrics Relay Search](https://metrics.torproject.org/rs.html) or directly query the [Onionoo API](https://onionoo.torproject.org/).

Query URL:
```
https://onionoo.torproject.org/details?lookup=A68097FE97D3065B1A6F4CE7187D753F8B8513F5
```

This returns details about the relay **olabobamanmu**:
- **IPv4:** `51.15.40.38`
- **First Seen:** `2020-04-03`
- **Contact:** `giannoug@gmail.com`
- **Effective Family:** Contains 3 fingerprints

### Step 3: Find Family Members

The relay's `effective_family` field lists all family members:
1. `414E64BA607560F9D9C196A825950DC968700420`
2. `A68097FE97D3065B1A6F4CE7187D753F8B8513F5` (original)
3. `B4CAFD9CBFB34EC5DAAC146920DC7DFAFE91EA20`

Looking up each family member:

| Fingerprint | Nickname | IPv4 Address | First Seen |
|------------|----------|--------------|------------|
| `B4CAFD9CBFB34EC5...` | netimanmu | 212.47.233.86 | **2019-02-18** |
| `A68097FE97D3065B...` | olabobamanmu | 51.15.40.38 | **2020-04-03** |
| `414E64BA607560F9...` | kanemeadminmanmu | 151.115.73.55 | **2024-12-29** |

All three relays are operated by the same person (`giannoug`) and hosted on Scaleway infrastructure.

### Step 4: Order by Age

Sorting from **oldest** to **youngest** based on `first_seen`:

1. `212.47.233.86` — first seen 2019-02-18 (oldest)
2. `51.15.40.38` — first seen 2020-04-03
3. `151.115.73.55` — first seen 2024-12-29 (youngest)

## Flag

```
RUSEC{212.47.233.86:51.15.40.38:151.115.73.55}
```

## Tools & Resources

- [Tor Metrics Relay Search](https://metrics.torproject.org/rs.html)
- [Onionoo API Documentation](https://metrics.torproject.org/onionoo.html)
