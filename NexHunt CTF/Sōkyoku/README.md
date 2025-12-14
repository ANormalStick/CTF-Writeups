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

# Sōkyoku — Covert Timing Channel (incident.log) Writeup

**Category:** Forensics / Covert Channels  
**Author:** N!L  
**Flag format:** `nexus{...}`  

**Flag:** `nexus{1m_r3a11y_pr0ud_0f_yuu_d3t3ct0r}`

---

## Summary

This challenge hides data inside seemingly benign system logs by abusing **timing anomalies**:

- Kernel log lines of the form `scheduler: tick processing delayed (NNNNus)` act as the covert channel.
- The delay values are quantized to multiples of **1000µs** (1–63 ms), which can be mapped to **Base64** symbols.
- `audit: comm_marker` entries provide **framing boundaries** (start/end) for multiple transmissions.
- A final “payload” segment is **XOR-encrypted**; the XOR key (`shadow`) is leaked in audit logs (`key_material="..."`).

---

## Files

- `incident.log` (given)

---

## Key Observations

### 1) Covert channel carrier: scheduler delay microseconds

In the log there are hundreds of entries like:

```
kernel: ... scheduler: tick processing delayed (23000us) on cpu7
```

When extracted, all delays satisfy:

- `delay_us % 1000 == 0`
- `delay_us // 1000` lies in a small alphabet (mostly 1..63)

That’s a classic “symbol stream” hiding in timing values.

### 2) Framing: audit comm_marker boundaries

The log includes explicit markers:

```
audit: comm_marker group=LOTUS state=SEND_START ...
audit: comm_marker group=LOTUS state=SEND_END ...
audit: comm_marker group=SPIRE state=SEND_START ...
audit: comm_marker group=LOTUS state=PAYLOAD_START ...
audit: comm_marker group=LOTUS state=PAYLOAD_END ...
```

These timestamps divide the delay stream into multiple message segments (two for LOTUS + two for SPIRE + one payload).

---

## Step-by-step Solution

### Step 1 — Parse markers and scheduler-delay lines

1. Parse all `audit: comm_marker` lines into `(timestamp, group, state)`.
2. Parse all scheduler-delay lines into `(timestamp, delay_us)`.

This lets you select “the delays that occurred between marker X and marker Y”.

### Step 2 — Segment the stream

Using the markers, the scheduler stream splits into five segments:

1. **LOTUS SEND** (handshake)  
2. **SPIRE SEND** (response)  
3. **LOTUS SEND** (follow-up / instructions)  
4. **SPIRE SEND** (handoff)  
5. **LOTUS PAYLOAD** (binary payload)

The counts in the provided log add up exactly to the full scheduler-delay list, confirming the framing is correct.

### Step 3 — Map delay symbols to Base64 characters

Let:

```
symbol = delay_us // 1000
```

Then map to Base64 with an off-by-one shift:

```
alphabet = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"
char = alphabet[symbol - 1]
```

This produces a Base64 string per segment.

### Step 4 — Base64-decode each segment

Segments 1–4 decode cleanly to ASCII messages:

- `[LOTUS] Channel established. Host logging is verbose - perfect cover. Standby for key rotation.`
- `[SPIRE] Acknowledged. Monitor for anomaly detection. Key distribution via policy fragments?`
- `[LOTUS] Affirmative. Fragmented in audit trail. Extract shadow-class identifier. No cleartext.`
- `[SPIRE] Confirmed. Payload handoff when ready. Standard protocol.`

Segment 5 decodes to a **binary blob** (not plaintext).

### Step 5 — Recover the XOR key from audit “key_material”

Later in the log, there are AppArmor audit entries containing:

```
key_material="c2hhZG93"
key_material="ZnJhZ18wXw=="
key_material="ZnJhZ18xXw=="
...
```

These are Base64 strings. Decoding them yields:

- `c2hhZG93` → `shadow`
- `ZnJhZ18wXw==` → `frag_0_` (and similar)

The LOTUS instruction explicitly mentions:

> “Extract shadow-class identifier.”

So the XOR key is:

```
key = b"shadow"
```

### Step 6 — XOR-decrypt the payload → flag

The payload bytes are XORed with the repeating key:

```
plaintext[i] = payload[i] XOR key[i % len(key)]
```

Decryption reveals:

```
nexus{1m_r3a11y_pr0ud_0f_yuu_d3t3ct0r}
```

---

## Proof of Work — Full Solver Script

Save as `solve.py` and run against `incident.log`.

```python
import re
import base64
from datetime import datetime

LOG = "incident.log"
alphabet = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"

def parse_dt(line: str):
    p = line.split()
    return datetime.strptime(p[0] + " " + p[1], "%Y-%m-%d %H:%M:%S")

def b64decode_relaxed(s: str) -> bytes:
    s = s + "=" * ((4 - (len(s) % 4)) % 4)
    return base64.b64decode(s)

with open(LOG, "r", errors="replace") as f:
    lines = f.read().splitlines()

# --- collect comm markers ---
markers = []
for l in lines:
    if "audit: comm_marker" in l:
        dt = parse_dt(l)
        m = re.search(r'group=(\w+)\s+state=(\w+)', l)
        if m:
            markers.append((dt, m.group(1), m.group(2)))

def get_marker(group, state, occurrence=1):
    hits = [dt for dt,g,s in markers if g == group and s == state]
    if len(hits) < occurrence:
        raise ValueError(f"marker not found: {group} {state} occ={occurrence}")
    return hits[occurrence - 1]

# marker windows (per the given log)
lotus1_start = get_marker("LOTUS", "SEND_START", 1)
lotus1_end   = get_marker("LOTUS", "SEND_END",   1)
spire1_start = get_marker("SPIRE", "SEND_START", 1)
spire1_end   = get_marker("SPIRE", "SEND_END",   1)
lotus2_start = get_marker("LOTUS", "SEND_START", 2)
lotus2_end   = get_marker("LOTUS", "SEND_END",   2)
spire2_start = get_marker("SPIRE", "SEND_START", 2)
spire2_end   = get_marker("SPIRE", "SEND_END",   2)
pay_start    = get_marker("LOTUS", "PAYLOAD_START", 1)
pay_end      = get_marker("LOTUS", "PAYLOAD_END",   1)

# --- scheduler delay stream ---
sched = []
for l in lines:
    if "scheduler: tick processing delayed" in l:
        dt = parse_dt(l)
        m = re.search(r"\((\d+)us\)", l)
        if m:
            sched.append((dt, int(m.group(1))))

def window_to_b64(start, end):
    chunk = [us for (dt, us) in sched if start <= dt <= end]
    # off-by-one mapping discovered in analysis:
    return "".join(alphabet[(us // 1000) - 1] for us in chunk)

def decode_window(start, end):
    return b64decode_relaxed(window_to_b64(start, end))

# decode the four plaintext segments
m1 = decode_window(lotus1_start, lotus1_end).decode()
m2 = decode_window(spire1_start, spire1_end).decode()
m3 = decode_window(lotus2_start, lotus2_end).decode()
m4 = decode_window(spire2_start, spire2_end).decode()

print(m1)
print(m2)
print(m3)
print(m4)

# payload
payload_b64 = window_to_b64(pay_start, pay_end)
payload = b64decode_relaxed(payload_b64)

# XOR key from key_material -> "shadow"
key = b"shadow"
flag = bytes(payload[i] ^ key[i % len(key)] for i in range(len(payload)))

print(flag.decode())
```

Expected final line:

```
nexus{1m_r3a11y_pr0ud_0f_yuu_d3t3ct0r}
```

---

## Flag

`nexus{1m_r3a11y_pr0ud_0f_yuu_d3t3ct0r}`
