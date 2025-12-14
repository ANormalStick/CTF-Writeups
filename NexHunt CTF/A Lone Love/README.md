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

# A Lone Love - Write-up

**Flag format:** `nexus{surname_day_month_year}`

## Summary
The challenge is OSINT: identify the **only survivor** from a WWII photo roster (August 1942) and the **date he was listed missing**, then format as a flag.

## 1) Find the source (“history” content)
The task text (Russian, *August 1942*, “twelve officers and a Red Army man”) is distinctive. Searching unique strings (e.g., `Август 1942 года` + “двенадцать офицеров”) leads to a PDF from the **Всероссийский литературный конкурс «Герои Великой Победы»** that contains the exact story and roster.

## 2) Locate the relevant passage
In the PDF:
- Use **Ctrl+F** for `Август 1942 года`.
- This jumps to the story describing a photo and listing the people on it.

CLI alternative:
```bash
pdftotext book.pdf - | grep -n "Август 1942 года"
```

## 3) Extract the roster (12 people)
The article lists the people on the back of the photo, including **Богатырев** among the twelve.

## 4) Identify the lone survivor
Later, the author describes searching archival portals and highlights **Николай Иванович Богатырев** as:
- “пропавший без вести” (missing),
- **but survived and made it to the end** (“выживший и дошедший до конца”).

This matches the challenge clue: the **only unexpected survivor**.

## 5) Get the “missing” date
In the same section, the author quotes the loss report line:
- `Пропал без вести 04.11.1944 ...`

So the missing date is **04.11.1944** (4 Nov 1944).

## 6) Build the flag
- surname: `bogatyrev`
- day: `four`
- month: `nov`
- year: `fortyfour`

**Flag:** `nexus{bogatyrev_four_nov_fortyfour}`
