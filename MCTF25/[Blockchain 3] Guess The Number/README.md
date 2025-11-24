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


# Blockchain 3 – Guess The Number

**Category:** Blockchain  
**Host:** `10.240.2.136`  
**RPC:** `http://10.240.2.136:8545`  
**TCP helper:** `10.240.2.136:31337`  

---

## Challenge Description

> Hey! I am trying to make it big with the internet money craze, do you think you can guess the number?

You are given an RPC endpoint and a TCP helper service that manages your personal challenge instance.

The Solidity contract (simplified) is:

```solidity
contract GuessTheNumber {
    bool public solved = false;
    uint256 public correctValue;

    constructor(uint256 _correctValue) {
        correctValue = _correctValue;
    }

    function submit(uint256 x) external {
        if (x == correctValue) {
            solved = true;
        }
    }

    function isSolved() external view returns (bool) {
        return solved;
    }
}
```

Key observation: `correctValue` is a **public** state variable, which means Solidity generates a public getter function `correctValue()`.  
So there is no “guessing” required – we just read the value via RPC.

---

## Exploitation Steps

### 1. Get instance info

Connect to the helper:

```bash
nc 10.240.2.136 31337
```

Then:

```text
> info
id: GuessTheNumber
rpc_port: 8545
chain_id: 31337
contract: 0xC529fE614D86C22939E20fa25D6960288B38a88A
deployer: 0xD228eE6353498A932dC83eDA84a6E0fDE302c980
```

Set environment in your shell:

```bash
export RPC_URL=http://10.240.2.136:8545
export CONTRACT=0xC529fE614D86C22939E20fa25D6960288B38a88A

cast chain-id --rpc-url $RPC_URL   # 31337
```

### 2. Read the “secret” number

```bash
cast call $CONTRACT "correctValue()" --rpc-url $RPC_URL
```

Output:

```text
0x0000000000000000000000000000000000000000000000000000000000000000
```

This is just `0`.

### 3. Create and fund a wallet

Create a keypair:

```bash
cast wallet new
# Address: 0xa8E0619A54734991D8D5884C7Ea6e97E9e175541
# Private key: 0x8576d1...
```

Export the private key (never do this on mainnet, obviously):

```bash
export PRIVATE_KEY=0x8576d146e4ef07f2e902e79c6b32b8b9d6766f5cc7780182a3c1432d265fdbb3
```

Fund the address via the helper:

```bash
nc 10.240.2.136 31337
> fund 0xa8E0619A54734991D8D5884C7Ea6e97E9e175541 1
```

(Optional) Check balance:

```bash
cast balance 0xa8E0619A54734991D8D5884C7Ea6e97E9e175541 --rpc-url $RPC_URL
```

### 4. Submit the correct number

```bash
cast send $CONTRACT "submit(uint256)" 0   --private-key $PRIVATE_KEY   --rpc-url $RPC_URL
```

Verify the challenge state:

```bash
cast call $CONTRACT "isSolved()" --rpc-url $RPC_URL
# -> true / 0x1
```

### 5. Get the flag

Back in the helper:

```bash
nc 10.240.2.136 31337
> flag
MCTF25{d4mm_y0u_c4n_s33_7h3_v4lu3}
```

---

### 6. Flag

MCTF25{d4mm_y0u_c4n_s33_7h3_v4lu3}
