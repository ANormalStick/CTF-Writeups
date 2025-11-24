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
