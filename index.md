---
title: "Mārtiņa-CTF 2025 Writeups"
---

<style>
:root {
  color-scheme: dark;
  --bg: #020617;
  --bg-alt: #020617;
  --card: #020617;
  --border-subtle: #1f2937;
  --border-strong: #374151;
  --fg: #e5e7eb;
  --fg-muted: #9ca3af;
  --accent: #38bdf8;
  --accent-soft: rgba(56, 189, 248, 0.18);
}

html, body {
  margin: 0;
  padding: 0;
  background: radial-gradient(circle at top left, #020617, #020617 40%, #020617 100%);
  color: var(--fg);
  font-family: system-ui, -apple-system, BlinkMacSystemFont, "SF Pro Text", "Segoe UI", sans-serif;
}

/* GitHub Pages / Jekyll wrappers (best-effort overrides) */
.page-content, .wrapper, article {
  background: transparent !important;
}

/* Main layout */
.ctf-page {
  max-width: 960px;
  margin: 0 auto;
  padding: 2.5rem 1.5rem 4rem;
}

.ctf-hero {
  border-radius: 0.9rem;
  border: 1px solid var(--border-subtle);
  background: radial-gradient(circle at top left, #020617 0, #020617 55%, #020617 100%);
  padding: 1.8rem 2rem 1.6rem;
  box-shadow: 0 22px 55px rgba(0, 0, 0, 0.65);
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

.ctf-link-row a {
  color: var(--accent);
  text-decoration: none;
  border-bottom: 1px solid transparent;
}

.ctf-link-row a:hover {
  border-bottom-color: var(--accent);
}

/* Headings / sections */

.ctf-section {
  margin-top: 2.4rem;
}

.ctf-section h2 {
  font-size: 1.1rem;
  letter-spacing: 0.04em;
  text-transform: uppercase;
  color: var(--fg-muted);
  margin-bottom: 0.9rem;
}

.ctf-section h3 {
  font-size: 1rem;
  margin-bottom: 0.3rem;
}

/* Tables */

table {
  border-collapse: collapse;
  width: 100%;
  font-size: 0.85rem;
  margin: 0.4rem 0 0.8rem;
  background: rgba(15, 23, 42, 0.7);
  border-radius: 0.6rem;
  overflow: hidden;
}

thead {
  background: rgba(15, 23, 42, 0.9);
}

th, td {
  padding: 0.5rem 0.75rem;
  border-bottom: 1px solid var(--border-subtle);
}

th {
  text-align: left;
  font-weight: 500;
  color: var(--fg-muted);
}

tbody tr:last-child td {
  border-bottom: none;
}

tbody tr:nth-child(even) {
  background: rgba(15, 23, 42, 0.6);
}

tbody tr:hover td {
  background: rgba(15, 23, 42, 0.95);
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

.ctf-small {
  font-size: 0.8rem;
  color: var(--fg-muted);
}
</style>

<div class="ctf-page">

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

  <section class="ctf-section">
    <h2>CTFs</h2>

This repo currently contains writeups for one event.

| Year | CTF Name         | Alias  | Index                             |
|------|------------------|--------|-----------------------------------|
| 2025 | Mārtiņa-CTF 2025 | MCTF25 | [Browse MCTF25 writeups](./MCTF25/) |

  </section>

  <section class="ctf-section">
    <h2>MCTF25 — Category Overview</h2>

Quick navigation by category:

- [Audio / Web](#audio--web)
- [Blockchain](#blockchain)
- [Blockchain / Forensics](#blockchain--forensics)
- [Crypto](#crypto)
- [Forensics](#forensics)
- [Misc / Fun](#misc--fun)
- [Pwn / Docker](#pwn--docker)
- [Web](#web)

  </section>

  <section class="ctf-section" id="audio--web">
    <h3>Audio / Web</h3>

| Challenge      | Writeup |
|----------------|---------|
| Astral Pulses  | [README](./MCTF25/Astral%20Pulses/README.md) |
| AI Translator  | [README](./MCTF25/AI%20Translator/README.md) |
  </section>

  <section class="ctf-section" id="blockchain">
    <h3>Blockchain</h3>

| Challenge                 | Writeup |
|---------------------------|---------|
| Guess The Number          | [README](./MCTF25/%5BBlockchain%203%5D%20Guess%20The%20Number/README_Blockchain3_GuessTheNumber.md) |
| Magical RPC Button        | [README](./MCTF25/%5BBlockchain%201%5D%20Magical%20RPC%20Button/README_Blockchain1_MagicalRPCButton.md) |
| Unlimited Void            | [README](./MCTF25/Unlimited%20Void/Unlimited_Void_README.md) |
| Where Did I Leave My Flag | [README](./MCTF25/%5BBlockchain%202%5D%20Where%20Did%20I%20LeaveMyFlag/README_Blockchain2_WhereDidILeaveMyFlag.md) |
  </section>

  <section class="ctf-section" id="blockchain--forensics">
    <h3>Blockchain / Forensics</h3>

| Challenge               | Writeup |
|-------------------------|---------|
| Titanium Safe           | [index](./MCTF25/Titanium%20Safe/) |
| Sacred Martins Sequence | [README](./MCTF25/%5BBlockchain%204%5D%20Sacred%20Martins%20Sequence/readme.md) |
| Sepolia Heist           | [README](./MCTF25/Sepolia%20Heist/readme.md) |
  </section>

  <section class="ctf-section" id="crypto">
    <h3>Crypto</h3>

| Challenge               | Writeup |
|-------------------------|---------|
| Radical Security Animal | [README](./MCTF25/%5BCryptography%204%5D%20Radical%20Security%20Animal/README_crypto4.md) |
  </section>

  <section class="ctf-section" id="forensics">
    <h3>Forensics</h3>

| Challenge         | Writeup |
|-------------------|---------|
| Rewritten History | [README](./MCTF25/Rewritten%20History/README.md) |
  </section>

  <section class="ctf-section" id="misc--fun">
    <h3>Misc / Fun</h3>

| Challenge         | Writeup |
|-------------------|---------|
| A series of tubes | [README](./MCTF25/A%20series%20of%20tubes/readme.md) |
| Jokemartins       | [README](./MCTF25/Jokemartins/Readme.md) |
  </section>

  <section class="ctf-section" id="pwn--docker">
    <h3>Pwn / Docker</h3>

| Challenge                  | Writeup |
|----------------------------|---------|
| ImgSharer                  | [README](./MCTF25/ImgSharer/README.md) |
| Docker? I barely know her! | [README](./MCTF25/Docker,%20I%20barely%20know%20her!/README.md) |
  </section>

  <section class="ctf-section" id="web">
    <h3>Web</h3>

| Challenge               | Writeup |
|-------------------------|---------|
| Gatekeeper              | [README](./MCTF25/Gatekeeper/Gatekeeper-README.md) |
| Homemade task system    | [README](./MCTF25/Homemade%20task%20system/readme.md) |
| Homemade task system 2  | [README](./MCTF25/Homemade%20task%20system%202/readme.md) |
| Homemade task system 3  | [README](./MCTF25/Homemade%20task%20system%203/README.md) |
| Parent Security         | [README](./MCTF25/Parent%20Security/README.md) |
| not!Windows registry    | [README](./MCTF25/not!Windows%20registry/README.md) |
  </section>

  <section class="ctf-section">
    <h2>About this site</h2>

This site is generated with <strong>GitHub Pages</strong> from  
<a href="https://github.com/ANormalStick/CTF-Writeups">ANormalStick/CTF-Writeups</a>.

<div class="ctf-small">
More CTFs will be added to the table above as I play them; MCTF25 is the first one.
</div>

  </section>

</div>
