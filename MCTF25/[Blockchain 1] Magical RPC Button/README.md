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


# Blockchain 1 – Magical RPC Button

**Category:** Blockchain  
**RPC Port:** `8545`  
**TCP helper Port:** `31337`  

> _“Seems this has been already solved? Can you get the flag?”_

This is the introductory blockchain challenge. The idea is that the blockchain instance is already in a “solved” state, or the contract exposes the flag via a read-only function, and your task is simply to use RPC to retrieve it.

We didn’t dump the exact bytecode in this session, but the typical pattern for this type of intro task is:

```solidity
contract MagicalRPCButton {
    string public flag = "MCTF25{example_flag}";

    function isSolved() external pure returns (bool) {
        return true;
    }
}
```

The important part is again `string public flag;` – Solidity auto-generates a public getter `flag()`.

---

## Exploitation Steps (Generic Pattern)

### 1. Get instance info

Connect to the helper for this challenge’s instance:

```bash
nc <CHALLENGE_IP> 31337
# or
telnet <CHALLENGE_IP> 31337
```

Then:

```text
> info
id: MagicalRPCButton
rpc_port: 8545
chain_id: 31337
contract: 0xABCDEF...
deployer: 0x...
```

Now set your environment:

```bash
export RPC_URL=http://<CHALLENGE_IP>:8545
export CONTRACT=0xABCDEF...

cast chain-id --rpc-url $RPC_URL   # should print 31337
```

### 2. Read the flag via the public getter

Because the variable is `string public flag`, the getter is `flag()(string)`:

```bash
cast call $CONTRACT "flag()(string)" --rpc-url $RPC_URL
```

This returns the flag directly, e.g.:

```text
MCTF25{7his_w4s_sup3r_dup3r_34sy_lik3_m4gic}
```

No transaction, no gas – just a view call.

### 3. Confirm with the helper (if needed)

```bash
nc <CHALLENGE_IP> 31337
> flag
```

The challenge manager will either show the same flag or simply confirm the challenge is solved.

---

### 4. Flag

MCTF25{7his_w4s_sup3r_dup3r_34sy_lik3_m4gic}
