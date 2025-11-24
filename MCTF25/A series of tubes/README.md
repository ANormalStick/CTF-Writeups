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


# A series of tubes

## Challenge Overview

We’re given a shell one-liner that operates on some **hidden input** (marked as `[REDACTED]`) and a list of MD5 hashes as output. The task is to recover the original input / flag.

```sh
echo "[REDACTED]" | fold -w 3 | xargs -L1 -I {} sh -c 'echo "{}" | md5sum' | awk '{print $1}'
```

And the provided output:

```text
edd94d6e72b217ef22de208d5246300c
73a338a5cb6f23fb56cfd9320ea18414
3e3e0a54a3a0bced0522f414db8df3aa
665dc131f2c2104e7cba175cdb0f37d7
6bc898592761e343f5284ee0e1a641be
cf358237d3a6cbea098de5fb5b733f24
c670e2c9053c8e9e600c8bc01ddd16ae
010b08f101ec8929efff98e5e6418b7c
e0bc06e650fd4a19f67951e6cf0b3615
83d06450b3ad8b05ba802f0c117e706f
32c24e443f8997e7eb14a212dccb707d
54021079e528a133037e732a36774509
```

Goal: recover `[REDACTED]` and thus the flag.

---

## Step 1 – Understand the Shell Pipeline

Break the command down stage by stage:

```sh
echo "[REDACTED]"   | fold -w 3   | xargs -L1 -I {} sh -c 'echo "{}" | md5sum'   | awk '{print $1}'
```

1. `echo "[REDACTED]"`  
   Prints the original secret string followed by a newline (`\n`).

2. `fold -w 3`  
   Splits the input into fixed-width chunks of 3 characters per line.  
   For example, if the input was:

   ```text
   ABCDEFGHI
   ```

   then `fold -w 3` would output:

   ```text
   ABC
   DEF
   GHI
   ```

3. `xargs -L1 -I {} sh -c 'echo "{}" | md5sum'`  
   - `xargs -L1` takes **one line at a time** from stdin.  
   - For each line (each 3‑character chunk) it runs:
     ```sh
     sh -c 'echo "{}" | md5sum'
     ```
   - Important detail: `echo` appends a newline. So the data being hashed is actually:  
     **`<3 characters> + "\n"`**

4. `awk '{print $1}'`  
   `md5sum` prints the hash plus `-` (or filename). `awk '{print $1}'` keeps only the hash itself.

So the pipeline can be summarized as:

> Split the secret into 3‑character chunks and output the MD5 hash of each chunk (including a trailing newline).

---

## Step 2 – Infer the Structure of the Secret

We are given **12 hashes**, which correspond to **12 chunks** of 3 characters each.

That means the original secret (not counting the newline that `echo` added to the *whole* string) is:

```text
12 chunks × 3 characters = 36 characters
```

In CTFs, flags are often of the form:
```text
MCTF25{...}
```

So it’s reasonable to expect the secret is a flag with 36 characters total.

---

## Step 3 – Brute Forcing Each 3‑Character Chunk

Now that we know each hash is:
```text
MD5( three_characters + "\n" )
```

we can brute force each chunk independently.

### Brute-force strategy

1. Choose a character set for the chunks.  
   In practice, printable characters plus typical flag characters are enough:
   - Uppercase letters: `A–Z`
   - Lowercase letters: `a–z`
   - Digits: `0–9`
   - Symbol characters commonly used in flags: `{ } _`

2. For each of the 12 hashes:
   - Try all possible 3‑character combinations from the charset.
   - For each combination `s`, compute `md5(s + "\n")`.
   - When the hash matches, we’ve found the corresponding chunk.

Because each chunk is independent, this is a **12 × (search space)** problem, but the search space for “reasonable flag” characters is small enough for a quick brute force.

### Example brute-force script (Python)

```python
import hashlib
import string

hashes = [
    "edd94d6e72b217ef22de208d5246300c",
    "73a338a5cb6f23fb56cfd9320ea18414",
    "3e3e0a54a3a0bced0522f414db8df3aa",
    "665dc131f2c2104e7cba175cdb0f37d7",
    "6bc898592761e343f5284ee0e1a641be",
    "cf358237d3a6cbea098de5fb5b733f24",
    "c670e2c9053c8e9e600c8bc01ddd16ae",
    "010b08f101ec8929efff98e5e6418b7c",
    "e0bc06e650fd4a19f67951e6cf0b3615",
    "83d06450b3ad8b05ba802f0c117e706f",
    "32c24e443f8997e7eb14a212dccb707d",
    "54021079e528a133037e732a36774509",
]

charset = string.ascii_letters + string.digits + "{}_"

def md5_with_newline(s: str) -> str:
    return hashlib.md5((s + "\n").encode()).hexdigest()

chunks = []

for h in hashes:
    found = None
    for a in charset:
        for b in charset:
            for c in charset:
                s = a + b + c
                if md5_with_newline(s) == h:
                    found = s
                    print(f"{h} -> {s!r}")
                    break
            if found:
                break
        if found:
            break
    if not found:
        raise ValueError(f"No match found for hash {h}")
    chunks.append(found)

secret = "".join(chunks)
print("Recovered secret:", secret)
```

Running this script yields the following chunks (in order):

```text
MCT
F25
{p1
p1n
g_h
0t_
t45
k_y
3t_
n0_
p1p
15}
```

Concatenating them gives the original 36‑character secret.

---

## Step 4 – The Flag

Putting the chunks together:

```text
MCTF25{p1p1ng_h0t_t45k_y3t_n0_p1p15}
```
