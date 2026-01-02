---
title: "ANormalStick"
---

<style>
:root {
  color-scheme: dark;
  --bg: #020617;
  --bg-card: rgba(15, 23, 42, 0.95);
  --border-subtle: #1f2937;
  --fg: #e5e7eb;
  --fg-muted: #9ca3af;
  --accent: #38bdf8;
  --accent-secondary: #a855f7;
  --accent-tertiary: #22c55e;
}

/* Base page styling */
html, body {
  margin: 0;
  padding: 0;
  background: radial-gradient(ellipse at top, #0b1120 0%, #020617 50%, #000 100%);
  color: var(--fg);
  font-family: system-ui, -apple-system, BlinkMacSystemFont, "SF Pro Text", "Segoe UI", sans-serif;
  line-height: 1.6;
}

body {
  animation: page-fade-in 0.6s ease-out;
}

/* GitHub Pages / Jekyll wrappers */
.page-content, .wrapper, article, .post {
  background: transparent !important;
  max-width: 1000px;
  margin: 0 auto;
  padding: 2.5rem 1.5rem 4rem;
}

html {
  scroll-behavior: smooth;
}

/* ===== MAIN HERO ===== */

.main-hero {
  position: relative;
  border-radius: 1rem;
  border: 1px solid var(--border-subtle);
  background: linear-gradient(135deg, rgba(15, 23, 42, 0.98) 0%, rgba(30, 41, 59, 0.95) 100%);
  padding: 2.5rem 2.5rem 2rem;
  box-shadow: 0 25px 60px rgba(0, 0, 0, 0.7);
  margin-bottom: 2.5rem;
  overflow: hidden;
  text-align: center;
}

.main-hero::before {
  content: "";
  position: absolute;
  inset: -50%;
  background: conic-gradient(
    from 180deg,
    rgba(56, 189, 248, 0.15),
    rgba(168, 85, 247, 0.1),
    rgba(34, 197, 94, 0.12),
    rgba(56, 189, 248, 0.15)
  );
  opacity: 0.5;
  animation: hero-rotate 25s linear infinite;
  pointer-events: none;
}

.main-hero > * {
  position: relative;
  z-index: 1;
}

.main-hero-name {
  font-size: 2.4rem;
  font-weight: 700;
  letter-spacing: -0.02em;
  margin: 0 0 0.5rem;
  background: linear-gradient(135deg, var(--fg) 0%, var(--accent) 50%, var(--accent-secondary) 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.main-hero-tagline {
  font-size: 1.1rem;
  color: var(--fg-muted);
  margin-bottom: 1.5rem;
  max-width: 600px;
  margin-left: auto;
  margin-right: auto;
}

.main-hero-links {
  display: flex;
  flex-wrap: wrap;
  gap: 0.6rem;
  justify-content: center;
  margin-bottom: 1.5rem;
}

.hero-link {
  display: inline-flex;
  align-items: center;
  gap: 0.4rem;
  padding: 0.5rem 1rem;
  border-radius: 999px;
  font-size: 0.85rem;
  border: 1px solid var(--border-subtle);
  background: rgba(15, 23, 42, 0.8);
  color: var(--fg);
  text-decoration: none;
  transition: all 0.18s ease-out;
}

.hero-link::after { display: none !important; }

.hero-link:hover {
  border-color: var(--accent);
  background: rgba(56, 189, 248, 0.15);
  transform: translateY(-2px);
}

.main-hero-stats {
  display: flex;
  flex-wrap: wrap;
  gap: 1.5rem;
  justify-content: center;
  padding-top: 1.5rem;
  border-top: 1px solid var(--border-subtle);
}

.stat-item {
  text-align: center;
}

.stat-number {
  font-size: 1.6rem;
  font-weight: 600;
  color: var(--accent);
}

.stat-label {
  font-size: 0.75rem;
  color: var(--fg-muted);
  text-transform: uppercase;
  letter-spacing: 0.1em;
}

/* ===== NAVIGATION TABS ===== */

.nav-tabs {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
  justify-content: center;
  margin-bottom: 2.5rem;
}

.nav-tab {
  display: inline-flex;
  align-items: center;
  gap: 0.4rem;
  padding: 0.6rem 1.2rem;
  border-radius: 0.6rem;
  font-size: 0.88rem;
  font-weight: 500;
  border: 1px solid var(--border-subtle);
  background: var(--bg-card);
  color: var(--fg-muted);
  text-decoration: none;
  transition: all 0.18s ease-out;
}

.nav-tab::after { display: none !important; }

.nav-tab:hover {
  color: var(--fg);
  border-color: var(--accent);
  background: rgba(56, 189, 248, 0.1);
  transform: translateY(-2px);
  box-shadow: 0 8px 25px rgba(56, 189, 248, 0.15);
}

/* ===== SECTION STYLING ===== */

.section-header {
  display: flex;
  align-items: center;
  gap: 0.8rem;
  margin-bottom: 1.2rem;
}

.section-icon {
  font-size: 1.4rem;
}

.section-title {
  font-size: 1.3rem;
  font-weight: 600;
  margin: 0;
  background: linear-gradient(90deg, var(--fg), var(--accent));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.section-subtitle {
  font-size: 0.9rem;
  color: var(--fg-muted);
  margin-bottom: 1.5rem;
}

/* ===== PROJECT CARDS (for game & challenges) ===== */

.project-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 1.2rem;
  margin-bottom: 2.5rem;
}

.project-card {
  position: relative;
  border-radius: 1rem;
  border: 1px solid var(--border-subtle);
  background: var(--bg-card);
  padding: 1.5rem;
  box-shadow: 0 15px 40px rgba(0, 0, 0, 0.5);
  overflow: hidden;
  transition: all 0.2s ease-out;
}

.project-card::before {
  content: "";
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 3px;
  background: linear-gradient(90deg, var(--accent), var(--accent-secondary), var(--accent-tertiary));
  opacity: 0;
  transition: opacity 0.2s ease-out;
}

.project-card:hover {
  transform: translateY(-4px);
  border-color: rgba(56, 189, 248, 0.5);
  box-shadow: 0 20px 50px rgba(8, 47, 73, 0.6);
}

.project-card:hover::before {
  opacity: 1;
}

.project-badge {
  display: inline-block;
  padding: 0.2rem 0.6rem;
  border-radius: 999px;
  font-size: 0.7rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  margin-bottom: 0.8rem;
}

.badge-game {
  background: linear-gradient(135deg, rgba(168, 85, 247, 0.3), rgba(168, 85, 247, 0.1));
  color: #c084fc;
  border: 1px solid rgba(168, 85, 247, 0.4);
}

.badge-ctf {
  background: linear-gradient(135deg, rgba(34, 197, 94, 0.3), rgba(34, 197, 94, 0.1));
  color: #4ade80;
  border: 1px solid rgba(34, 197, 94, 0.4);
}

.badge-web {
  background: linear-gradient(135deg, rgba(56, 189, 248, 0.3), rgba(56, 189, 248, 0.1));
  color: #7dd3fc;
  border: 1px solid rgba(56, 189, 248, 0.4);
}

.badge-misc {
  background: linear-gradient(135deg, rgba(251, 191, 36, 0.3), rgba(251, 191, 36, 0.1));
  color: #fcd34d;
  border: 1px solid rgba(251, 191, 36, 0.4);
}

.project-title {
  font-size: 1.15rem;
  font-weight: 600;
  margin: 0 0 0.4rem;
}

.project-desc {
  font-size: 0.88rem;
  color: var(--fg-muted);
  margin-bottom: 1rem;
  line-height: 1.5;
}

.project-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 0.4rem;
  margin-bottom: 1rem;
}

.project-tag {
  padding: 0.15rem 0.5rem;
  border-radius: 0.3rem;
  font-size: 0.72rem;
  background: rgba(56, 189, 248, 0.1);
  color: var(--accent);
  border: 1px solid rgba(56, 189, 248, 0.2);
}

.project-actions {
  display: flex;
  gap: 0.5rem;
}

.project-btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: 0.4rem 1rem;
  border-radius: 0.5rem;
  font-size: 0.8rem;
  font-weight: 500;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  border: 1px solid rgba(56, 189, 248, 0.6);
  background: linear-gradient(135deg, rgba(56, 189, 248, 0.2), rgba(56, 189, 248, 0.05));
  color: var(--accent);
  text-decoration: none;
  transition: all 0.15s ease-out;
}

.project-btn::after { display: none !important; }

.project-btn:hover {
  background: linear-gradient(135deg, rgba(56, 189, 248, 0.35), rgba(56, 189, 248, 0.15));
  border-color: var(--accent);
  transform: translateY(-1px);
  box-shadow: 0 4px 15px rgba(56, 189, 248, 0.25);
}

/* ===== CTF CARDS ===== */

.ctf-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 1rem;
  margin-bottom: 2.5rem;
}

.ctf-card {
  position: relative;
  border-radius: 0.9rem;
  border: 1px solid var(--border-subtle);
  background: var(--bg-card);
  padding: 1.2rem 1.4rem;
  box-shadow: 0 12px 35px rgba(0, 0, 0, 0.5);
  transition: all 0.18s ease-out;
  overflow: hidden;
}

.ctf-card:hover {
  transform: translateY(-3px);
  border-color: rgba(56, 189, 248, 0.5);
  box-shadow: 0 18px 45px rgba(8, 47, 73, 0.6);
}

.ctf-card-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 0.6rem;
}

.ctf-card-year {
  font-size: 0.72rem;
  text-transform: uppercase;
  letter-spacing: 0.12em;
  color: var(--fg-muted);
}

.ctf-placement {
  padding: 0.15rem 0.5rem;
  border-radius: 999px;
  font-size: 0.7rem;
  font-weight: 600;
}

.placement-gold {
  background: linear-gradient(135deg, rgba(251, 191, 36, 0.3), rgba(251, 191, 36, 0.1));
  color: #fcd34d;
}

.placement-silver {
  background: linear-gradient(135deg, rgba(148, 163, 184, 0.3), rgba(148, 163, 184, 0.1));
  color: #cbd5e1;
}

.placement-bronze {
  background: linear-gradient(135deg, rgba(217, 119, 6, 0.3), rgba(217, 119, 6, 0.1));
  color: #fbbf24;
}

.placement-top10 {
  background: linear-gradient(135deg, rgba(34, 197, 94, 0.3), rgba(34, 197, 94, 0.1));
  color: #4ade80;
}

.ctf-card-title {
  font-size: 1.05rem;
  font-weight: 600;
  margin: 0 0 0.3rem;
}

.ctf-card-team {
  font-size: 0.82rem;
  color: var(--fg-muted);
  margin-bottom: 0.8rem;
}

.ctf-card-btn {
  display: inline-flex;
  align-items: center;
  padding: 0.3rem 0.8rem;
  border-radius: 0.4rem;
  font-size: 0.75rem;
  text-transform: uppercase;
  letter-spacing: 0.06em;
  border: 1px solid rgba(56, 189, 248, 0.5);
  background: rgba(56, 189, 248, 0.1);
  color: var(--accent);
  text-decoration: none;
  transition: all 0.15s ease-out;
}

.ctf-card-btn::after { display: none !important; }

.ctf-card-btn:hover {
  background: rgba(56, 189, 248, 0.2);
  border-color: var(--accent);
}

/* ===== CHALLENGE LIST ===== */

.challenge-section {
  margin-bottom: 2.5rem;
}

.challenge-list {
  border-radius: 0.9rem;
  border: 1px solid var(--border-subtle);
  background: var(--bg-card);
  box-shadow: 0 15px 40px rgba(0, 0, 0, 0.5);
  overflow: hidden;
}

.challenge-list-header {
  display: flex;
  justify-content: space-between;
  padding: 0.6rem 1.2rem;
  font-size: 0.75rem;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  color: var(--fg-muted);
  background: linear-gradient(90deg, rgba(15, 23, 42, 0.98), rgba(30, 64, 175, 0.4));
  border-bottom: 1px solid var(--border-subtle);
}

.challenge-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.65rem 1.2rem;
  border-top: 1px solid rgba(30, 41, 59, 0.5);
  background: rgba(15, 23, 42, 0.6);
  transition: all 0.15s ease-out;
}

.challenge-row:first-of-type {
  border-top: none;
}

.challenge-row:hover {
  background: rgba(56, 189, 248, 0.08);
}

.challenge-name {
  font-size: 0.9rem;
}

.challenge-btn {
  padding: 0.25rem 0.7rem;
  border-radius: 0.4rem;
  font-size: 0.72rem;
  text-transform: uppercase;
  letter-spacing: 0.06em;
  border: 1px solid rgba(56, 189, 248, 0.5);
  background: rgba(56, 189, 248, 0.1);
  color: var(--accent);
  text-decoration: none;
  transition: all 0.15s ease-out;
}

.challenge-btn::after { display: none !important; }

.challenge-btn:hover {
  background: rgba(56, 189, 248, 0.25);
  border-color: var(--accent);
}

/* ===== CATEGORY NAV ===== */

.category-nav {
  display: flex;
  flex-wrap: wrap;
  gap: 0.4rem;
  margin-bottom: 1.5rem;
}

.category-chip {
  padding: 0.3rem 0.7rem;
  border-radius: 999px;
  font-size: 0.75rem;
  border: 1px solid var(--border-subtle);
  background: rgba(15, 23, 42, 0.8);
  color: var(--fg-muted);
  text-decoration: none;
  transition: all 0.15s ease-out;
}

.category-chip::after { display: none !important; }

.category-chip:hover {
  border-color: var(--accent);
  color: var(--accent);
  background: rgba(56, 189, 248, 0.1);
}

/* ===== FOOTER ===== */

.site-footer {
  margin-top: 3rem;
  padding-top: 2rem;
  border-top: 1px solid var(--border-subtle);
  text-align: center;
}

.footer-text {
  font-size: 0.85rem;
  color: var(--fg-muted);
  margin-bottom: 0.5rem;
}

.footer-links {
  display: flex;
  gap: 1rem;
  justify-content: center;
}

.footer-link {
  font-size: 0.85rem;
  color: var(--accent);
}

/* ===== LINKS ===== */

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

/* ===== RESPONSIVE ===== */

@media (max-width: 640px) {
  .page-content, .wrapper, article, .post {
    padding: 1.5rem 1rem 3rem;
  }

  .main-hero {
    padding: 1.8rem 1.2rem 1.5rem;
  }

  .main-hero-name {
    font-size: 1.8rem;
  }

  .project-grid, .ctf-grid {
    grid-template-columns: 1fr;
  }

  .main-hero-stats {
    gap: 1rem;
  }
}

/* ===== ANIMATIONS ===== */

@keyframes page-fade-in {
  from {
    opacity: 0;
    transform: translateY(8px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes hero-rotate {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}
</style>

<!-- ===== MAIN HERO ===== -->

<section class="main-hero">
  <h1 class="main-hero-name">ANormalStick</h1>
  <p class="main-hero-tagline">
    CTF player, challenge creator, and game developer. Building puzzles, breaking systems, and occasionally making platformers that will test your patience.
  </p>

  <div class="main-hero-links">
    <a class="hero-link" href="https://github.com/ANormalStick">📦 GitHub</a>
    <a class="hero-link" href="#projects">🎮 Projects</a>
    <a class="hero-link" href="#ctf-writeups">🚩 CTF Writeups</a>
    <a class="hero-link" href="#my-challenges">🧩 My Challenges</a>
  </div>

  <div class="main-hero-stats">
    <div class="stat-item">
      <div class="stat-number">5</div>
      <div class="stat-label">CTFs Documented</div>
    </div>
    <div class="stat-item">
      <div class="stat-number">🥉 3rd</div>
      <div class="stat-label">Best Placement</div>
    </div>
    <div class="stat-item">
      <div class="stat-number">2</div>
      <div class="stat-label">Challenges Created</div>
    </div>
    <div class="stat-item">
      <div class="stat-number">1</div>
      <div class="stat-label">Game Released</div>
    </div>
  </div>
</section>

<!-- ===== NAVIGATION ===== -->

<nav class="nav-tabs">
  <a class="nav-tab" href="#projects">🎮 Projects</a>
  <a class="nav-tab" href="#my-challenges">🧩 My Challenges</a>
  <a class="nav-tab" href="#ctf-writeups">🚩 CTF Writeups</a>
  <a class="nav-tab" href="#all-challenges">📋 All Challenges</a>
</nav>

<!-- ===== PROJECTS SECTION ===== -->

<section id="projects">
  <div class="section-header">
    <span class="section-icon">🎮</span>
    <h2 class="section-title">Projects</h2>
  </div>
  <p class="section-subtitle">Games and tools I've built</p>

  <div class="project-grid">
    <article class="project-card">
      <span class="project-badge badge-game">Game</span>
      <h3 class="project-title">Shattered Towers</h3>
      <p class="project-desc">
        A 2D platformer puzzle/rage game that will test your skills and patience. Navigate through challenging towers filled with traps, puzzles, and precision-based obstacles.
      </p>
      <div class="project-tags">
        <span class="project-tag">2D Platformer</span>
        <span class="project-tag">Puzzle</span>
        <span class="project-tag">Rage Game</span>
      </div>
      <div class="project-actions">
        <a class="project-btn" href="./ShatteredTowers/">Download</a>
      </div>
    </article>
  </div>
</section>

<!-- ===== MY CHALLENGES SECTION ===== -->

<section id="my-challenges">
  <div class="section-header">
    <span class="section-icon">🧩</span>
    <h2 class="section-title">CTF Challenges I Created</h2>
  </div>
  <p class="section-subtitle">Original challenges designed to test various skills</p>

  <div class="project-grid">
    <article class="project-card">
      <span class="project-badge badge-misc">Misc / Steganography</span>
      <h3 class="project-title">Music Box v2</h3>
      <p class="project-desc">
        A multi-layered puzzle involving hidden hex blobs, XOR encoding, spectrogram analysis, Morse code, corrupted images, and AES decryption. Find the real present among the fake ones.
      </p>
      <div class="project-tags">
        <span class="project-tag">Steganography</span>
        <span class="project-tag">Audio Analysis</span>
        <span class="project-tag">AES</span>
        <span class="project-tag">XOR</span>
      </div>
      <div class="project-actions">
        <a class="project-btn" href="./ChallangesIMade/MusicBoxV2/">Writeup</a>
      </div>
    </article>

    <article class="project-card">
      <span class="project-badge badge-web">Web / GraphQL</span>
      <h3 class="project-title">Naughty or Nice</h3>
      <p class="project-desc">
        Exploit GraphQL vulnerabilities to access Santa's protected data. Features introspection discovery, authorization bypass, and batch query attacks.
      </p>
      <div class="project-tags">
        <span class="project-tag">GraphQL</span>
        <span class="project-tag">Authorization Bypass</span>
        <span class="project-tag">Web Exploitation</span>
      </div>
      <div class="project-actions">
        <a class="project-btn" href="./ChallangesIMade/NaughtyOrNice/">Writeup</a>
      </div>
    </article>
  </div>
</section>

<!-- ===== CTF WRITEUPS SECTION ===== -->

<section id="ctf-writeups">
  <div class="section-header">
    <span class="section-icon">🚩</span>
    <h2 class="section-title">CTF Writeups</h2>
  </div>
  <p class="section-subtitle">Competitions I've participated in with detailed writeups</p>

  <div class="ctf-grid">
    <article class="ctf-card">
      <div class="ctf-card-header">
        <span class="ctf-card-year">2025 · MetaCTF</span>
        <span class="ctf-placement placement-bronze">🥉 3rd</span>
      </div>
      <h3 class="ctf-card-title">MetaCTF December 2025 Flash CTF</h3>
      <p class="ctf-card-team">Solo</p>
      <a class="ctf-card-btn" href="./MetaCTF%20December%202025%20Flash%20CTF/">View Writeups</a>
    </article>

    <article class="ctf-card">
      <div class="ctf-card-header">
        <span class="ctf-card-year">2025 · NexHunt</span>
        <span class="ctf-placement placement-top10">4th</span>
      </div>
      <h3 class="ctf-card-title">NexHunt CTF</h3>
      <p class="ctf-card-team">Team: THEM?!</p>
      <a class="ctf-card-btn" href="./NexHunt%20CTF/">View Writeups</a>
    </article>

    <article class="ctf-card">
      <div class="ctf-card-header">
        <span class="ctf-card-year">2025 · MCTF25</span>
        <span class="ctf-placement placement-top10">8th</span>
      </div>
      <h3 class="ctf-card-title">Mārtiņa-CTF 2025</h3>
      <p class="ctf-card-team">Team: Dikti cool ctf komanda</p>
      <a class="ctf-card-btn" href="./MCTF25/">View Writeups</a>
    </article>

    <article class="ctf-card">
      <div class="ctf-card-header">
        <span class="ctf-card-year">2025 · HeroCTF</span>
        <span class="ctf-placement">30th</span>
      </div>
      <h3 class="ctf-card-title">HeroCTF v7</h3>
      <p class="ctf-card-team">Team: ByteC4Ts</p>
      <a class="ctf-card-btn" href="./HeroCTF%20v7/">View Writeups</a>
    </article>

    <article class="ctf-card">
      <div class="ctf-card-header">
        <span class="ctf-card-year">2025 · BSides</span>
        <span class="ctf-placement">TBD</span>
      </div>
      <h3 class="ctf-card-title">BSides Algiers 2025</h3>
      <p class="ctf-card-team">Team: 0xFun · Scoreboard Frozen</p>
      <a class="ctf-card-btn" href="./BSides%20Algiers%202025/">View Writeups</a>
    </article>
  </div>
</section>

<!-- ===== ALL CHALLENGES BY CATEGORY ===== -->

<section id="all-challenges">
  <div class="section-header">
    <span class="section-icon">📋</span>
    <h2 class="section-title">All Challenge Writeups</h2>
  </div>
  <p class="section-subtitle">Quick navigation by category</p>

  <nav class="category-nav">
    <a class="category-chip" href="#cat-audio-web">Audio / Web</a>
    <a class="category-chip" href="#cat-blockchain">Blockchain</a>
    <a class="category-chip" href="#cat-blockchain-forensics">Blockchain / Forensics</a>
    <a class="category-chip" href="#cat-crypto">Crypto</a>
    <a class="category-chip" href="#cat-forensics">Forensics</a>
    <a class="category-chip" href="#cat-misc">Misc / Fun</a>
    <a class="category-chip" href="#cat-osint">OSINT</a>
    <a class="category-chip" href="#cat-pwn">Pwn / Docker</a>
    <a class="category-chip" href="#cat-prog">Prog</a>
    <a class="category-chip" href="#cat-re">Reverse Engineering</a>
    <a class="category-chip" href="#cat-system">System</a>
    <a class="category-chip" href="#cat-web">Web</a>
  </nav>

  <!-- Audio / Web -->
  <div class="challenge-section" id="cat-audio-web">
    <h3>Audio / Web</h3>
    <div class="challenge-list">
      <div class="challenge-list-header">
        <span>Challenge</span>
        <span>Writeup</span>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">Astral Pulses</span>
        <a class="challenge-btn" href="./MCTF25/Astral%20Pulses/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">AI Translator</span>
        <a class="challenge-btn" href="./MCTF25/AI%20Translator/">README</a>
      </div>
    </div>
  </div>

  <!-- Blockchain -->
  <div class="challenge-section" id="cat-blockchain">
    <h3>Blockchain</h3>
    <div class="challenge-list">
      <div class="challenge-list-header">
        <span>Challenge</span>
        <span>Writeup</span>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">Guess The Number</span>
        <a class="challenge-btn" href="./MCTF25/%5BBlockchain%203%5D%20Guess%20The%20Number/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">Magical RPC Button</span>
        <a class="challenge-btn" href="./MCTF25/%5BBlockchain%201%5D%20Magical%20RPC%20Button/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">Unlimited Void</span>
        <a class="challenge-btn" href="./MCTF25/Unlimited%20Void/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">Where Did I Leave My Flag</span>
        <a class="challenge-btn" href="./MCTF25/%5BBlockchain%202%5D%20Where%20Did%20I%20Leave%20My%20Flag/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">RustRoll</span>
        <a class="challenge-btn" href="./NexHunt%20CTF/RustRoll/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">BSides Only-invited party</span>
        <a class="challenge-btn" href="./BSides%20Algiers%202025/BSides%20Only-invited%20party/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">BSides Only-invited party REVENGE</span>
        <a class="challenge-btn" href="./BSides%20Algiers%202025/BSides%20Only-invited%20party%20REVENGE/">README</a>
      </div>
    </div>
  </div>

  <!-- Blockchain / Forensics -->
  <div class="challenge-section" id="cat-blockchain-forensics">
    <h3>Blockchain / Forensics</h3>
    <div class="challenge-list">
      <div class="challenge-list-header">
        <span>Challenge</span>
        <span>Writeup</span>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">Titanium Safe</span>
        <a class="challenge-btn" href="./MCTF25/Titanium%20Safe/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">Sacred Martins Sequence</span>
        <a class="challenge-btn" href="./MCTF25/%5BBlockchain%204%5D%20Sacred%20Martins%20Sequence/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">Sepolia Heist</span>
        <a class="challenge-btn" href="./MCTF25/Sepolia%20Heist/">README</a>
      </div>
    </div>
  </div>

  <!-- Crypto -->
  <div class="challenge-section" id="cat-crypto">
    <h3>Crypto</h3>
    <div class="challenge-list">
      <div class="challenge-list-header">
        <span>Challenge</span>
        <span>Writeup</span>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">Radical Security Animal</span>
        <a class="challenge-btn" href="./MCTF25/%5BCryptography%204%5D%20Radical%20Security%20Animal/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">Andor</span>
        <a class="challenge-btn" href="./HeroCTF%20v7/Andor/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">Genie</span>
        <a class="challenge-btn" href="./BSides%20Algiers%202025/Genie/">README</a>
      </div>
    </div>
  </div>

  <!-- Forensics -->
  <div class="challenge-section" id="cat-forensics">
    <h3>Forensics</h3>
    <div class="challenge-list">
      <div class="challenge-list-header">
        <span>Challenge</span>
        <span>Writeup</span>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">Rewritten History</span>
        <a class="challenge-btn" href="./MCTF25/Rewritten%20History/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">Operation Pensieve Breach - 1</span>
        <a class="challenge-btn" href="./HeroCTF%20v7/Operation%20Pensieve%20Breach%20-%201/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">Operation Pensieve Breach - 2</span>
        <a class="challenge-btn" href="./HeroCTF%20v7/Operation%20Pensieve%20Breach%20-%202/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">Making The Naughty List</span>
        <a class="challenge-btn" href="./MetaCTF%20December%202025%20Flash%20CTF/MakingTheNaughtyList/">README</a>
      </div>
    </div>
  </div>

  <!-- Misc / Fun -->
  <div class="challenge-section" id="cat-misc">
    <h3>Misc / Fun</h3>
    <div class="challenge-list">
      <div class="challenge-list-header">
        <span>Challenge</span>
        <span>Writeup</span>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">A series of tubes</span>
        <a class="challenge-btn" href="./MCTF25/A%20series%20of%20tubes/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">Jokemartins</span>
        <a class="challenge-btn" href="./MCTF25/Jokemartins/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">LSD#4</span>
        <a class="challenge-btn" href="./HeroCTF%20v7/LSD%234/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">Neverland</span>
        <a class="challenge-btn" href="./HeroCTF%20v7/Neverland/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">Bootloader</span>
        <a class="challenge-btn" href="./HeroCTF%20v7/Bootloader/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">the-scribe</span>
        <a class="challenge-btn" href="./NexHunt%20CTF/the-scribe/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">Sōkyoku</span>
        <a class="challenge-btn" href="./NexHunt%20CTF/S%C5%8Dkyoku/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">Perl Poetry</span>
        <a class="challenge-btn" href="./MetaCTF%20December%202025%20Flash%20CTF/PerlPoetry/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">Santa's Christmas Calculator</span>
        <a class="challenge-btn" href="./MetaCTF%20December%202025%20Flash%20CTF/SantasChristmasCalculator/">README</a>
      </div>
    </div>
  </div>

  <!-- OSINT -->
  <div class="challenge-section" id="cat-osint">
    <h3>OSINT</h3>
    <div class="challenge-list">
      <div class="challenge-list-header">
        <span>Challenge</span>
        <span>Writeup</span>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">A Lone Love</span>
        <a class="challenge-btn" href="./NexHunt%20CTF/A%20Lone%20Love/">README</a>
      </div>
    </div>
  </div>

  <!-- Pwn / Docker -->
  <div class="challenge-section" id="cat-pwn">
    <h3>Pwn / Docker</h3>
    <div class="challenge-list">
      <div class="challenge-list-header">
        <span>Challenge</span>
        <span>Writeup</span>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">ImgSharer</span>
        <a class="challenge-btn" href="./MCTF25/ImgSharer/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">Docker? I barely know her!</span>
        <a class="challenge-btn" href="./MCTF25/Docker,%20I%20barely%20know%20her!/">README</a>
      </div>
    </div>
  </div>

  <!-- Prog -->
  <div class="challenge-section" id="cat-prog">
    <h3>Prog</h3>
    <div class="challenge-list">
      <div class="challenge-list-header">
        <span>Challenge</span>
        <span>Writeup</span>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">PVE - Pirate Race #1</span>
        <a class="challenge-btn" href="./HeroCTF%20v7/PVE%20-%20Pirate%20Race%20%231/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">PVE - Pirate Race #2</span>
        <a class="challenge-btn" href="./HeroCTF%20v7/PVE%20-%20Pirate%20Race%20%232/">README</a>
      </div>
    </div>
  </div>

  <!-- Reverse Engineering -->
  <div class="challenge-section" id="cat-re">
    <h3>Reverse Engineering</h3>
    <div class="challenge-list">
      <div class="challenge-list-header">
        <span>Challenge</span>
        <span>Writeup</span>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">HuntMe1</span>
        <a class="challenge-btn" href="./NexHunt%20CTF/HuntMe1/">README</a>
      </div>
    </div>
  </div>

  <!-- System -->
  <div class="challenge-section" id="cat-system">
    <h3>System</h3>
    <div class="challenge-list">
      <div class="challenge-list-header">
        <span>Challenge</span>
        <span>Writeup</span>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">Middle Earth</span>
        <a class="challenge-btn" href="./HeroCTF%20v7/Middle%20Earth/">README</a>
      </div>
    </div>
  </div>

  <!-- Web -->
  <div class="challenge-section" id="cat-web">
    <h3>Web</h3>
    <div class="challenge-list">
      <div class="challenge-list-header">
        <span>Challenge</span>
        <span>Writeup</span>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">Gatekeeper</span>
        <a class="challenge-btn" href="./MCTF25/Gatekeeper/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">Homemade task system</span>
        <a class="challenge-btn" href="./MCTF25/Homemade%20task%20system/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">Homemade task system 2</span>
        <a class="challenge-btn" href="./MCTF25/Homemade%20task%20system%202/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">Homemade task system 3</span>
        <a class="challenge-btn" href="./MCTF25/Homemade%20task%20system%203/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">Parent Security</span>
        <a class="challenge-btn" href="./MCTF25/Parent%20Security/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">not!Windows registry</span>
        <a class="challenge-btn" href="./MCTF25/not!Windows%20registry/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">Dz-Kitab</span>
        <a class="challenge-btn" href="./NexHunt%20CTF/Dz-Kitab/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">Gigs</span>
        <a class="challenge-btn" href="./MetaCTF%20December%202025%20Flash%20CTF/Gigs/">README</a>
      </div>
    </div>
  </div>
</section>

<!-- ===== FOOTER ===== -->

<footer class="site-footer">
  <p class="footer-text">Built with 💙 by ANormalStick</p>
  <div class="footer-links">
    <a class="footer-link" href="https://github.com/ANormalStick/CTF-Writeups">Source Code</a>
  </div>
</footer>
