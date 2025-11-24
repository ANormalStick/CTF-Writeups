## Homemade task system

**Challenge ID:** Homemade task system  
**Author:** Mārtiņš #1337  
**Target:** `10.240.2.228:80` (Caddy HTTP server)

### Overview

A retro Windows 98–style “task tracker” site exposes a small checklist of steps for building a CTF challenge. The visible UI shows 4 steps, but the progress indicator hints at 5 total steps. The goal is to find the missing step and recover the flag.

### Enumeration

Basic port scan:

```bash
nmap -sC -sV 10.240.2.228
```

Only port 80 is open, running Caddy with the title *“Mārtiņš’ Task Tracker”*.

The main page (`/`) lists four steps:

- `/1.html` — Plan  
- `/2.html` — Build  
- `/3.html` — Integrate and push  
- `/4.html` — CTFd  

Each step page is a simple HTML file. The key detail is in the status bar on the step pages:
- Step 1: `Wizard Progress: 1 / 5`
- Step 2: `Wizard Progress: 2 / 5`
- Step 3: `Wizard Progress: 3 / 5`
- Step 4: `Wizard Progress: 4 / 5`

The index only links to 4 steps, but the progress clearly expects 5, which implies a non-indexed fifth page.

### Exploitation

The obvious guess for the missing page is a numbered step:

```bash
curl http://10.240.2.228/5.html
```

This hidden page exists and contains the final “step” plus the flag. The exact flag value depends on the running instance, but it is presented in the usual format:

```text
MCTF25{f4iling_t0_pl4n_m34n5_pl4nn1ng_t0_f4il}
```
