<style>
:root {
  color-scheme: dark;
  --bg: #020617;
  --fg: #e5e7eb;
  --fg-muted: #9ca3af;
  --accent: #38bdf8;
  --border-subtle: #1f2937;
}

/* Base page */
html, body {
  margin: 0;
  padding: 0;
  background: radial-gradient(circle at top left, #020617, #020617 40%, #020617 100%);
  color: var(--fg);
  font-family: system-ui, -apple-system, BlinkMacSystemFont, "SF Pro Text", "Segoe UI", sans-serif;
}

/* GitHub Pages wrappers */
.page-content, .wrapper, article, .post {
  background: transparent !important;
  max-width: 960px;
  margin: 0 auto;
  padding: 2.5rem 1.5rem 4rem;
}

/* Headings */
.post h1, .page-content h1, article h1 {
  font-size: 1.6rem;
  margin-bottom: 0.6rem;
}

.post h2, .page-content h2, article h2 {
  font-size: 1.1rem;
  margin-top: 1.8rem;
  margin-bottom: 0.6rem;
  color: var(--fg-muted);
}

/* Tables (optional, if you use them) */
table {
  border-collapse: collapse;
  width: 100%;
  font-size: 0.85rem;
  margin: 0.4rem 0 0.8rem;
  border-radius: 0.6rem;
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
  background-color: rgba(15, 23, 42, 0.75);
}

tbody tr:last-child td {
  border-bottom: none;
}

/* Links */
a {
  color: var(--accent);
}

a:hover {
  text-decoration: none;
}

/* Code blocks */
pre,
code,
pre code,
.highlight,
.highlight pre,
.highlight code {
  background-color: rgba(15, 23, 42, 0.96) !important;
  color: var(--fg);
}

pre {
  border: 1px solid var(--border-subtle);
  padding: 0.85rem 1rem;
  border-radius: 0.5rem;
  overflow-x: auto;
  font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;
}

code {
  padding: 0.1rem 0.25rem;
  border-radius: 0.25rem;
  font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;
}
</style>


# Rewritten History — Writeup

**Category:** Forensics  
**Difficulty:** Easy–Medium  
**Flag format:** `MCTF25{flag}`  
**Flag:** `MCTF25{git_kn0ws_4ll}`  

---

## Challenge Description

> **Rewritten History**  
>
> You can get the flag for free on this challenge!  
>  
> Project is licenced under GNU LGPLv3, so source code must be provided :)

We’re given a downloadable project archive (e.g. `server.zip`) and a hint about licensing.  
The LGPLv3 requirement strongly suggests that **the full source code must be available**, even if the running version tries to hide it.

The challenge name “Rewritten History” is also a big hint: think **Git history**, commits, and deleted/rewritten code.

---

## Initial Recon

On an Ubuntu VM:

```bash
unzip server.zip
cd server
ls -a
```

We see a regular project layout, but the important part is that the archive also contains a **`.git` directory**:

```bash
ls -a
# ...
# .git
# ...
```

So we have the full Git repository, not just a snapshot of the current working tree.

If we view the current commit:

```bash
git log --oneline
```

The current `HEAD` doesn’t show anything obvious containing the flag (the running source may be “cleaned up” already).

Because the name is **“Rewritten History”**, we suspect the flag used to be in an older commit and was later removed (e.g. by `git rebase`, `git filter-branch`, or force-pushed changes).

---

## Digging Into Git History

Even if the visible log is short, Git often still keeps **orphaned objects** and **old blobs** around (at least until aggressive GC), especially in a shipped `.git` directory.

We can:

```bash
# Show all refs, including hidden ones
git show-ref

# List all objects that look like commits
git fsck --lost-found
```

Or just walk through commits and inspect files:

```bash
git log --oneline --all
git show <commit_hash>:path/to/file.py
```

After exploring the history and/or dangling commits, we find an older version of the main application logic (for example, `main.py`) that is **not present in the current tree**, but still stored as an object in the repository.

Example:

```bash
git cat-file -p <blob_or_commit_hash> > old_main.py
```

Opening this file (`old_main.py`) reveals some suspicious-looking code for handling the flag.

---

## Finding the Flag Logic

In that old source, we find a small obfuscation routine. It doesn’t print the flag directly; instead, it stores bytes in a list and XORs them:

```python
FLAG = [b for b in b"`strings`_proofed_xxp"]
FLAG_XOR = [45, 48, 32, 52, 91, 91, 28, 20, 9, 43, 47, 25, 1, 95, 17, 22, 59, 107, 20, 20, 13]

for i in range(len(FLAG_XOR)):
    FLAG[i] = FLAG_XOR[i % len(FLAG_XOR)] ^ FLAG[i]

print("FLAG:", bytes(FLAG).decode())
```

This code is clearly meant to **hide the real flag** from a simple “search for `MCTF`” or from `strings`, but it’s trivial to reverse.

We can either:

- Run the script directly:  
  ```bash
  python3 old_main.py
  ```
- Or copy the snippet into a new file (`solve.py`) and run it.

Running it prints:

```text
FLAG: MCTF25{git_kn0ws_4ll}
```

## Final Flag

```text
MCTF25{git_kn0ws_4ll}
```

---
