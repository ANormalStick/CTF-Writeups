---
title: "CTF Writeups"
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

/* gradient bar under section headings */
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

/* CTF card list */

.ctf-card-list {
  display: flex;
  flex-wrap: wrap;
  gap: 0.9rem;
  margin-bottom: 1.4rem;
  justify-content: center;
}

.ctf-card {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
  padding: 0.9rem 1.2rem;
  border-radius: 0.9rem;
  border: 1px solid rgba(148, 163, 184, 0.25);
  background: radial-gradient(circle at top left, rgba(15, 23, 42, 0.99), rgba(15, 23, 42, 0.94));
  box-shadow: 0 16px 40px rgba(0, 0, 0, 0.6);
  transition:
    transform 0.18s ease-out,
    box-shadow 0.18s ease-out,
    border-color 0.18s ease-out;
  flex: 1 1 280px;     /* allow cards to sit side by side */
  max-width: 460px;    /* stops them from stretching too wide */
}


.ctf-card:hover {
  transform: translateY(-2px);
  border-color: rgba(56, 189, 248, 0.7);
  box-shadow: 0 22px 60px rgba(8, 47, 73, 0.8);
}

.ctf-card-year {
  font-size: 0.8rem;
  text-transform: uppercase;
  letter-spacing: 0.15em;
  color: var(--fg-muted);
}

.ctf-card-main {
  flex: 1 1 auto;
}

.ctf-card-title {
  font-size: 0.98rem;
  font-weight: 500;
}

.ctf-card-meta {
  font-size: 0.8rem;
  color: var(--fg-muted);
}

/* Challenge lists (no tables!) */

.challenge-list {
  border-radius: 0.9rem;
  border: 1px solid rgba(148, 163, 184, 0.25);
  background: radial-gradient(circle at top left, rgba(15, 23, 42, 0.99), rgba(15, 23, 42, 0.94));
  box-shadow: 0 18px 45px rgba(0, 0, 0, 0.6);
  margin: 0.6rem 0 1.6rem;
  overflow: hidden;
}

.challenge-list-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 1rem;
  padding: 0.55rem 1rem;
  font-size: 0.75rem;
  text-transform: uppercase;
  letter-spacing: 0.09em;
  color: var(--fg-muted);
  border-bottom: 1px solid var(--border-subtle);
  background: linear-gradient(to right, rgba(15, 23, 42, 0.95), rgba(30, 64, 175, 0.85));
}

.challenge-list-header span:last-child {
  text-align: right;
}

.challenge-row {
  position: relative;
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 1rem;
  padding: 0.6rem 1rem;
  border-top: 1px solid rgba(15, 23, 42, 1);
  background: rgba(15, 23, 42, 0.98);
  overflow: hidden;
  transition:
    background-color 0.16s ease-out,
    transform 0.12s ease-out,
    box-shadow 0.16s ease-out;
}

.challenge-row:first-of-type {
  border-top: none;
}

.challenge-row::before {
  content: "";
  position: absolute;
  inset: 0;
  opacity: 0;
  background: linear-gradient(
    120deg,
    rgba(56, 189, 248, 0.12),
    transparent 40%,
    transparent 60%,
    rgba(168, 85, 247, 0.12)
  );
  pointer-events: none;
  transition: opacity 0.18s ease-out;
}

.challenge-row:hover {
  transform: translateY(-1px);
  box-shadow: 0 14px 40px rgba(8, 47, 73, 0.7);
}

.challenge-row:hover::before {
  opacity: 1;
}

.challenge-name {
  font-size: 0.9rem;
}

/* general tables (for other pages) – neutral dark */
table {
  border-collapse: collapse;
  width: 100%;
  font-size: 0.85rem;
  margin: 0.8rem 0 1.2rem;
  border-radius: 0.4rem;
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
  background-color: rgba(15, 23, 42, 0.8);
}

/* README-style buttons */

.challenge-actions {
  flex: 0 0 auto;
}

.challenge-btn {
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
  color: var(--accent);
  text-decoration: none;
  position: relative;
  transition:
    transform 0.14s ease-out,
    box-shadow 0.16s ease-out,
    border-color 0.16s ease-out,
    background 0.16s ease-out;
}

/* kill underline animation on buttons */
.challenge-btn::after {
  display: none !important;
}

.challenge-btn:hover {
  transform: translateY(-1px);
  border-color: rgba(56, 189, 248, 1);
  background: radial-gradient(circle at top, rgba(56, 189, 248, 0.35), rgba(15, 23, 42, 0.95));
  box-shadow:
    0 0 0 1px rgba(15, 23, 42, 1),
    0 0 24px rgba(56, 189, 248, 0.4);
}

/* Links – animated underline (normal links) */

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

  .ctf-card {
    flex-direction: column;
    align-items: flex-start;
  }

  .challenge-row {
    align-items: flex-start;
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
    Mārtiņa-CTF 2025 (MCTF25), HeroCTF v7, NexHunt CTF, and BSides Algiers 2025 - personal writeups, grouped by category.
  </div>

  <div class="ctf-hero-meta">
    <div class="ctf-pill">CTFs: MCTF25 · HeroCTF v7 · NexHunt CTF · BSides Algiers 2025</div>
    <div class="ctf-pill">Focus: blockchain · web · crypto · forensics · misc · system · OSINT · RE</div>
    <div class="ctf-pill">Format: per-challenge writeups</div>
  </div>

  <div class="ctf-link-row">
    Source:&nbsp;<a href="https://github.com/ANormalStick/CTF-Writeups">ANormalStick/CTF-Writeups</a>
  </div>
</section>

## CTFs

<div class="ctf-card-list">
  <article class="ctf-card">
    <div class="ctf-card-main">
      <div class="ctf-card-year">2025 · MCTF25</div>
      <div class="ctf-card-title">Mārtiņa-CTF 2025</div>
      <div class="ctf-card-meta">First event · full writeups</div>
    </div>
    <div class="ctf-card-actions">
      <a class="challenge-btn" href="./MCTF25/">Index</a>
    </div>
  </article>

  <article class="ctf-card">
    <div class="ctf-card-main">
      <div class="ctf-card-year">2025 · HeroCTF v7</div>
      <div class="ctf-card-title">HeroCTF v7</div>
      <div class="ctf-card-meta">Second event · full writeups</div>
    </div>
    <div class="ctf-card-actions">
      <a class="challenge-btn" href="./HeroCTF%20v7/">Index</a>
    </div>
  </article>

  <article class="ctf-card">
    <div class="ctf-card-main">
      <div class="ctf-card-year">2025 · NexHunt CTF</div>
      <div class="ctf-card-title">NexHunt CTF</div>
      <div class="ctf-card-meta">Third event · full writeups</div>
    </div>
    <div class="ctf-card-actions">
      <a class="challenge-btn" href="./NexHunt%20CTF/">Index</a>
    </div>
  </article>

  <article class="ctf-card">
    <div class="ctf-card-main">
      <div class="ctf-card-year">2025 · BSides Algiers</div>
      <div class="ctf-card-title">BSides Algiers 2025</div>
      <div class="ctf-card-meta">Fourth event · full writeups</div>
    </div>
    <div class="ctf-card-actions">
      <a class="challenge-btn" href="./BSides%20Algiers%202025/">Index</a>
    </div>
  </article>
</div>

## Overall Challenge Overview

Quick navigation by category:

- [Audio / Web](#audio--web)
- [Blockchain](#blockchain)
- [Blockchain / Forensics](#blockchain--forensics)
- [Crypto](#crypto)
- [Forensics](#forensics)
- [Misc / Fun](#misc--fun)
- [OSINT](#osint)
- [Pwn / Docker](#pwn--docker)
- [Prog](#prog)
- [Reverse Engineering](#reverse-engineering)
- [System](#system)
- [Web](#web)

<a id="audio--web"></a>
### Audio / Web

<div class="challenge-list">
  <div class="challenge-list-header">
    <span>Challenge</span>
    <span>Writeup</span>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Astral Pulses</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./MCTF25/Astral%20Pulses/">README</a>
    </div>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">AI Translator</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./MCTF25/AI%20Translator/">README</a>
    </div>
  </div>
</div>

<a id="blockchain"></a>
### Blockchain

<div class="challenge-list">
  <div class="challenge-list-header">
    <span>Challenge</span>
    <span>Writeup</span>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Guess The Number</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./MCTF25/%5BBlockchain%203%5D%20Guess%20The%20Number/">README</a>
    </div>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Magical RPC Button</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./MCTF25/%5BBlockchain%201%5D%20Magical%20RPC%20Button/">README</a>
    </div>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Unlimited Void</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./MCTF25/Unlimited%20Void/">README</a>
    </div>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Where Did I Leave My Flag</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./MCTF25/%5BBlockchain%202%5D%20Where%20Did%20I%20Leave%20My%20Flag/">README</a>
    </div>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">RustRoll</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./NexHunt%20CTF/RustRoll/">README</a>
    </div>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">BSides Only-invited party</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./BSides%20Algiers%202025/BSides%20Only-invited%20party/">README</a>
    </div>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">BSides Only-invited party REVENGE</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./BSides%20Algiers%202025/BSides%20Only-invited%20party%20REVENGE/">README</a>
    </div>
  </div>
</div>

<a id="blockchain--forensics"></a>
### Blockchain / Forensics

<div class="challenge-list">
  <div class="challenge-list-header">
    <span>Challenge</span>
    <span>Writeup</span>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Titanium Safe</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./MCTF25/Titanium%20Safe/">README</a>
    </div>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Sacred Martins Sequence</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./MCTF25/%5BBlockchain%204%5D%20Sacred%20Martins%20Sequence/">README</a>
    </div>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Sepolia Heist</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./MCTF25/Sepolia%20Heist/">README</a>
    </div>
  </div>
</div>

<a id="crypto"></a>
### Crypto

<div class="challenge-list">
  <div class="challenge-list-header">
    <span>Challenge</span>
    <span>Writeup</span>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Radical Security Animal</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./MCTF25/%5BCryptography%204%5D%20Radical%20Security%20Animal/">README</a>
    </div>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Andor</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./HeroCTF%20v7/Andor/">README</a>
    </div>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Genie</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./BSides%20Algiers%202025/Genie/">README</a>
    </div>
  </div>
</div>

<a id="forensics"></a>
### Forensics

<div class="challenge-list">
  <div class="challenge-list-header">
    <span>Challenge</span>
    <span>Writeup</span>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Rewritten History</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./MCTF25/Rewritten%20History/">README</a>
    </div>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Operation Pensieve Breach - 1</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./HeroCTF%20v7/Operation%20Pensieve%20Breach%20-%201/">README</a>
    </div>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Operation Pensieve Breach - 2</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./HeroCTF%20v7/Operation%20Pensieve%20Breach%20-%202/">README</a>
    </div>
  </div>
</div>

<a id="misc--fun"></a>
### Misc / Fun

<div class="challenge-list">
  <div class="challenge-list-header">
    <span>Challenge</span>
    <span>Writeup</span>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">A series of tubes</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./MCTF25/A%20series%20of%20tubes/">README</a>
    </div>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Jokemartins</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./MCTF25/Jokemartins/">README</a>
    </div>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">LSD#4</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./HeroCTF%20v7/LSD%234/">README</a>
    </div>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Neverland</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./HeroCTF%20v7/Neverland/">README</a>
    </div>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Bootloader</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./HeroCTF%20v7/Bootloader/">README</a>
    </div>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">the-scribe</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./NexHunt%20CTF/the-scribe/">README</a>
    </div>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Sōkyoku</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./NexHunt%20CTF/S%C5%8Dkyoku/">README</a>
    </div>
  </div>
</div>

<a id="osint"></a>
### OSINT

<div class="challenge-list">
  <div class="challenge-list-header">
    <span>Challenge</span>
    <span>Writeup</span>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">A Lone Love</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./NexHunt%20CTF/A%20Lone%20Love/">README</a>
    </div>
  </div>
</div>

<a id="pwn--docker"></a>
### Pwn / Docker

<div class="challenge-list">
  <div class="challenge-list-header">
    <span>Challenge</span>
    <span>Writeup</span>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">ImgSharer</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./MCTF25/ImgSharer/">README</a>
    </div>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Docker? I barely know her!</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./MCTF25/Docker,%20I%20barely%20know%20her!/">README</a>
    </div>
  </div>
</div>

<a id="prog"></a>
### Prog

<div class="challenge-list">
  <div class="challenge-list-header">
    <span>Challenge</span>
    <span>Writeup</span>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">PVE - Pirate Race #1</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./HeroCTF%20v7/PVE%20-%20Pirate%20Race%20%231/">README</a>
    </div>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">PVE - Pirate Race #2</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./HeroCTF%20v7/PVE%20-%20Pirate%20Race%20%232/">README</a>
    </div>
  </div>
</div>

<a id="reverse-engineering"></a>
### Reverse Engineering

<div class="challenge-list">
  <div class="challenge-list-header">
    <span>Challenge</span>
    <span>Writeup</span>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">HuntMe1</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./NexHunt%20CTF/HuntMe1/">README</a>
    </div>
  </div>
</div>

<a id="system"></a>
### System

<div class="challenge-list">
  <div class="challenge-list-header">
    <span>Challenge</span>
    <span>Writeup</span>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Middle Earth</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./HeroCTF%20v7/Middle%20Earth/">README</a>
    </div>
  </div>
</div>

<a id="web"></a>
### Web

<div class="challenge-list">
  <div class="challenge-list-header">
    <span>Challenge</span>
    <span>Writeup</span>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Gatekeeper</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./MCTF25/Gatekeeper/">README</a>
    </div>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Homemade task system</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./MCTF25/Homemade%20task%20system/">README</a>
    </div>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Homemade task system 2</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./MCTF25/Homemade%20task%20system%202/">README</a>
    </div>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Homemade task system 3</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./MCTF25/Homemade%20task%20system%203/">README</a>
    </div>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Parent Security</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./MCTF25/Parent%20Security/">README</a>
    </div>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">not!Windows registry</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./MCTF25/not!Windows%20registry/">README</a>
    </div>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Dz-Kitab</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./NexHunt%20CTF/Dz-Kitab/">README</a>
    </div>
  </div>
</div>

## About this site

This site is generated with **GitHub Pages** from  
[`ANormalStick/CTF-Writeups`](https://github.com/ANormalStick/CTF-Writeups).

MCTF25 was the first event documented here, followed by HeroCTF v7, NexHunt CTF, and BSides Algiers 2025.  
More CTFs (and more writeups) will be added over time.
