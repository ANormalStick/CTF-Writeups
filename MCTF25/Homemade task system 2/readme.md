## 2. Homemade task system 2

**Challenge ID:** Homemade task system 2  
**Author:** Mārtiņš #1337  
**Target:** `10.240.2.232:80` (Caddy HTTP server)

### Overview

This is a follow-up to *Homemade task system*. The author “fixes” the original problem by encoding file names, claiming this is “security by encoding”. The goal is to find the hidden non-indexed step and recover the flag.

### Enumeration

Port scan:

```bash
nmap -sC -sV 10.240.2.232
```

Only HTTP (port 80) is open.

Main page (`/`) shows an XP-themed “Task Tracker II” with a *Playbook steps* list:

```html
<li><a href="/dmllbnM=.html">Prepare</a></li>
<li><a href="/ZGl2aQ==.html">Identify</a></li>
<li><a href="/dHLEq3M=.html">Secure</a></li>
<li><a href="/xI1ldHJp.html">Contain</a></li>
```

The link names themselves are suspicious: each looks like Base64.

### Decoding the paths

Decode each name:

```bash
printf 'dmllbnM=' | base64 -d; echo
printf 'ZGl2aQ==' | base64 -d; echo
printf 'dHLEq3M=' | base64 -d; echo
printf 'xI1ldHJp' | base64 -d; echo
```

Results:

- `dmllbnM=` → `viens`
- `ZGl2aQ==` → `divi`
- `dHLEq3M=` → `trīs`
- `xI1ldHJp` → `četri`

These are Latvian words for 1, 2, 3, 4.

Each page again shows progress like `Playbook Progress: N / 5`, so there should be a fifth hidden step.

### Guessing the hidden page

By analogy, the missing step should be “five” in Latvian:

- `pieci` (5) → Base64: `cGllY2k=`

So the expected hidden file is:

```bash
curl -s http://10.240.2.232/cGllY2k=.html -o 5.html
```

This returns a **“Protect – Hidden Step”** page with a “Protected artifact” field:

```text
TUNURjI1e2VuYzBkMW5nXzE1bnRfdGgzX3M0bTNfYXNfM25jcnlwdDFvbn0=
```

### Recovering the flag

The “protected artifact” is just Base64 again. Decode it:

```bash
echo 'TUNURjI1e2VuYzBkMW5nXzE1bnRfdGgzX3M0bTNfYXNfM25jcnlwdDFvbn0=' | base64 -d
```

Decoded:

```text
MCTF25{enc0d1ng_15nt_th3_s4m3_as_3ncrypt1on}
```