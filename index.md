---
title: "Mārtiņa-CTF 2025 Writeups"
---

<style>
:root {
  color-scheme: dark;
  --bg: #020617;
  --border-subtle: #1f2937;
  --fg: #e5e7eb;
  --fg-muted: #9ca3af;
  --accent: #38bdf8;
}

/* Base page styling */
html, body {
  margin: 0;
  padding: 0;
  background: radial-gradient(circle at top left, #020617, #020617 40%, #020617 100%);
  color: var(--fg);
  font-family: system-ui, -apple-system, BlinkMacSystemFont, "SF Pro Text", "Segoe UI", sans-serif;
}

/* GitHub Pages / Jekyll wrappers */
.page-content, .wrapper, article, .post {
  background: transparent !important;
  max-width: 960px;
  margin: 0 auto;
  padding: 2.5rem 1.5rem 4rem;
}

/* Hero */

.ctf-hero {
  border-radius: 0.9rem;
  border: 1px solid var(--border-subtle);
  background: radial-gradient(circle at top left, #020617 0, #020617 55%, #020617 100%);
  padding: 1.8rem 2rem 1.6rem;
  box-shadow: 0 22px 55px rgba(0, 0, 0, 0.65);
  margin-bottom: 2.4rem;
}

.ctf-hero-title {
  font-size: 1.6rem;
  letter-spacing: 0.03em;
  margin: 0 0 0.25rem;
}

.ctf-hero-subtitle {
  font-size: 0.95rem;
  color: var(--fg-muted);
  margin-bottom: 1.3rem;
}

.ctf-hero-meta {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
  font-size: 0.75rem;
  color: var(--fg-muted);
}

.ctf-pill {
  border-radius: 999px;
  padding: 0.25rem 0.7rem;
  border: 1px solid var(--border-subtle);
  background: linear-gradient(to right, rgba(15, 23, 42, 0.6), rgba(15, 23, 42, 0.2));
}

.ctf-link-row {
  margin-top: 1rem;
  font-size: 0.85rem;
}

/* Headings */

.post h2, .page-content h2, article h2 {
  font-size: 1.1rem;
  letter-spacing: 0.04em;
  text-transform: uppercase;
  color: var(--fg-muted);
  margin-top: 2.2rem;
  margin-bottom: 0.8rem;
}

.post h3, .page-content h3, article h3 {
  font-size: 1rem;
  margin-top: 1.6rem;
  margin-bottom: 0.3rem;
}

/* Tables – force dark cells */

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
  background-color: rgba(15, 23, 42, 0.9); /* kill the white */
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

tbody tr:hover td {
  background-color: rgba(15, 23, 42, 0.98);
}

/* Links */

a {
  color: var(--accent);
}

a:hover {
  text-decoration: none;
}

/* Misc */

hr {
  border: 0;
  border-top: 1px solid var(--border-subtle);
  margin: 2rem 0 1.5rem;
}

code, pre {
  font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;
}
</style>

<section class="ctf-hero">
  <div class="ctf-hero-title">ANormalStick / CTF Writeups</div>
  <div class="ctf-hero-subtitle">
    Mārtiņa-CTF 2025 (MCTF25) — first CTF, documented challenge by challenge.
  </div>

  <div class="ctf-hero-meta">
    <div class="ctf-pill">CTF: Mārtiņa-CTF 2025</div>
    <div class="ctf-pill">Focus: blockchain · web · forensics · misc</div>
    <div class="ctf-pill">Writeups: per challenge</div>
  </div>

  <div class="ctf-link-row">
    Source: <a href="https://github.com/ANormalStick/CTF-Writeups">ANormalStick/CTF-Writeups</a>
  </div>
</section>

## CTFs

This repo currently contains writeups for one event.

| Year | CTF Name         | Alias  | Index                             |
|------|------------------|--------|-----------------------------------|
| 2025 | Mārtiņa-CTF 2025 | MCTF25 | [Browse MCTF25 writeups](./MCTF25/) |

## MCTF25 — Category Overview

Quick navigation by category:

- [Audio / Web](#audio--web)
- [Blockchain](#blockchain)
- [Blockchain / Forensics](#blockchain--forensics)
- [Crypto](#crypto)
- [Forensics](#forensics)
- [Misc / Fun](#misc--fun)
- [Pwn / Docker](#pwn--docker)
- [Web](#web)

### Audio / Web

| Challenge     | Writeup                                                   |
|--------------|-----------------------------------------------------------|
| Astral Pulses | [index](./MCTF25/Astral%20Pulses/)             |
| AI Translator | [README](./MCTF25/AI%20Translator/)             |

### Blockchain

| Challenge                 | Writeup                                                                 |
|---------------------------|-------------------------------------------------------------------------|
| Guess The Number          | [README](./MCTF25/%5BBlockchain%203%5D%20Guess%20The%20Number/) |
| Magical RPC Button        | [README](./MCTF25/%5BBlockchain%201%5D%20Magical%20RPC%20Button/) |
| Unlimited Void            | [README](./MCTF25/Unlimited%20Void/)           |
| Where Did I Leave My Flag | [README](./MCTF25/%5BBlockchain%202%5D%20Where%20Did%20I%20Leave%20My%20Flag/) |

### Blockchain / Forensics

| Challenge               | Writeup                                                                 |
|-------------------------|-------------------------------------------------------------------------|
| Titanium Safe           | [README](./MCTF25/Titanium%20Safe/)                                     |
| Sacred Martins Sequence | [README](./MCTF25/%5BBlockchain%204%5D%20Sacred%20Martins%20Sequence/) |
| Sepolia Heist           | [README](./MCTF25/Sepolia%20Heist/)                           |

### Crypto

| Challenge               | Writeup                                                                 |
|-------------------------|-------------------------------------------------------------------------|
| Radical Security Animal | [README](./MCTF25/%5BCryptography%204%5D%20Radical%20Security%20Animal/) |

### Forensics

| Challenge         | Writeup                                             |
|-------------------|-----------------------------------------------------|
| Rewritten History | [README](./MCTF25/Rewritten%20History/)    |

### Misc / Fun

| Challenge         | Writeup                                             |
|-------------------|-----------------------------------------------------|
| A series of tubes | [README](./MCTF25/A%20series%20of%20tubes/) |
| Jokemartins       | [README](./MCTF25/Jokemartins/)            |

### Pwn / Docker

| Challenge                  | Writeup                                                              |
|----------------------------|----------------------------------------------------------------------|
| ImgSharer                  | [README](./MCTF25/ImgSharer/)                              |
| Docker? I barely know her! | [README](./MCTF25/Docker,%20I%20barely%20know%20her!/)     |

### Web

| Challenge              | Writeup                                                              |
|------------------------|----------------------------------------------------------------------|
| Gatekeeper             | [README](./MCTF25/Gatekeeper/)                  |
| Homemade task system   | [README](./MCTF25/Homemade%20task%20system/)               |
| Homemade task system 2 | [README](./MCTF25/Homemade%20task%20system%202/)           |
| Homemade task system 3 | [README](./MCTF25/Homemade%20task%20system%203/)           |
| Parent Security        | [README](./MCTF25/Parent%20Security/)                      |
| not!Windows registry   | [README](./MCTF25/not!Windows%20registry/)                 |

## About this site

This site is generated with **GitHub Pages** from  
[`ANormalStick/CTF-Writeups`](https://github.com/ANormalStick/CTF-Writeups).

MCTF25 is the first event; more CTFs will be added here over time.
