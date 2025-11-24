---
title: "Mārtiņa-CTF 2025 Writeups"
---

<style>
.terminal-root {
  max-width: 900px;
  margin: 2rem auto 2.5rem auto;
  border-radius: 10px;
  border: 1px solid #1f2933;
  background: radial-gradient(circle at top left, #111827, #020617);
  font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;
  color: #e5e7eb;
  box-shadow: 0 18px 45px rgba(0, 0, 0, 0.55);
}

.terminal-header {
  padding: 0.4rem 0.75rem;
  border-bottom: 1px solid #111827;
  display: flex;
  align-items: center;
  justify-content: space-between;
  font-size: 0.75rem;
  background: linear-gradient(to right, #020617, #020617, #020617);
}

.terminal-header-left {
  letter-spacing: 0.06em;
  text-transform: uppercase;
  color: #6b7280;
}

.terminal-header-right {
  font-size: 0.7rem;
  color: #4b5563;
}

.terminal-body {
  padding: 1.4rem 1.6rem 1.3rem 1.6rem;
  font-size: 0.85rem;
  line-height: 1.5;
}

.terminal-prompt {
  color: #22c55e;
}

.terminal-path {
  color: #38bdf8;
}

.terminal-accent {
  color: #a855f7;
}

.terminal-comment {
  color: #6b7280;
}

.terminal-title {
  font-size: 1rem;
  margin-bottom: 0.4rem;
  color: #e5e7eb;
}

.terminal-subtitle {
  font-size: 0.8rem;
  color: #9ca3af;
}

.terminal-body pre {
  margin: 0.3rem 0;
  white-space: pre-wrap;
}

.terminal-body code {
  background: transparent;
  padding: 0;
}

.terminal-divider {
  margin: 1rem 0 0.6rem 0;
  border-top: 1px solid #111827;
}

@media (prefers-color-scheme: light) {
  .terminal-root {
    background: radial-gradient(circle at top left, #111827, #020617);
  }
}
</style>

<div class="terminal-root">
  <div class="terminal-header">
    <div class="terminal-header-left">CTF://ANORMALSTICK</div>
    <div class="terminal-header-right">session mctf25-index</div>
  </div>
  <div class="terminal-body">
    <div class="terminal-title">ANormalStick — CTF Writeups</div>
    <div class="terminal-subtitle">Mārtiņa-CTF 2025 (MCTF25) · first CTF · full writeups</div>

    <div class="terminal-divider"></div>

    <pre><code><span class="terminal-comment"># repository bootstrap</span>
<span class="terminal-prompt">$</span> ls
MCTF25

<span class="terminal-comment"># inspect event index</span>
<span class="terminal-prompt">$</span> cat MCTF25/README.md
<span class="terminal-accent">MCTF25</span> — blockchain, web, forensics, misc and more
</code></pre>
  </div>
</div>

<div align="center">

[GitHub: `ANormalStick/CTF-Writeups`](https://github.com/ANormalStick/CTF-Writeups)

</div>

---

## Overview

This repo is a growing collection of CTF writeups.

Currently indexed:

| Year | CTF Name         | Alias  | Index                             |
|------|------------------|--------|-----------------------------------|
| 2025 | Mārtiņa-CTF 2025 | MCTF25 | [Browse MCTF25 writeups](./MCTF25/) |

---

## MCTF25 — Quick Navigation

Jump directly to a category:

- [Audio / Web](#audio--web)
- [Blockchain](#blockchain)
- [Blockchain / Forensics](#blockchain--forensics)
- [Crypto](#crypto)
- [Forensics](#forensics)
- [Misc / Fun](#misc--fun)
- [Pwn / Docker](#pwn--docker)
- [Web](#web)

---

## Category Index (MCTF25)

### Audio / Web

| Challenge      | Writeup |
|----------------|---------|
| Astral Pulses  | [README](./MCTF25/Astral%20Pulses/README.md) |
| AI Translator  | [README](./MCTF25/AI%20Translator/README.md) |

---

### Blockchain

| Challenge                 | Writeup |
|---------------------------|---------|
| Guess The Number          | [README](./MCTF25/%5BBlockchain%203%5D%20Guess%20The%20Number/README_Blockchain3_GuessTheNumber.md) |
| Magical RPC Button        | [README](./MCTF25/%5BBlockchain%201%5D%20Magical%20RPC%20Button/README_Blockchain1_MagicalRPCButton.md) |
| Unlimited Void            | [README](./MCTF25/Unlimited%20Void/Unlimited_Void_README.md) |
| Where Did I Leave My Flag | [README](./MCTF25/%5BBlockchain%202%5D%20Where%20Did%20I%20Leave%20My%20Flag/README_Blockchain2_WhereDidILeaveMyFlag.md) |

---

### Blockchain / Forensics

| Challenge               | Writeup |
|-------------------------|---------|
| Titanium Safe           | [index](./MCTF25/Titanium%20Safe/) |
| Sacred Martins Sequence | [README](./MCTF25/%5BBlockchain%204%5D%20Sacred%20Martins%20Sequence/readme.md) |
| Sepolia Heist           | [README](./MCTF25/Sepolia%20Heist/readme.md) |

---

### Crypto

| Challenge               | Writeup |
|-------------------------|---------|
| Radical Security Animal | [README](./MCTF25/%5BCryptography%204%5D%20Radical%20Security%20Animal/README_crypto4.md) |

---

### Forensics

| Challenge         | Writeup |
|-------------------|---------|
| Rewritten History | [README](./MCTF25/Rewritten%20History/README.md) |

---

### Misc / Fun

| Challenge         | Writeup |
|-------------------|---------|
| A series of tubes | [README](./MCTF25/A%20series%20of%20tubes/readme.md) |
| Jokemartins       | [README](./MCTF25/Jokemartins/Readme.md) |

---

### Pwn / Docker

| Challenge                  | Writeup |
|----------------------------|---------|
| ImgSharer                  | [README](./MCTF25/ImgSharer/README.md) |
| Docker? I barely know her! | [README](./MCTF25/Docker,%20I%20barely%20know%20her!/README.md) |

---

### Web

| Challenge               | Writeup |
|-------------------------|---------|
| Gatekeeper              | [README](./MCTF25/Gatekeeper/Gatekeeper-README.md) |
| Homemade task system    | [README](./MCTF25/Homemade%20task%20system/readme.md) |
| Homemade task system 2  | [README](./MCTF25/Homemade%20task%20system%202/readme.md) |
| Homemade task system 3  | [README](./MCTF25/Homemade%20task%20system%203/README.md) |
| Parent Security         | [README](./MCTF25/Parent%20Security/README.md) |
| not!Windows registry    | [README](./MCTF25/not!Windows%20registry/README.md) |

---

## About

This site is generated with **GitHub Pages** from  
[`ANormalStick/CTF-Writeups`](https://github.com/ANormalStick/CTF-Writeups).

MCTF25 is the first event; future CTFs will appear in the overview table once added.
