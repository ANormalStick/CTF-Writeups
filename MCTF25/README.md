# Mārtiņa-CTF 2025 (MCTF25)

<style>
.mctf-terminal {
  max-width: 900px;
  margin: 1.5rem 0 2rem 0;
  border-radius: 10px;
  border: 1px solid #1f2933;
  background: radial-gradient(circle at top left, #020617, #020617, #020617);
  font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;
  color: #e5e7eb;
  box-shadow: 0 16px 40px rgba(0, 0, 0, 0.6);
}

.mctf-terminal-header {
  padding: 0.4rem 0.75rem;
  border-bottom: 1px solid #111827;
  display: flex;
  align-items: center;
  justify-content: space-between;
  font-size: 0.75rem;
  background: linear-gradient(to right, #020617, #020617, #020617);
}

.mctf-terminal-header-left {
  letter-spacing: 0.06em;
  text-transform: uppercase;
  color: #6b7280;
}

.mctf-terminal-header-right {
  font-size: 0.7rem;
  color: #4b5563;
}

.mctf-terminal-body {
  padding: 1.3rem 1.6rem 1.2rem 1.6rem;
  font-size: 0.85rem;
  line-height: 1.5;
}

.mctf-terminal-body pre {
  margin: 0.3rem 0;
  white-space: pre-wrap;
}

.mctf-terminal-body code {
  background: transparent;
  padding: 0;
}

.mctf-prompt {
  color: #22c55e;
}

.mctf-path {
  color: #38bdf8;
}

.mctf-tag {
  color: #a855f7;
}

.mctf-comment {
  color: #6b7280;
}

.mctf-title {
  font-size: 0.95rem;
  margin-bottom: 0.25rem;
}

.mctf-sub {
  font-size: 0.8rem;
  color: #9ca3af;
}
</style>

<div class="mctf-terminal">
  <div class="mctf-terminal-header">
    <div class="mctf-terminal-header-left">SESSION MCTF25</div>
    <div class="mctf-terminal-header-right">ctf@anormalstick:/MCTF25</div>
  </div>
  <div class="mctf-terminal-body">
    <div class="mctf-title">Mārtiņa-CTF 2025 — writeups</div>
    <div class="mctf-sub">Blockchain, web, forensics, misc and more.</div>

    <pre><code><span class="mctf-comment"># enumerate solved challenges</span>
<span class="mctf-prompt">$</span> tree -L 1 .
<span class="mctf-tag">.</span>
├── Audio_Web
├── Blockchain
├── Blockchain_Forensics
├── Crypto
├── Forensics
├── Misc_Fun
├── Pwn_Docker
└── Web</code></pre>
  </div>
</div>

Writeups and notes for **Mārtiņa-CTF 2025**, organized by category.

---

## Navigation

- [Audio / Web](#audio--web)
- [Blockchain](#blockchain)
- [Blockchain / Forensics](#blockchain--forensics)
- [Crypto](#crypto)
- [Forensics](#forensics)
- [Misc / Fun](#misc--fun)
- [Pwn / Docker](#pwn--docker)
- [Web](#web)

---

<details open>
<summary><strong>Audio / Web</strong></summary>

- **Astral Pulses** → [writeup](./Astral%20Pulses/README.md)  
- **AI Translator** → [writeup](./AI%20Translator/README.md)

</details>

---

<details open>
<summary><strong>Blockchain</strong></summary>

- **Guess The Number** → [writeup](./%5BBlockchain%203%5D%20Guess%20The%20Number/README_Blockchain3_GuessTheNumber.md)  
- **Magical RPC Button** → [writeup](./%5BBlockchain%201%5D%20Magical%20RPC%20Button/README_Blockchain1_MagicalRPCButton.md)  
- **Unlimited Void** → [writeup](./Unlimited%20Void/Unlimited_Void_README.md)  
- **Where Did I Leave My Flag** → [writeup](./%5BBlockchain%202%5D%20Where%20Did%20I%20Leave%20My%20Flag/README_Blockchain2_WhereDidILeaveMyFlag.md)

</details>

---

<details open>
<summary><strong>Blockchain / Forensics</strong></summary>

- **Titanium Safe** → [writeup](./Titanium%20Safe/)  
- **Sacred Martins Sequence** → [writeup](./%5BBlockchain%204%5D%20Sacred%20Martins%20Sequence/readme.md)  
- **Sepolia Heist** → [writeup](./Sepolia%20Heist/readme.md)

</details>

---

<details open>
<summary><strong>Crypto</strong></summary>

- **Radical Security Animal** → [writeup](./%5BCryptography%204%5D%20Radical%20Security%20Animal/README_crypto4.md)

</details>

---

<details open>
<summary><strong>Forensics</strong></summary>

- **Rewritten History** → [writeup](./Rewritten%20History/README.md)

</details>

---

<details open>
<summary><strong>Misc / Fun</strong></summary>

- **A series of tubes** → [writeup](./A%20series%20of%20tubes/readme.md)  
- **Jokemartins** → [writeup](./Jokemartins/Readme.md)

</details>

---

<details open>
<summary><strong>Pwn / Docker</strong></summary>

- **ImgSharer** → [writeup](./ImgSharer/README.md)  
- **Docker, I barely know her!** → [writeup](./Docker,%20I%20barely%20know%20her!/README.md)

</details>

---

<details open>
<summary><strong>Web</strong></summary>

- **Gatekeeper** → [writeup](./Gatekeeper/Gatekeeper-README.md)  
- **Homemade task system** → [writeup](./Homemade%20task%20system/readme.md)  
- **Homemade task system 2** → [writeup](./Homemade%20task%20system%202/readme.md)  
- **Homemade task system 3** → [writeup](./Homemade%20task%20system%203/README.md)  
- **not!Windows registry** → [writeup](./not!Windows%20registry/README.md)  
- **Parent Security** → [writeup](./Parent%20Security/README.md)

</details>

---

## Notes

- All writeups are written post-CTF with access to challenge files.
- Focus is on:
  - clear exploit paths,
  - minimal but precise code,
  - enough context to re-solve without guessing.

If you spot an error or have a cleaner solve, open an issue or PR in  
[`ANormalStick/CTF-Writeups`](https://github.com/ANormalStick/CTF-Writeups).
