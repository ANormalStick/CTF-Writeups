# Mārtiņa-CTF 2025 (MCTF25)

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

/* GitHub / Jekyll wrappers (best-effort overrides) */
.page-content, .wrapper, article {
  background: transparent !important;
}

/* Layout */

.mctf-page {
  max-width: 960px;
  margin: 0 auto;
  padding: 2.2rem 1.5rem 4rem;
}

.mctf-hero {
  border-radius: 0.9rem;
  border: 1px solid var(--border-subtle);
  background: radial-gradient(circle at top left, #020617 0, #020617 55%, #020617 100%);
  padding: 1.6rem 2rem 1.4rem;
  box-shadow: 0 22px 55px rgba(0, 0, 0, 0.65);
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

.mctf-section {
  margin-top: 2.2rem;
}

.mctf-section h2 {
  font-size: 1.05rem;
  letter-spacing: 0.04em;
  text-transform: uppercase;
  color: var(--fg-muted);
  margin-bottom: 0.8rem;
}

.mctf-section h3 {
  font-size: 0.95rem;
  margin-bottom: 0.35rem;
}

.mctf-small {
  font-size: 0.8rem;
  color: var(--fg-muted);
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

<div class="mctf-page">

  <section class="mctf-hero">
    <div class="mctf-hero-title">MCTF25 — Mārtiņa-CTF 2025</div>
    <div class="mctf-hero-subtitle">
      Personal writeups for Mārtiņa-CTF 2025. Mostly focused on blockchain and web, with some forensics and misc.
    </div>

    <div class="mctf-hero-meta">
      <div class="mctf-pill">First CTF</div>
      <div class="mctf-pill">Writeups per challenge</div>
      <div class="mctf-pill">Tech stack: Python · Solidity · Web</div>
    </div>
  </section>

  <section class="mctf-section">
    <h2>Navigation</h2>

- [Audio / Web](#audio--web)
- [Blockchain](#blockchain)
- [Blockchain / Forensics](#blockchain--forensics)
- [Crypto](#crypto)
- [Forensics](#forensics)
- [Misc / Fun](#misc--fun)
- [Pwn / Docker](#pwn--docker)
- [Web](#web)

  </section>

  <section class="mctf-section" id="audio--web">
    <h2>Audio / Web</h2>

<ul class="mctf-cat-list">
  <li><strong>Astral Pulses</strong> → <a href="./Astral%20Pulses/README.md">writeup</a></li>
  <li><strong>AI Translator</strong> → <a href="./AI%20Translator/README.md">writeup</a></li>
</ul>
  </section>

  <section class="mctf-section" id="blockchain">
    <h2>Blockchain</h2>

<ul class="mctf-cat-list">
  <li><strong>Guess The Number</strong> → <a href="./%5BBlockchain%203%5D%20Guess%20The%20Number/README_Blockchain3_GuessTheNumber.md">writeup</a></li>
  <li><strong>Magical RPC Button</strong> → <a href="./%5BBlockchain%201%5D%20Magical%20RPC%20Button/README_Blockchain1_MagicalRPCButton.md">writeup</a></li>
  <li><strong>Unlimited Void</strong> → <a href="./Unlimited%20Void/Unlimited_Void_README.md">writeup</a></li>
  <li><strong>Where Did I Leave My Flag</strong> → <a href="./%5BBlockchain%202%5D%20Where%20Did%20I%20Leave%20My%20Flag/README_Blockchain2_WhereDidILeaveMyFlag.md">writeup</a></li>
</ul>
  </section>

  <section class="mctf-section" id="blockchain--forensics">
    <h2>Blockchain / Forensics</h2>

<ul class="mctf-cat-list">
  <li><strong>Titanium Safe</strong> → <a href="./Titanium%20Safe/">writeup</a></li>
  <li><strong>Sacred Martins Sequence</strong> → <a href="./%5BBlockchain%204%5D%20Sacred%20Martins%20Sequence/readme.md">writeup</a></li>
  <li><strong>Sepolia Heist</strong> → <a href="./Sepolia%20Heist/readme.md">writeup</a></li>
</ul>
  </section>

  <section class="mctf-section" id="crypto">
    <h2>Crypto</h2>

<ul class="mctf-cat-list">
  <li><strong>Radical Security Animal</strong> → <a href="./%5BCryptography%204%5D%20Radical%20Security%20Animal/README_crypto4.md">writeup</a></li>
</ul>
  </section>

  <section class="mctf-section" id="forensics">
    <h2>Forensics</h2>

<ul class="mctf-cat-list">
  <li><strong>Rewritten History</strong> → <a href="./Rewritten%20History/README.md">writeup</a></li>
</ul>
  </section>

  <section class="mctf-section" id="misc--fun">
    <h2>Misc / Fun</h2>

<ul class="mctf-cat-list">
  <li><strong>A series of tubes</strong> → <a href="./A%20series%20of%20tubes/readme.md">writeup</a></li>
  <li><strong>Jokemartins</strong> → <a href="./Jokemartins/Readme.md">writeup</a></li>
</ul>
  </section>

  <section class="mctf-section" id="pwn--docker">
    <h2>Pwn / Docker</h2>

<ul class="mctf-cat-list">
  <li><strong>ImgSharer</strong> → <a href="./ImgSharer/README.md">writeup</a></li>
  <li><strong>Docker, I barely know her!</strong> → <a href="./Docker,%20I%20barely%20know%20her!/README.md">writeup</a></li>
</ul>
  </section>

  <section class="mctf-section" id="web">
    <h2>Web</h2>

<ul class="mctf-cat-list">
  <li><strong>Gatekeeper</strong> → <a href="./Gatekeeper/Gatekeeper-README.md">writeup</a></li>
  <li><strong>Homemade task system</strong> → <a href="./Homemade%20task%20system/readme.md">writeup</a></li>
  <li><strong>Homemade task system 2</strong> → <a href="./Homemade%20task%20system%202/readme.md">writeup</a></li>
  <li><strong>Homemade task system 3</strong> → <a href="./Homemade%20task%20system%203/README.md">writeup</a></li>
  <li><strong>not!Windows registry</strong> → <a href="./not!Windows%20registry/README.md">writeup</a></li>
  <li><strong>Parent Security</strong> → <a href="./Parent%20Security/README.md">writeup</a></li>
</ul>
  </section>

  <section class="mctf-section">
    <h2>Notes</h2>

<div class="mctf-small">
All writeups are done post-CTF with access to challenge files.  
I try to keep:
<ul>
  <li>clear exploit paths,</li>
  <li>minimal but precise code,</li>
  <li>enough context to re-solve without guesswork.</li>
</ul>

If you see a mistake or have a cleaner solve, open an issue or PR in  
<a href="https://github.com/ANormalStick/CTF-Writeups">ANormalStick/CTF-Writeups</a>.
</div>

  </section>

</div>
