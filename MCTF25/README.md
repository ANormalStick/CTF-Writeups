# Mārtiņa-CTF 2025 (MCTF25)

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

html {
  scroll-behavior: smooth;
}

body {
  animation: mctf-fade-in 0.5s ease-out;
}

/* GitHub Pages / Jekyll wrappers */
.page-content, .wrapper, article, .post {
  background: transparent !important;
  max-width: 960px;
  margin: 0 auto;
  padding: 2.5rem 1.5rem 4rem;
}

/* Hero */

.mctf-hero {
  position: relative;
  border-radius: 0.9rem;
  border: 1px solid var(--border-subtle);
  background: radial-gradient(circle at top left, #020617 0, #020617 55%, #020617 100%);
  padding: 1.6rem 2rem 1.4rem;
  box-shadow: 0 22px 55px rgba(0, 0, 0, 0.65);
  margin-bottom: 2.2rem;
  overflow: hidden;
  transform: translateY(0);
  transition:
    transform 0.18s ease-out,
    box-shadow 0.18s ease-out,
    border-color 0.18s ease-out,
    background 0.25s ease-out;
}

.mctf-hero::before {
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
  animation: mctf-hero-orbit 18s linear infinite;
  pointer-events: none;
}

.mctf-hero:hover {
  transform: translateY(-3px);
  box-shadow: 0 26px 65px rgba(0, 0, 0, 0.85);
  border-color: rgba(56, 189, 248, 0.6);
  background: radial-gradient(circle at top left, #020617 0, #020617 40%, #020617 100%);
}

.mctf-hero:hover::before {
  opacity: 1;
}

.mctf-hero > * {
  position: relative;
  z-index: 1;
}

.mctf-hero-title {
  font-size: 1.55rem;
  letter-spacing: 0.04em;
  margin: 0 0 0.35rem;
}

.mctf-hero-subtitle {
  font-size: 0.92rem;
  color: var(--fg-muted);
  margin-bottom: 1rem;
}

.mctf-hero-meta {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
  font-size: 0.75rem;
  color: var(--fg-muted);
}

.mctf-pill {
  border-radius: 999px;
  padding: 0.25rem 0.7rem;
  border: 1px solid var(--border-subtle);
  background: linear-gradient(to right, rgba(15, 23, 42, 0.8), rgba(15, 23, 42, 0.35));
  backdrop-filter: blur(4px);
  white-space: nowrap;
  transition:
    border-color 0.18s ease-out,
    background 0.18s ease-out,
    transform 0.18s ease-out;
}

.mctf-pill:hover {
  border-color: rgba(56, 189, 248, 0.7);
  background: linear-gradient(to right, rgba(15, 23, 42, 0.95), rgba(15, 23, 42, 0.7));
  transform: translateY(-1px);
}

/* Headings */

.post h2, .page-content h2, article h2 {
  font-size: 1.05rem;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  color: var(--fg-muted);
  margin-top: 2.2rem;
  margin-bottom: 0.8rem;
}

.post h3, .page-content h3, article h3 {
  font-size: 0.98rem;
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
  animation: mctf-heading-glow 1.2s ease-out forwards;
}

/* Navigation chips */

.mctf-nav {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
  margin: 0.4rem 0 1.4rem;
}

.mctf-nav-chip {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: 0.25rem 0.8rem;
  border-radius: 999px;
  font-size: 0.78rem;
  text-transform: uppercase;
  letter-spacing: 0.09em;
  color: var(--fg-muted);
  border: 1px solid rgba(148, 163, 184, 0.5);
  background: radial-gradient(circle at top, rgba(15, 23, 42, 0.95), rgba(15, 23, 42, 0.8));
  box-shadow: 0 10px 28px rgba(0, 0, 0, 0.6);
  text-decoration: none;
  position: relative;
  transition:
    transform 0.14s ease-out,
    box-shadow 0.16s ease-out,
    border-color 0.16s ease-out,
    background 0.16s ease-out,
    color 0.16s ease-out;
}

/* kill underline animation on nav chips */
.mctf-nav-chip::after {
  display: none !important;
}

.mctf-nav-chip:hover {
  transform: translateY(-1px);
  border-color: rgba(56, 189, 248, 0.9);
  color: var(--accent);
  background: radial-gradient(circle at top, rgba(15, 23, 42, 1), rgba(15, 23, 42, 0.9));
  box-shadow: 0 14px 40px rgba(8, 47, 73, 0.7);
}

/* Challenge lists */

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

.challenge-actions {
  flex: 0 0 auto;
}

/* README buttons */

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

/* Notes card */

.mctf-notes {
  margin-top: 0.6rem;
  padding: 1rem 1.2rem;
  border-radius: 0.9rem;
  border: 1px solid rgba(148, 163, 184, 0.35);
  background: radial-gradient(circle at top left, rgba(15, 23, 42, 0.98), rgba(15, 23, 42, 0.94));
  box-shadow: 0 16px 40px rgba(0, 0, 0, 0.6);
  font-size: 0.9rem;
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

/* Code / pre */

code, pre {
  font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;
}

/* Small screens */

@media (max-width: 640px) {
  .page-content, .wrapper, article, .post {
    padding: 1.8rem 1.1rem 3rem;
  }

  .mctf-hero {
    padding: 1.4rem 1.3rem 1.2rem;
  }

  .challenge-row {
    align-items: flex-start;
  }
}

/* Animations */

@keyframes mctf-fade-in {
  from {
    opacity: 0;
    transform: translateY(4px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes mctf-hero-orbit {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

@keyframes mctf-heading-glow {
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

<section class="mctf-hero">
  <div class="mctf-hero-title">MCTF25 - Mārtiņa-CTF 2025</div>
  <div class="mctf-hero-subtitle">
    Personal writeups for Mārtiņa-CTF 2025. Mostly blockchain and web, with some forensics and misc.
  </div>

  <div class="mctf-hero-meta">
    <div class="mctf-pill">First CTF</div>
    <div class="mctf-pill">Per-challenge writeups</div>
    <div class="mctf-pill">Tech: Python · Solidity · Web</div>
  </div>
</section>

## Navigation

<div class="mctf-nav">
  <a class="mctf-nav-chip" href="#audio--web">Audio / Web</a>
  <a class="mctf-nav-chip" href="#blockchain">Blockchain</a>
  <a class="mctf-nav-chip" href="#blockchain--forensics">Blockchain / Forensics</a>
  <a class="mctf-nav-chip" href="#crypto">Crypto</a>
  <a class="mctf-nav-chip" href="#forensics">Forensics</a>
  <a class="mctf-nav-chip" href="#misc--fun">Misc / Fun</a>
  <a class="mctf-nav-chip" href="#pwn--docker">Pwn / Docker</a>
  <a class="mctf-nav-chip" href="#web">Web</a>
</div>

## Audio / Web

<div class="challenge-list">
  <div class="challenge-list-header">
    <span>Challenge</span>
    <span>Writeup</span>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Astral Pulses</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./Astral%20Pulses/">README</a>
    </div>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">AI Translator</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./AI%20Translator/">README</a>
    </div>
  </div>
</div>

## Blockchain

<div class="challenge-list">
  <div class="challenge-list-header">
    <span>Challenge</span>
    <span>Writeup</span>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Guess The Number</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./%5BBlockchain%203%5D%20Guess%20The%20Number/">README</a>
    </div>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Magical RPC Button</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./%5BBlockchain%201%5D%20Magical%20RPC%20Button/">README</a>
    </div>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Unlimited Void</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./Unlimited%20Void/">README</a>
    </div>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Where Did I Leave My Flag</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./%5BBlockchain%202%5D%20Where%20Did%20I%20Leave%20My%20Flag/">README</a>
    </div>
  </div>
</div>

## Blockchain / Forensics

<div class="challenge-list">
  <div class="challenge-list-header">
    <span>Challenge</span>
    <span>Writeup</span>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Titanium Safe</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./Titanium%20Safe/">README</a>
    </div>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Sacred Martins Sequence</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./%5BBlockchain%204%5D%20Sacred%20Martins%20Sequence/">README</a>
    </div>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Sepolia Heist</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./Sepolia%20Heist/">README</a>
    </div>
  </div>
</div>

## Crypto

<div class="challenge-list">
  <div class="challenge-list-header">
    <span>Challenge</span>
    <span>Writeup</span>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Radical Security Animal</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./%5BCryptography%204%5D%20Radical%20Security%20Animal/">README</a>
    </div>
  </div>
</div>

## Forensics

<div class="challenge-list">
  <div class="challenge-list-header">
    <span>Challenge</span>
    <span>Writeup</span>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Rewritten History</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./Rewritten%20History/">README</a>
    </div>
  </div>
</div>

## Misc / Fun

<div class="challenge-list">
  <div class="challenge-list-header">
    <span>Challenge</span>
    <span>Writeup</span>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">A series of tubes</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./A%20series%20of%20tubes/">README</a>
    </div>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Jokemartins</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./Jokemartins/">README</a>
    </div>
  </div>
</div>

## Pwn / Docker

<div class="challenge-list">
  <div class="challenge-list-header">
    <span>Challenge</span>
    <span>Writeup</span>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">ImgSharer</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./ImgSharer/">README</a>
    </div>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Docker, I barely know her!</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./Docker,%20I%20barely%20know%20her!/">README</a>
    </div>
  </div>
</div>

## Web

<div class="challenge-list">
  <div class="challenge-list-header">
    <span>Challenge</span>
    <span>Writeup</span>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Gatekeeper</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./Gatekeeper/">README</a>
    </div>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Homemade task system</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./Homemade%20task%20system/">README</a>
    </div>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Homemade task system 2</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./Homemade%20task%20system%202/">README</a>
    </div>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Homemade task system 3</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./Homemade%20task%20system%203/">README</a>
    </div>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">not!Windows registry</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./not!Windows%20registry/">README</a>
    </div>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Parent Security</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./Parent%20Security/">README</a>
    </div>
  </div>
</div>

## Notes

<div class="mctf-notes">
  <p>All writeups are post-CTF, with access to challenge files.</p>

  <p>I try to keep:</p>
  <ul>
    <li>clear exploit paths,</li>
    <li>minimal but precise code,</li>
    <li>enough context to re-solve without guessing.</li>
  </ul>

  <p>
    If you spot a mistake or have a cleaner solve, open an issue or PR in
    <a href="https://github.com/ANormalStick/CTF-Writeups"><code>ANormalStick/CTF-Writeups</code></a>.
  </p>
</div>
