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


# Blockchain 2 – Where Did I Leave My Flag

**Category:** Blockchain  
**Host:** `10.240.3.20`  
**RPC:** `http://10.240.3.20:8545`  
**TCP helper:** `10.240.3.20:31337`  

---

## Challenge Description

> Yoo! This blockchain stuff is very cool! I think im getting the hang of it altho... I think I lost my flag somewhere... Can you find it?

We are given a contract address and deployer, and the challenge name suggests the flag is “somewhere on-chain”.  
The provided Solidity code is:

```solidity
contract WhereDidILeaveMyFlag {
    string public flag;

    constructor(string memory _flag) {
        flag = _flag;
    }

    function isSolved() external pure returns (bool) {
        return false;
    }
}
```

The important part: `string public flag;` – Solidity auto-generates a getter `flag()`.  
So the flag is already available via a read-only call; no tx needed.

---

## Exploitation Steps

### 1. Get instance info

From the helper:

```bash
nc 10.240.3.20 31337
> info
id: WhereDidILeaveMyFlag
rpc_port: 8545
chain_id: 31337
contract: 0x817C4554ba65c39F71Ede6389be31573B9C6ee78
deployer: 0xCEF32d52c8fc929B192Dfbb70cFf9078C6A35de8
```

Set environment:

```bash
export RPC_URL=http://10.240.3.20:8545
export CONTRACT=0x817C4554ba65c39F71Ede6389be31573B9C6ee78

cast chain-id --rpc-url $RPC_URL   # 31337
```

We also briefly inspected raw storage:

```bash
for i in $(seq 0 10); do
  echo "slot $i:"
  cast storage $CONTRACT $i --rpc-url $RPC_URL
done
```

Slot 0 contained `0x...6b` (= 107 in decimal), indicating the string length; other slots were zero. This matches Solidity’s layout for a dynamic `string`. But we don’t actually need to parse storage manually.

### 2. Read the flag via the public getter

Since the variable is `string public flag`, the ABI of the getter is `flag()(string)`:

```bash
cast call $CONTRACT "flag()(string)" --rpc-url $RPC_URL
```

This directly returns the plaintext flag, for example:

```text
MCTF25{m4gic4i_c4n_s33_3v3ry7hing_0nch4in_im_s470shi}
```

No gas or transactions involved; it’s just a view call.

### 3. Confirm with the helper

```bash
nc 10.240.3.20 31337
> flag
# The manager prints the same flag
```

---

### 4. Flag

MCTF25{m4gic4i_c4n_s33_3v3ry7hing_0nch4in_im_s470shi}
