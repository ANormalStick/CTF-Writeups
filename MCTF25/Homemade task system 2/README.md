<style>
:root {
  color-scheme: dark;
  --bg: #020617;
  --fg: #e5e7eb;
  --fg-muted: #9ca3af;
  --accent: #38bdf8;
  --border-subtle: #1f2937;
}

/* Base page */
html, body {
  margin: 0;
  padding: 0;
  background: radial-gradient(circle at top left, #020617, #020617 40%, #020617 100%);
  color: var(--fg);
  font-family: system-ui, -apple-system, BlinkMacSystemFont, "SF Pro Text", "Segoe UI", sans-serif;
}

/* GitHub Pages wrappers */
.page-content, .wrapper, article, .post {
  background: transparent !important;
  max-width: 960px;
  margin: 0 auto;
  padding: 2.5rem 1.5rem 4rem;
}

/* Headings */
.post h1, .page-content h1, article h1 {
  font-size: 1.6rem;
  margin-bottom: 0.6rem;
}

.post h2, .page-content h2, article h2 {
  font-size: 1.1rem;
  margin-top: 1.8rem;
  margin-bottom: 0.6rem;
  color: var(--fg-muted);
}

/* Tables (optional, if you use them) */
table {
  border-collapse: collapse;
  width: 100%;
  font-size: 0.85rem;
  margin: 0.4rem 0 0.8rem;
  border-radius: 0.6rem;
  overflow: hidden;
}

th,
td {
  padding: 0.5rem 0.75rem;
  border-bottom: 1px solid var(--border-subtle);
  background-color: rgba(15, 23, 42, 0.9);
}

th {
  text-align: left;
  font-weight: 500;
  color: var(--fg-muted);
}

tbody tr:nth-child(even) td {
  background-color: rgba(15, 23, 42, 0.75);
}

tbody tr:last-child td {
  border-bottom: none;
}

/* Links */
a {
  color: var(--accent);
}

a:hover {
  text-decoration: none;
}

/* Code blocks */
pre,
code,
pre code,
.highlight,
.highlight pre,
.highlight code {
  background-color: rgba(15, 23, 42, 0.96) !important;
  color: var(--fg);
}

pre {
  border: 1px solid var(--border-subtle);
  padding: 0.85rem 1rem;
  border-radius: 0.5rem;
  overflow-x: auto;
  font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;
}

code {
  padding: 0.1rem 0.25rem;
  border-radius: 0.25rem;
  font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;
}
</style>


## Homemade task system 2

**Challenge ID:** Homemade task system 2  
**Author:** Mārtiņš #1337  
**Target:** `10.240.2.232:80` (Caddy HTTP server)

### Overview

This is a follow-up to *Homemade task system*. The author “fixes” the original problem by encoding file names, claiming this is “security by encoding”. The goal is to find the hidden non-indexed step and recover the flag.

### Enumeration

Port scan:

```bash
nmap -sC -sV 10.240.2.232
```

Only HTTP (port 80) is open.

Main page (`/`) shows an XP-themed “Task Tracker II” with a *Playbook steps* list:

```html
<li><a href="/dmllbnM=.html">Prepare</a></li>
<li><a href="/ZGl2aQ==.html">Identify</a></li>
<li><a href="/dHLEq3M=.html">Secure</a></li>
<li><a href="/xI1ldHJp.html">Contain</a></li>
```

The link names themselves are suspicious: each looks like Base64.

### Decoding the paths

Decode each name:

```bash
printf 'dmllbnM=' | base64 -d; echo
printf 'ZGl2aQ==' | base64 -d; echo
printf 'dHLEq3M=' | base64 -d; echo
printf 'xI1ldHJp' | base64 -d; echo
```

Results:

- `dmllbnM=` → `viens`
- `ZGl2aQ==` → `divi`
- `dHLEq3M=` → `trīs`
- `xI1ldHJp` → `četri`

These are Latvian words for 1, 2, 3, 4.

Each page again shows progress like `Playbook Progress: N / 5`, so there should be a fifth hidden step.

### Guessing the hidden page

By analogy, the missing step should be “five” in Latvian:

- `pieci` (5) → Base64: `cGllY2k=`

So the expected hidden file is:

```bash
curl -s http://10.240.2.232/cGllY2k=.html -o 5.html
```

This returns a **“Protect – Hidden Step”** page with a “Protected artifact” field:

```text
TUNURjI1e2VuYzBkMW5nXzE1bnRfdGgzX3M0bTNfYXNfM25jcnlwdDFvbn0=
```

### Recovering the flag

The “protected artifact” is just Base64 again. Decode it:

```bash
echo 'TUNURjI1e2VuYzBkMW5nXzE1bnRfdGgzX3M0bTNfYXNfM25jcnlwdDFvbn0=' | base64 -d
```

Decoded:

```text
MCTF25{enc0d1ng_15nt_th3_s4m3_as_3ncrypt1on}
```

