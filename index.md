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
  background: radial-gradient(circle at top left, #0b1120 0, #020617 55%, #000 120%);
  color: var(--fg);
  font-family: system-ui, -apple-system, BlinkMacSystemFont, "SF Pro Text", "Segoe UI", sans-serif;
  line-height: 1.6;
}

body {
  animation: ctf-fade-in 0.5s ease-out;
}

/* GitHub Pages / Jekyll wrappers */
.page-content, .wrapper, article, .post {
  background: transparent !important;
  max-width: 960px;
  margin: 0 auto;
  padding: 2.5rem 1.5rem 4rem;
}

/* Smooth anchor scrolling */
html {
  scroll-behavior: smooth;
}

/* Hero */

.ctf-hero {
  position: relative;
  border-radius: 0.9rem;
  border: 1px solid var(--border-subtle);
  background: radial-gradient(circle at top left, #020617 0, #020617 55%, #020617 100%);
  padding: 1.8rem 2rem 1.6rem;
  box-shadow: 0 22px 55px rgba(0, 0, 0, 0.65);
  margin-bottom: 2.4rem;
  overflow: hidden;
  transform: translateY(0);
  transition:
    transform 0.18s ease-out,
    box-shadow 0.18s ease-out,
    border-color 0.18s ease-out,
    background 0.25s ease-out;
}

.ctf-hero::before {
  content: "";
  position: absolute;
  inset: -40%;
  background: conic-gradient(
    from 180deg,
    rgba(56, 189, 248, 0.12),
    rgba(129, 140, 248, 0.08),
    rgba(45, 212, 191, 0.12),
    rgba(56, 189, 248, 0.12)
  );
  opacity: 0;
  transform: rotate(0deg);
  animation: ctf-hero-orbit 18s linear infinite;
  pointer-events: none;
}

.ctf-hero:hover {
  transform: translateY(-3px);
  box-shadow: 0 26px 65px rgba(0, 0, 0, 0.85);
  border-color: rgba(56, 189, 248, 0.6);
  background: radial-gradient(circle at top left, #020617 0, #020617 40%, #020617 100%);
}

.ctf-hero:hover::before {
  opacity: 1;
}

.ctf-hero > * {
  position: relative;
  z-index: 1;
}

.ctf-hero-title {
  font-size: 1.7rem;
  letter-spacing: 0.04em;
  font-weight: 600;
  margin: 0 0 0.35rem;
}

.ctf-hero-subtitle {
  font-size: 0.97rem;
  color: var(--fg-muted);
  margin-bottom: 1.4rem;
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
  background: linear-gradient(to right, rgba(15, 23, 42, 0.9), rgba(15, 23, 42, 0.4));
  white-space: nowrap;
  backdrop-filter: blur(6px);
  transition:
    border-color 0.18s ease-out,
    background 0.18s ease-out,
    transform 0.18s ease-out;
}

.ctf-pill:hover {
  border-color: rgba(56, 189, 248, 0.7);
  background: linear-gradient(to right, rgba(15, 23, 42, 0.95), rgba(15, 23, 42, 0.7));
  transform: translateY(-1px);
}

.ctf-link-row {
  margin-top: 1rem;
  font-size: 0.85rem;
}

/* Headings */

.post h1, .page-content h1, article h1 {
  font-size: 1.4rem;
  margin: 0 0 0.8rem;
}

.post h2, .page-content h2, article h2 {
  font-size: 1.05rem;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  color: var(--fg-muted);
  margin-top: 2.2rem;
  margin-bottom: 0.8rem;
}

.post h3, .page-content h3, article h3 {
  font-size: 1rem;
  margin-top: 1.8rem;
  margin-bottom: 0.8rem;
  position: relative;
}

/* fancy little bar under category headings */
.post h3::after,
.page-content h3::after,
article h3::after {
  content: "";
  display: block;
  width: 56px;
  height: 2px;
  margin-top: 0.25rem;
  border-radius: 999px;
  background: linear-gradient(to right, #38bdf8, #22c55e, #a855f7);
  opacity: 0.9;
  transform-origin: left;
  transform: scaleX(0.6);
  animation: ctf-heading-glow 1.2s ease-out forwards;
}

/* Tables – compact cards + glowing buttons */

table {
  border-collapse: collapse;
  width: 100%;
  max-width: 520px;
  font-size: 0.85rem;
  margin: 0.6rem auto 1.4rem;
  border-radius: 0.9rem;
  overflow: hidden;
  background: radial-gradient(circle at top left, rgba(15, 23, 42, 0.98), rgba(15, 23, 42, 0.94));
  border: 1px solid rgba(148, 163, 184, 0.2);
  box-shadow: 0 18px 45px rgba(0, 0, 0, 0.6);
}

thead {
  background: linear-gradient(to right, rgba(15, 23, 42, 0.98), rgba(30, 64, 175, 0.85));
}

th,
td {
  padding: 0.55rem 0.9rem;
  border-bottom: 1px solid var(--border-subtle);
}

th {
  text-align: left;
  font-weight: 500;
  color: var(--fg-muted);
  font-size: 0.78rem;
  text-transform: uppercase;
  letter-spacing: 0.08em;
}

tbody td:first-child {
  width: 100%;
}

tbody td:last-child {
  white-space: nowrap;
  text-align: right;
}

tbody tr:nth-child(even) td {
  background-color: rgba(15, 23, 42, 0.94);
}

tbody tr:nth-child(odd) td {
  background-color: rgba(15, 23, 42, 0.98);
}

tbody tr:last-child td {
  border-bottom: none;
}

tbody tr {
  transition:
    background-color 0.16s ease-out,
    transform 0.12s ease-out,
    box-shadow 0.16s ease-out;
}

tbody tr:hover {
  transform: translateY(-1px);
  box-shadow: 0 14px 40px rgba(8, 47, 73, 0.7);
}

tbody tr:hover td {
  background-color: rgba(15, 23, 42, 1);
  box-shadow: inset 2px 0 0 var(--accent);
}

/* README button styling inside tables */

td a {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: 0.28rem 0.9rem;
  border-radius: 0.45rem;
  font-size: 0.78rem;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  border: 1px solid rgba(56, 189, 248, 0.7);
  background: radial-gradient(circle at top, rgba(56, 189, 248, 0.2), rgba(15, 23, 42, 0.9));
  box-shadow: 0 0 0 1px rgba(15, 23, 42, 1), 0 0 16px rgba(56, 189, 248, 0.18);
  transition:
    transform 0.14s ease-out,
    box-shadow 0.16s ease-out,
    border-color 0.16s ease-out,
    background 0.16s ease-out;
}

/* kill underline animation inside tables */
td a::after {
  display: none !important;
}

td a:hover {
  transform: translateY(-1px);
  border-color: rgba(56, 189, 248, 1);
  background: radial-gradient(circle at top, rgba(56, 189, 248, 0.35), rgba(15, 23, 42, 0.95));
  box-shadow:
    0 0 0 1px rgba(15, 23, 42, 1),
    0 0 24px rgba(56, 189, 248, 0.4);
}

/* Links – animated underline (outside tables) */

a {
  color: var(--accent);
  text-decoration: none;
  position: relative;
}

a::after {
  content: "";
  position: absolute;
  left: 0;
  bottom: -2px;
  width: 0;
  height: 1px;
  background: var(--accent);
  transition: width 0.18s ease-out;
}

a:hover::after {
  width: 100%;
}

/* Lists */

ul {
  padding-left: 1.2rem;
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

/* Small screens */

@media (max-width: 640px) {
  .page-content, .wrapper, article, .post {
    padding: 1.8rem 1.1rem 3rem;
  }

  .ctf-hero {
    padding: 1.5rem 1.3rem 1.4rem;
  }

  table {
    max-width: 100%;
    box-shadow: 0 12px 30px rgba(0, 0, 0, 0.6);
  }
}

/* Animations */

@keyframes ctf-fade-in {
  from {
    opacity: 0;
    transform: translateY(4px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes ctf-hero-orbit {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

@keyframes ctf-heading-glow {
  0% {
    opacity: 0;
    transform: scaleX(0.2);
  }
  60% {
    opacity: 1;
    transform: scaleX(1.05);
  }
  100% {
    opacity: 0.9;
    transform: scaleX(1);
  }
}
</style>

<section class="ctf-hero">
  <div class="ctf-hero-title">ANormalStick / CTF Writeups</div>
  <div class="ctf-hero-subtitle">
    Mārtiņa-CTF 2025 (MCTF25) — my first CTF, documented challenge by challenge.
  </div>

  <div class="ctf-hero-meta">
    <div class="ctf-pill">CTF: Mārtiņa-CTF 2025</div>
    <div class="ctf-pill">Focus: blockchain · web · forensics · misc</div>
    <div class="ctf-pill">Format: per-challenge writeups</div>
  </div>

  <div class="ctf-link-row">
    Source:&nbsp;<a href="https://github.com/ANormalStick/CTF-Writeups">ANormalStick/CTF-Writeups</a>
  </div>
</section>

## CTFs

This repository currently contains writeups for one event.

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

| Challenge      | Writeup                             |
|----------------|-------------------------------------|
| Astral Pulses  | [README](./MCTF25/Astral%20Pulses/) |
| AI Translator  | [README](./MCTF25/AI%20Translator/) |

### Blockchain

| Challenge                 | Writeup                                                                 |
|---------------------------|-------------------------------------------------------------------------|
| Guess The Number          | [README](./MCTF25/%5BBlockchain%203%5D%20Guess%20The%20Number/)         |
| Magical RPC Button        | [README](./MCTF25/%5BBlockchain%201%5D%20Magical%20RPC%20Button/)       |
| Unlimited Void            | [README](./MCTF25/Unlimited%20Void/)                                    |
| Where Did I Leave My Flag | [README](./MCTF25/%5BBlockchain%202%5D%20Where%20Did%20I%20Leave%20My%20Flag/) |

### Blockchain / Forensics

| Challenge               | Writeup                                                                 |
|-------------------------|-------------------------------------------------------------------------|
| Titanium Safe           | [README](./MCTF25/Titanium%20Safe/)                                     |
| Sacred Martins Sequence | [README](./MCTF25/%5BBlockchain%204%5D%20Sacred%20Martins%20Sequence/)  |
| Sepolia Heist           | [README](./MCTF25/Sepolia%20Heist/)                                     |

### Crypto

| Challenge               | Writeup                                                                 |
|-------------------------|-------------------------------------------------------------------------|
| Radical Security Animal | [README](./MCTF25/%5BCryptography%204%5D%20Radical%20Security%20Animal/) |

### Forensics

| Challenge         | Writeup                                  |
|-------------------|------------------------------------------|
| Rewritten History | [README](./MCTF25/Rewritten%20History/)  |

### Misc / Fun

| Challenge         | Writeup                                      |
|-------------------|----------------------------------------------|
| A series of tubes | [README](./MCTF25/A%20series%20of%20tubes/)  |
| Jokemartins       | [README](./MCTF25/Jokemartins/)              |

### Pwn / Docker

| Challenge                  | Writeup                                        |
|----------------------------|------------------------------------------------|
| ImgSharer                  | [README](./MCTF25/ImgSharer/)                 |
| Docker? I barely know her! | [README](./MCTF25/Docker,%20I%20barely%20know%20her!/) |

### Web

| Challenge              | Writeup                                          |
|------------------------|--------------------------------------------------|
| Gatekeeper             | [README](./MCTF25/Gatekeeper/)                   |
| Homemade task system   | [README](./MCTF25/Homemade%20task%20system/)     |
| Homemade task system 2 | [README](./MCTF25/Homemade%20task%20system%202/) |
| Homemade task system 3 | [README](./MCTF25/Homemade%20task%20system%203/) |
| Parent Security        | [README](./MCTF25/Parent%20Security/)            |
| not!Windows registry   | [README](./MCTF25/not!Windows%20registry/)       |

## About this site

This site is generated with **GitHub Pages** from  
[`ANormalStick/CTF-Writeups`](https://github.com/ANormalStick/CTF-Writeups).

MCTF25 is the first event documented here. More CTFs (and more writeups) will be added over time.
