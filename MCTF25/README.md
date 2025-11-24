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

.mctf-hero {
  border-radius: 0.9rem;
  border: 1px solid var(--border-subtle);
  background: radial-gradient(circle at top left, #020617 0, #020617 55%, #020617 100%);
  padding: 1.6rem 2rem 1.4rem;
  box-shadow: 0 22px 55px rgba(0, 0, 0, 0.65);
  margin-bottom: 2.2rem;
}

.mctf-hero-title {
  font-size: 1.45rem;
  letter-spacing: 0.03em;
  margin: 0 0 0.35rem;
}

.mctf-hero-subtitle {
  font-size: 0.9rem;
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
  background: linear-gradient(to right, rgba(15, 23, 42, 0.7), rgba(15, 23, 42, 0.25));
}

/* Headings */

.post h2, .page-content h2, article h2 {
  font-size: 1.05rem;
  letter-spacing: 0.04em;
  text-transform: uppercase;
  color: var(--fg-muted);
  margin-top: 2.2rem;
  margin-bottom: 0.8rem;
}

.post h3, .page-content h3, article h3 {
  font-size: 0.95rem;
  margin-top: 1.8rem;
  margin-bottom: 0.4rem;
}

/* Lists */

.mctf-cat-list {
  list-style: none;
  padding-left: 0;
  margin: 0.2rem 0 0.6rem;
}

.mctf-cat-list li {
  margin: 0.15rem 0;
}

/* Links */

a {
  color: var(--accent);
}

a:hover {
  text-decoration: none;
}

/* Code / pre */

code, pre {
  font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;
}
</style>

<section class="mctf-hero">
  <div class="mctf-hero-title">MCTF25 — Mārtiņa-CTF 2025</div>
  <div class="mctf-hero-subtitle">
    Personal writeups for Mārtiņa-CTF 2025. Mostly blockchain and web, with some forensics and misc.
  </div>

  <div class="mctf-hero-meta">
    <div class="mctf-pill">First CTF</div>
    <div class="mctf-pill">Writeups per challenge</div>
    <div class="mctf-pill">Tech: Python · Solidity · Web</div>
  </div>
</section>

## Navigation

- [Audio / Web](#audio--web)
- [Blockchain](#blockchain)
- [Blockchain / Forensics](#blockchain--forensics)
- [Crypto](#crypto)
- [Forensics](#forensics)
- [Misc / Fun](#misc--fun)
- [Pwn / Docker](#pwn--docker)
- [Web](#web)

## Audio / Web

- `Astral Pulses` → [writeup](./Astral%20Pulses/README.md)  
- `AI Translator` → [writeup](./AI%20Translator/README.md)

## Blockchain

- `Guess The Number` → [writeup](./%5BBlockchain%203%5D%20Guess%20The%20Number/README.md)  
- `Magical RPC Button` → [writeup](./%5BBlockchain%201%5D%20Magical%20RPC%20Button/README.md)  
- `Unlimited Void` → [writeup](./Unlimited%20Void/README.md)  
- `Where Did I Leave My Flag` → [writeup](./%5BBlockchain%202%5D%20Where%20Did%20I%20Leave%20My%20Flag/README.md)

## Blockchain / Forensics

- `Titanium Safe` → [writeup](./Titanium%20Safe/README.md)  
- `Sacred Martins Sequence` → [writeup](./%5BBlockchain%204%5D%20Sacred%20Martins%20Sequence/README.md)  
- `Sepolia Heist` → [writeup](./Sepolia%20Heist/README.md)

## Crypto

- `Radical Security Animal` → [writeup](./%5BCryptography%204%5D%20Radical%20Security%20Animal/README.md)

## Forensics

- `Rewritten History` → [writeup](./Rewritten%20History/README.md)

## Misc / Fun

- `A series of tubes` → [writeup](./A%20series%20of%20tubes/README.md)  
- `Jokemartins` → [writeup](./Jokemartins/README.md)

## Pwn / Docker

- `ImgSharer` → [writeup](./ImgSharer/README.md)  
- `Docker, I barely know her!` → [writeup](./Docker,%20I%20barely%20know%20her!/README.md)

## Web

- `Gatekeeper` → [writeup](./Gatekeeper/README.md)  
- `Homemade task system` → [writeup](./Homemade%20task%20system/README.md)  
- `Homemade task system 2` → [writeup](./Homemade%20task%20system%202/README.md)  
- `Homemade task system 3` → [writeup](./Homemade%20task%20system%203/README.md)  
- `not!Windows registry` → [writeup](./not!Windows%20registry/README.md)  
- `Parent Security` → [writeup](./Parent%20Security/README.md)

## Notes

All writeups are post-CTF, with access to challenge files.

I try to keep:

- clear exploit paths,  
- minimal but precise code,  
- enough context to re-solve without guessing.

If you spot a mistake or have a cleaner solve, open an issue or PR in  
[`ANormalStick/CTF-Writeups`](https://github.com/ANormalStick/CTF-Writeups).
