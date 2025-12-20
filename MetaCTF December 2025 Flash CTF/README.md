# MetaCTF December 2025 Flash CTF

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
  animation: meta-fade-in 0.5s ease-out;
}

/* GitHub Pages / Jekyll wrappers */
.page-content, .wrapper, article, .post {
  background: transparent !important;
  max-width: 960px;
  margin: 0 auto;
  padding: 2.5rem 1.5rem 4rem;
}

/* Hero */

.meta-hero {
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

.meta-hero::before {
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
  animation: meta-hero-orbit 18s linear infinite;
  pointer-events: none;
}

.meta-hero:hover {
  transform: translateY(-3px);
  box-shadow: 0 26px 65px rgba(0, 0, 0, 0.85);
  border-color: rgba(56, 189, 248, 0.6);
  background: radial-gradient(circle at top left, #020617 0, #020617 40%, #020617 100%);
}

.meta-hero:hover::before {
  opacity: 1;
}

.meta-hero > * {
  position: relative;
  z-index: 1;
}

.meta-hero-title {
  font-size: 1.55rem;
  letter-spacing: 0.04em;
  margin: 0 0 0.35rem;
}

.meta-hero-subtitle {
  font-size: 0.92rem;
  color: var(--fg-muted);
  margin-bottom: 1rem;
}

.meta-hero-meta {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
  font-size: 0.75rem;
  color: var(--fg-muted);
}

.meta-pill {
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

.meta-pill:hover {
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
  animation: meta-heading-glow 1.2s ease-out forwards;
}

/* Navigation chips */

.meta-nav {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
  margin: 0.4rem 0 1.4rem;
}

.meta-nav-chip {
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
.meta-nav-chip::after {
  display: none !important;
}

.meta-nav-chip:hover {
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

.meta-notes {
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

  .meta-hero {
    padding: 1.4rem 1.3rem 1.2rem;
  }

  .challenge-row {
    align-items: flex-start;
  }
}

/* Animations */

@keyframes meta-fade-in {
  from {
    opacity: 0;
    transform: translateY(4px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes meta-hero-orbit {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

@keyframes meta-heading-glow {
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

<section class="meta-hero">
  <div class="meta-hero-title">MetaCTF December 2025 Flash CTF</div>
  <div class="meta-hero-subtitle">
    Personal writeups for MetaCTF December 2025 Flash CTF. Web, forensics, and misc challenges.
  </div>

  <div class="meta-hero-meta">
    <div class="meta-pill">🥉 3rd Place</div>
    <div class="meta-pill">Solo</div>
    <div class="meta-pill">Per-challenge writeups</div>
    <div class="meta-pill">Tech: Python · Perl · x86-64</div>
  </div>
</section>

## Navigation

<div class="meta-nav">
  <a class="meta-nav-chip" href="#forensics">Forensics</a>
  <a class="meta-nav-chip" href="#misc">Misc</a>
  <a class="meta-nav-chip" href="#web">Web</a>
</div>

## Forensics

<div class="challenge-list">
  <div class="challenge-list-header">
    <span>Challenge</span>
    <span>Writeup</span>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Making The Naughty List</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./MakingTheNaughtyList/">README</a>
    </div>
  </div>
</div>

## Misc

<div class="challenge-list">
  <div class="challenge-list-header">
    <span>Challenge</span>
    <span>Writeup</span>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Perl Poetry</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./PerlPoetry/">README</a>
    </div>
  </div>
  <div class="challenge-row">
    <div class="challenge-name">Santa's Christmas Calculator</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./SantasChristmasCalculator/">README</a>
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
    <div class="challenge-name">Gigs</div>
    <div class="challenge-actions">
      <a class="challenge-btn" href="./Gigs/">README</a>
    </div>
  </div>
</div>

## Notes

<div class="meta-notes">
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
