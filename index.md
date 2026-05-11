---
title: "ANormalStick's Blog"
layout: default
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

/* Hide default Jekyll/GitHub Pages header and title */
.site-header, header.site-header,
.page-header, .project-name, .project-tagline,
h1.project-name, .site-title, header h1,
.post-title, article > h1:first-child,
main > h1:first-child, .container-lg > h1:first-child,
.markdown-body > h1:first-child {
  display: none !important;
}

/* Target the specific repo title that GitHub Pages adds */
body > main > h1:first-of-type,
.page-content > .wrapper > h1:first-of-type,
.page-content > h1:first-of-type {
  display: none !important;
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
  margin-bottom: 1rem;
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

/* Contact Info */
.contact-info {
  display: flex;
  flex-wrap: wrap;
  gap: 1.5rem;
  justify-content: center;
  margin-bottom: 1.5rem;
  font-size: 0.9rem;
  color: var(--fg-muted);
}

.contact-item {
  display: inline-flex;
  align-items: center;
  gap: 0.4rem;
}

.contact-item a {
  color: var(--accent);
}

.contact-item a::after { display: none !important; }

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

.stat-number.gold-text {
  background: linear-gradient(135deg, #ffd700, #ffec8b, #ffd700);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  text-shadow: 0 0 20px rgba(255, 215, 0, 0.3);
}

.stat-label {
  font-size: 0.75rem;
  color: var(--fg-muted);
  text-transform: uppercase;
  letter-spacing: 0.1em;
}

/* Subtitle for name */
.main-hero-subtitle {
  font-size: 1rem;
  color: var(--accent);
  margin: -0.3rem 0 0.8rem;
  font-weight: 500;
  letter-spacing: 0.05em;
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

.badge-forensics {
  background: linear-gradient(135deg, rgba(249, 115, 22, 0.3), rgba(249, 115, 22, 0.1));
  color: #fdba74;
  border: 1px solid rgba(249, 115, 22, 0.4);
}

.badge-osint {
  background: linear-gradient(135deg, rgba(20, 184, 166, 0.3), rgba(20, 184, 166, 0.1));
  color: #5eead4;
  border: 1px solid rgba(20, 184, 166, 0.4);
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

/* ===== CTF TIMELINE ===== */

.ctf-timeline {
  position: relative;
  margin-bottom: 3rem;
}

.ctf-timeline::before {
  content: "";
  position: absolute;
  left: 20px;
  top: 0;
  bottom: 0;
  width: 2px;
  background: linear-gradient(180deg, var(--accent), var(--accent-secondary), var(--accent-tertiary));
  border-radius: 2px;
}

.ctf-entry {
  position: relative;
  padding-left: 60px;
  padding-bottom: 2rem;
}

.ctf-entry:last-child {
  padding-bottom: 0;
}

.ctf-entry::before {
  content: "";
  position: absolute;
  left: 12px;
  top: 6px;
  width: 18px;
  height: 18px;
  border-radius: 50%;
  background: var(--bg);
  border: 3px solid var(--accent);
  z-index: 1;
}

.ctf-entry.gold::before {
  border-color: #ffd700;
  box-shadow: 0 0 15px rgba(255, 215, 0, 0.5);
}

.ctf-entry.silver::before {
  border-color: #c0c0c0;
  box-shadow: 0 0 15px rgba(192, 192, 192, 0.4);
}

.ctf-entry.bronze::before {
  border-color: #cd7f32;
  box-shadow: 0 0 15px rgba(205, 127, 50, 0.4);
}

.ctf-entry-card {
  background: linear-gradient(145deg, var(--bg-card), rgba(30, 41, 59, 0.7));
  border: 1px solid var(--border-subtle);
  border-radius: 1rem;
  padding: 1.5rem;
  transition: all 0.2s ease-out;
}

.ctf-entry-card:hover {
  transform: translateX(8px);
  border-color: rgba(56, 189, 248, 0.5);
  box-shadow: 0 15px 40px rgba(8, 47, 73, 0.5);
}

.ctf-entry-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 0.8rem;
  flex-wrap: wrap;
  gap: 0.5rem;
}

.ctf-entry-title {
  font-size: 1.25rem;
  font-weight: 600;
  margin: 0;
  color: var(--fg);
}

.ctf-rank-badge {
  display: inline-flex;
  align-items: center;
  gap: 0.3rem;
  padding: 0.35rem 0.9rem;
  border-radius: 999px;
  font-size: 0.85rem;
  font-weight: 700;
}

.ctf-rank-badge.gold {
  background: linear-gradient(135deg, #ffd700 0%, #ffec8b 50%, #ffd700 100%);
  color: #1a1a1a;
  box-shadow: 0 4px 15px rgba(255, 215, 0, 0.4);
}

.ctf-rank-badge.silver {
  background: linear-gradient(135deg, #e8e8e8 0%, #c0c0c0 50%, #e8e8e8 100%);
  color: #1a1a1a;
  box-shadow: 0 4px 15px rgba(192, 192, 192, 0.4);
}

.ctf-rank-badge.bronze {
  background: linear-gradient(135deg, #daa06d 0%, #cd7f32 50%, #daa06d 100%);
  color: #1a1a1a;
  box-shadow: 0 4px 15px rgba(205, 127, 50, 0.4);
}

.ctf-rank-badge.top10 {
  background: linear-gradient(135deg, rgba(34, 197, 94, 0.9), rgba(22, 163, 74, 0.9));
  color: #fff;
  box-shadow: 0 4px 15px rgba(34, 197, 94, 0.3);
}

.ctf-rank-badge.other {
  background: rgba(56, 189, 248, 0.2);
  color: var(--accent);
  border: 1px solid rgba(56, 189, 248, 0.4);
}

.ctf-entry-meta {
  display: flex;
  flex-wrap: wrap;
  gap: 1.5rem;
  margin-bottom: 1rem;
  font-size: 0.9rem;
  color: var(--fg-muted);
}

.ctf-meta-item {
  display: flex;
  align-items: center;
  gap: 0.4rem;
}

.ctf-entry-link {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.6rem 1.4rem;
  border-radius: 0.6rem;
  font-size: 0.85rem;
  font-weight: 500;
  background: linear-gradient(135deg, rgba(56, 189, 248, 0.2), rgba(56, 189, 248, 0.05));
  border: 1px solid rgba(56, 189, 248, 0.5);
  color: var(--accent);
  text-decoration: none;
  transition: all 0.15s ease-out;
}

.ctf-entry-link::after { display: none !important; }

.ctf-entry-link:hover {
  background: linear-gradient(135deg, rgba(56, 189, 248, 0.35), rgba(56, 189, 248, 0.1));
  transform: translateX(5px);
  box-shadow: 0 5px 20px rgba(56, 189, 248, 0.25);
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
    Computer Science student at University of Latvia. CTF player and member of <a href="https://ctftime.org/team/354033" style="color: var(--accent);">0xFUN</a> (Team Rank #7). Passionate about cybersecurity, game development, and building secure software solutions.
  </p>

  <div class="main-hero-links">
    <a class="hero-link" href="https://github.com/ANormalStick">📦 GitHub</a>
    <a class="hero-link" href="https://www.linkedin.com/in/j%C4%81nis-m%C4%81rti%C5%86%C5%A1-%C4%ABv%C4%81ns-0927962a0/">💼 LinkedIn</a>
    <a class="hero-link" href="https://ctftime.org/team/354033">🏴 0xFUN</a>
    <a class="hero-link" href="#projects">🎮 Projects</a>
    <a class="hero-link" href="#ctf-writeups">🚩 CTF Writeups</a>
    <a class="hero-link" href="#my-challenges">🧩 My Challenges</a>
  </div>

  <div class="contact-info">
    <span class="contact-item">💬 Discord: <strong>ANormalStick</strong></span>
    <span class="contact-item">📧 <a href="mailto:agitaundainis@gmail.com">agitaundainis@gmail.com</a></span>
  </div>

  <div class="main-hero-stats">
    <div class="stat-item">
      <div class="stat-number">8</div>
      <div class="stat-label">CTFs Documented</div>
    </div>
    <div class="stat-item">
      <div class="stat-number gold-text">1st</div>
      <div class="stat-label">Best Placement</div>
    </div>
    <div class="stat-item">
      <div class="stat-number">#7</div>
      <div class="stat-label">Team Global Rank</div>
    </div>
    <div class="stat-item">
      <div class="stat-number">9</div>
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
      <span class="project-badge badge-game">University Project</span>
      <h3 class="project-title">Shattered Towers</h3>
      <p class="project-desc">
        A 2D puzzle platformer built with Godot 4.5 featuring a unique dimension-switching mechanic. Players switch between Hope and Despair dimensions, affecting platform visibility and level layout. Includes wall sliding, dashing, and custom shaders for dimension transitions.
      </p>
      <div class="project-tags">
        <span class="project-tag">Godot 4.5</span>
        <span class="project-tag">GDScript</span>
        <span class="project-tag">2D Platformer</span>
        <span class="project-tag">Shaders</span>
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
        <a class="project-btn" href="./ChallangesIMade/MusicBoxV2/Music%20Box%20v2.7z">Download</a>
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
        <a class="project-btn" href="./ChallangesIMade/NaughtyOrNice/NaughtyOrNice.zip">Download</a>
      </div>
    </article>

    <article class="project-card">
      <span class="project-badge badge-misc">Minecraft / Forensics / OSINT</span>
      <h3 class="project-title">ANormalJourney</h3>
      <p class="project-desc">
        A Minecraft world forensics puzzle. Recover the creator's last logout position from NBT data, decode Base64 clues hidden in books, use bedrock pattern matching to locate the final flag stash.
      </p>
      <div class="project-tags">
        <span class="project-tag">Minecraft</span>
        <span class="project-tag">NBT Analysis</span>
        <span class="project-tag">Base64</span>
        <span class="project-tag">Bedrock Patterns</span>
      </div>
      <div class="project-actions">
        <a class="project-btn" href="./ChallangesIMade/ANormalJourney/">Writeup</a>
        <a class="project-btn" href="./ChallangesIMade/ANormalJourney/ANormalSticks%20Let%27s%20Play%20%231.rar">Download</a>
      </div>
    </article>

    <article class="project-card">
      <span class="project-badge badge-misc">Signal Processing</span>
      <h3 class="project-title">Lines of Contact</h3>
      <p class="project-desc">
        Decode a deep-space audio transmission hiding raster images. Detect sync pulses in a WAV file, extract scanlines, and reconstruct hidden pictures — Golden Record style.
      </p>
      <div class="project-tags">
        <span class="project-tag">Audio Analysis</span>
        <span class="project-tag">Raster Decoding</span>
        <span class="project-tag">Signal Processing</span>
      </div>
      <div class="project-actions">
        <a class="project-btn" href="./ChallangesIMade/LinesOfContact/">Writeup</a>
        <a class="project-btn" href="./ChallangesIMade/LinesOfContact/Lines%20of%20Contact.rar">Download</a>
      </div>
    </article>

    <article class="project-card">
      <span class="project-badge badge-forensics">Forensics</span>
      <h3 class="project-title">Pixel Rehab</h3>
      <p class="project-desc">
        A corrupted PNG with a hidden 7z archive appended after the IEND chunk. Fix the signature byte, parse PNG chunks to find the trailer, swap the magic bytes, and extract the real flag.
      </p>
      <div class="project-tags">
        <span class="project-tag">PNG Forensics</span>
        <span class="project-tag">File Carving</span>
        <span class="project-tag">7z Archive</span>
      </div>
      <div class="project-actions">
        <a class="project-btn" href="./ChallangesIMade/PixelRehab/">Writeup</a>
        <a class="project-btn" href="./ChallangesIMade/PixelRehab/pixel.fun">Download</a>
      </div>
    </article>

    <article class="project-card">
      <span class="project-badge badge-misc">Misc</span>
      <h3 class="project-title">Skyglyph I: Guide Star</h3>
      <p class="project-desc">
        A star-tracker calibration puzzle. Use labeled guide stars to fit a camera model with radial distortion, then invert it to map all detections back to sky coordinates. The brightest stars spell a hidden message.
      </p>
      <div class="project-tags">
        <span class="project-tag">Astrometry</span>
        <span class="project-tag">Camera Calibration</span>
        <span class="project-tag">Gnomonic Projection</span>
      </div>
      <div class="project-actions">
        <a class="project-btn" href="./ChallangesIMade/SkyglyphIGuideStar/">Writeup</a>
        <a class="project-btn" href="./ChallangesIMade/SkyglyphIGuideStar/Skyglyph%20I%20Guide%20Star.zip">Download</a>
      </div>
    </article>

    <article class="project-card">
      <span class="project-badge badge-misc">Misc / Crypto / Forensics</span>
      <h3 class="project-title">Skyglyph II: Blind Drift</h3>
      <p class="project-desc">
        Blind plate-solve 4 noisy star frames against a catalog, extract matching star IDs, derive per-frame ChaCha20-Poly1305 keys, and decrypt flag parts. AEAD authentication enforces perfect correctness.
      </p>
      <div class="project-tags">
        <span class="project-tag">Plate Solving</span>
        <span class="project-tag">ChaCha20-Poly1305</span>
        <span class="project-tag">RANSAC</span>
        <span class="project-tag">SHA256</span>
      </div>
      <div class="project-actions">
        <a class="project-btn" href="./ChallangesIMade/SkyglyphIIBlindDrift/">Writeup</a>
        <a class="project-btn" href="./ChallangesIMade/SkyglyphIIBlindDrift/Skyglyph%20II%20Blind%20Drift.zip">Download</a>
      </div>
    </article>

    <article class="project-card">
      <span class="project-badge badge-osint">OSINT</span>
      <h3 class="project-title">Temptation. Stone. Silence.</h3>
      <p class="project-desc">
        Three images, three fragments — each pointing to a Latvian place. Use reverse image search, identify carved faces, and trace devil folklore to pinpoint three locations with proper Latvian diacritics.
      </p>
      <div class="project-tags">
        <span class="project-tag">Reverse Image Search</span>
        <span class="project-tag">Geolocation</span>
        <span class="project-tag">Latvian Culture</span>
      </div>
      <div class="project-actions">
        <a class="project-btn" href="./ChallangesIMade/TemptationStoneSilence/">Writeup</a>
        <a class="project-btn" href="./ChallangesIMade/TemptationStoneSilence/Temptation.%20Stone.%20Silence..zip">Download</a>
      </div>
    </article>

    <article class="project-card">
      <span class="project-badge badge-osint">OSINT / GTA V</span>
      <h3 class="project-title">Where's Franklin?</h3>
      <p class="project-desc">
        A GTA V geolocation challenge. Given a screenshot of Franklin standing next to a road, identify the exact in-game location using database lookups or manual map exploration.
      </p>
      <div class="project-tags">
        <span class="project-tag">GTA V</span>
        <span class="project-tag">Geolocation</span>
        <span class="project-tag">GeoGuessr</span>
      </div>
      <div class="project-actions">
        <a class="project-btn" href="./ChallangesIMade/WheresFranklin/">Writeup</a>
        <a class="project-btn" href="./ChallangesIMade/WheresFranklin/e0652c78-acce-48d5-86e2-5106bb6e6248.jpg">Download</a>
      </div>
    </article>
  </div>
</section>

<!-- ===== CTF WRITEUPS SECTION ===== -->

<section id="ctf-writeups">
  <div class="section-header">
    <span class="section-icon">🚩</span>
    <h2 class="section-title">CTF Competition History</h2>
  </div>
  <p class="section-subtitle">Documented competitions with detailed writeups</p>

  <div class="ctf-timeline">

    <div class="ctf-entry bronze">
      <div class="ctf-entry-card">
        <div class="ctf-entry-header">
          <h3 class="ctf-entry-title">0xLaugh CTF V5</h3>
          <span class="ctf-rank-badge bronze">3rd Place</span>
        </div>
        <div class="ctf-entry-meta">
          <span class="ctf-meta-item">📅 February 2026</span>
          <span class="ctf-meta-item">👥 Team: 0xFUN</span>
          <span class="ctf-meta-item">🏷️ Blockchain, Crypto, DFIR, Pwn, RE, Web</span>
        </div>
        <a class="ctf-entry-link" href="./0xLaugh%20CTF%20V5/">View Writeups →</a>
      </div>
    </div>

    <div class="ctf-entry">
      <div class="ctf-entry-card">
        <div class="ctf-entry-header">
          <h3 class="ctf-entry-title">UofTCTF 2026</h3>
          <span class="ctf-rank-badge other">22nd Place</span>
        </div>
        <div class="ctf-entry-meta">
          <span class="ctf-meta-item">📅 January 2026</span>
          <span class="ctf-meta-item">👥 Team: 0xFUN</span>
          <span class="ctf-meta-item">🏷️ Rev, PWN, Web, Crypto, Forensics, Misc</span>
        </div>
        <a class="ctf-entry-link" href="./UofTCTF%202026/">View Writeups →</a>
      </div>
    </div>

    <div class="ctf-entry top10">
      <div class="ctf-entry-card">
        <div class="ctf-entry-header">
          <h3 class="ctf-entry-title">Scarlet CTF 2026</h3>
          <span class="ctf-rank-badge top10">8th Place</span>
        </div>
        <div class="ctf-entry-meta">
          <span class="ctf-meta-item">📅 January 2026</span>
          <span class="ctf-meta-item">👥 Team: 0xFUN</span>
          <span class="ctf-meta-item">🏷️ Web, Crypto, Forensics, OSINT</span>
        </div>
        <a class="ctf-entry-link" href="./Scarlet%20CTF/">View Writeups →</a>
      </div>
    </div>
    
    <div class="ctf-entry gold">
      <div class="ctf-entry-card">
        <div class="ctf-entry-header">
          <h3 class="ctf-entry-title">BSides Algiers 2025</h3>
          <span class="ctf-rank-badge gold">1st Place</span>
        </div>
        <div class="ctf-entry-meta">
          <span class="ctf-meta-item">📅 2025</span>
          <span class="ctf-meta-item">👥 Team: 0xFUN</span>
          <span class="ctf-meta-item">🏷️ Blockchain, Crypto</span>
        </div>
        <a class="ctf-entry-link" href="./BSides%20Algiers%202025/">View Writeups →</a>
      </div>
    </div>

    <div class="ctf-entry bronze">
      <div class="ctf-entry-card">
        <div class="ctf-entry-header">
          <h3 class="ctf-entry-title">MetaCTF December 2025 Flash CTF</h3>
          <span class="ctf-rank-badge bronze">3rd Place</span>
        </div>
        <div class="ctf-entry-meta">
          <span class="ctf-meta-item">📅 December 2025</span>
          <span class="ctf-meta-item">👤 Solo</span>
          <span class="ctf-meta-item">🏷️ Misc, Forensics, Web</span>
        </div>
        <a class="ctf-entry-link" href="./MetaCTF%20December%202025%20Flash%20CTF/">View Writeups →</a>
      </div>
    </div>

    <div class="ctf-entry">
      <div class="ctf-entry-card">
        <div class="ctf-entry-header">
          <h3 class="ctf-entry-title">NexHunt CTF</h3>
          <span class="ctf-rank-badge top10">4th Place</span>
        </div>
        <div class="ctf-entry-meta">
          <span class="ctf-meta-item">📅 2025</span>
          <span class="ctf-meta-item">👥 Team: THEM?!</span>
          <span class="ctf-meta-item">🏷️ OSINT, Misc, Web, RE</span>
        </div>
        <a class="ctf-entry-link" href="./NexHunt%20CTF/">View Writeups →</a>
      </div>
    </div>

    <div class="ctf-entry">
      <div class="ctf-entry-card">
        <div class="ctf-entry-header">
          <h3 class="ctf-entry-title">Mārtiņa-CTF 2025</h3>
          <span class="ctf-rank-badge top10">8th Place</span>
        </div>
        <div class="ctf-entry-meta">
          <span class="ctf-meta-item">📅 2025</span>
          <span class="ctf-meta-item">👥 Team: Dikti cool ctf komanda</span>
          <span class="ctf-meta-item">🏷️ Blockchain, Web, Pwn</span>
        </div>
        <a class="ctf-entry-link" href="./MCTF25/">View Writeups →</a>
      </div>
    </div>

    <div class="ctf-entry">
      <div class="ctf-entry-card">
        <div class="ctf-entry-header">
          <h3 class="ctf-entry-title">HeroCTF v7</h3>
          <span class="ctf-rank-badge other">30th Place</span>
        </div>
        <div class="ctf-entry-meta">
          <span class="ctf-meta-item">📅 2025</span>
          <span class="ctf-meta-item">👥 Team: ByteC4Ts</span>
          <span class="ctf-meta-item">🏷️ Crypto, Forensics, Prog, System</span>
        </div>
        <a class="ctf-entry-link" href="./HeroCTF%20v7/">View Writeups →</a>
      </div>
    </div>

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
    <a class="category-chip" href="#cat-dfir">DFIR</a>
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
      <div class="challenge-row">
        <span class="challenge-name">House of Illusions</span>
        <a class="challenge-btn" href="./0xLaugh%20CTF%20V5/House%20of%20Illusions/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">Void Bound Blade</span>
        <a class="challenge-btn" href="./0xLaugh%20CTF%20V5/Void%20Bound%20Blade/">README</a>
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
      <div class="challenge-row">
        <span class="challenge-name">Coloring Fraud</span>
        <a class="challenge-btn" href="./Scarlet%20CTF/Coloring%20Fraud/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">Coloring Heist</span>
        <a class="challenge-btn" href="./Scarlet%20CTF/Coloring%20Heist/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">Gambler's Fallacy</span>
        <a class="challenge-btn" href="./UofTCTF%202026/Gambler's%20Fallacy/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">Reduced Dimension</span>
        <a class="challenge-btn" href="./0xLaugh%20CTF%20V5/Reduced%20Dimension/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">SCA2</span>
        <a class="challenge-btn" href="./0xLaugh%20CTF%20V5/SCA2/">README</a>
      </div>
    </div>
  </div>

  <!-- DFIR -->
  <div class="challenge-section" id="cat-dfir">
    <h3>DFIR</h3>
    <div class="challenge-list">
      <div class="challenge-list-header">
        <span>Challenge</span>
        <span>Writeup</span>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">The Hood</span>
        <a class="challenge-btn" href="./0xLaugh%20CTF%20V5/The%20Hood/">README</a>
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
      <div class="challenge-row">
        <span class="challenge-name">Sad Face</span>
        <a class="challenge-btn" href="./Scarlet%20CTF/Sad%20Face/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">My Pokemon Card is Fake!</span>
        <a class="challenge-btn" href="./UofTCTF%202026/My%20Pokemon%20Card%20is%20Fake!/">README</a>
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
      <div class="challenge-row">
        <span class="challenge-name">Lottery</span>
        <a class="challenge-btn" href="./UofTCTF%202026/Lottery/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">Vibe Code</span>
        <a class="challenge-btn" href="./UofTCTF%202026/Vibe%20Code/">README</a>
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
      <div class="challenge-row">
        <span class="challenge-name">Scouts Honor 2.0</span>
        <a class="challenge-btn" href="./Scarlet%20CTF/Scouts%20Honor%202.0/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">Stuck In The Middle With You</span>
        <a class="challenge-btn" href="./Scarlet%20CTF/Stuck%20In%20The%20Middle%20With%20You/">README</a>
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
      <div class="challenge-row">
        <span class="challenge-name">extended-eBPF</span>
        <a class="challenge-btn" href="./UofTCTF%202026/extended-eBPF/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">House Of Pain</span>
        <a class="challenge-btn" href="./0xLaugh%20CTF%20V5/House%20Of%20Pain/">README</a>
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
      <div class="challenge-row">
        <span class="challenge-name">Bring Your Own Program</span>
        <a class="challenge-btn" href="./UofTCTF%202026/Bring%20Your%20Own%20Program/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">Symbol of Hope</span>
        <a class="challenge-btn" href="./UofTCTF%202026/Symbol%20of%20Hope/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">12</span>
        <a class="challenge-btn" href="./0xLaugh%20CTF%20V5/12/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">Psycho Flag</span>
        <a class="challenge-btn" href="./0xLaugh%20CTF%20V5/Psycho%20Flag/">README</a>
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
      <div class="challenge-row">
        <span class="challenge-name">Campus One</span>
        <a class="challenge-btn" href="./Scarlet%20CTF/Campus%20One/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">Mole in the Wall</span>
        <a class="challenge-btn" href="./Scarlet%20CTF/Mole%20in%20the%20Wall/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">Firewall</span>
        <a class="challenge-btn" href="./UofTCTF%202026/Firewall/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">Personal Blog</span>
        <a class="challenge-btn" href="./UofTCTF%202026/Personal%20Blog/">README</a>
      </div>
      <div class="challenge-row">
        <span class="challenge-name">PDF.EXE</span>
        <a class="challenge-btn" href="./0xLaugh%20CTF%20V5/PDF.EXE/">README</a>
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
