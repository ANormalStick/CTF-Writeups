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


## [Blockchain 4] Sacred Martins Sequence

**Challenge ID:** Sacred Martins Sequence  
**Type:** Ethereum smart contract puzzle  
**RPC:** `10.240.3.250:8545` (Anvil, chainId 31337)  
**TCP service:** `10.240.3.250:31337`

### Overview

An Ethereum private chain (Anvil) hosts a single challenge contract. A separate TCP service checks on-chain state via `isSolved()` and prints a flag once the contract reports success.

### Finding the contract

Enumerate the RPC:

```bash
curl -s -X POST http://10.240.3.250:8545   -H "Content-Type: application/json"   -d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}'
```

The chain has only a couple of blocks. By walking them with `eth_getBlockByNumber` and inspecting their `transactions`, only one contract creation transaction is found, which yields the contract address, for example:

```text
0x60b4839202782c669c445b82202716b7c6c3ac36
```

### Inspecting contract storage

Check storage slots:

```bash
# slot 2
curl -s -X POST http://10.240.3.250:8545   -H "Content-Type: application/json"   -d '{
    "jsonrpc":"2.0",
    "method":"eth_getStorageAt",
    "params":["0x60b4...ac36","0x2","latest"],
    "id":2
  }'

# slot 3
curl -s -X POST http://10.240.3.250:8545   -H "Content-Type: application/json"   -d '{
    "jsonrpc":"2.0",
    "method":"eth_getStorageAt",
    "params":["0x60b4...ac36","0x3","latest"],
    "id":3
  }'
```

- Slot 2 contains a random-looking 32-byte value.  
- Slot 3, interpreted as ASCII, is the string `"abandoned"`.

### Reverse engineering

Fetch and decompile the bytecode:

```bash
curl -s -X POST http://10.240.3.250:8545   -H "Content-Type: application/json"   -d '{
    "jsonrpc":"2.0",
    "method":"eth_getCode",
    "params":["0x60b4...ac36","latest"],
    "id":4
  }'
```

Decompilation shows four relevant functions:

- `scissors()`  
  - Reads values from storage slots (including slot 1 and slot 3).  
  - If a specific condition holds, assigns `storage[1] = storage[3]`.

- `openDoor()`  
  - Checks `storage[1] == storage[3]`.  
  - If true, sets a “solved” flag in `storage[0]`.

- `isSolved()`  
  - Returns whether the low byte of `storage[0]` is non-zero.

- `dust()`  
  - Returns the string `"abandoned"` and is a red herring.

Initial state (from storage):

- `storage[1] = 0`
- `storage[3] = "abandoned"`

The puzzle is to make `storage[1] == storage[3]` and then call `openDoor()`.

### Solving on chain

Use an Anvil account (for example `0x0d1207fbdfea912219e8cd0e6ae336e841525a3e`) to send two transactions:

1. **Call `scissors()`**

   Selector (from 4byte lookup / decompile): `0x1f2a3e06`.

   ```bash
   DATA_SCISSORS="0x1f2a3e06"

   curl -s -X POST http://10.240.3.250:8545      -H "Content-Type: application/json"      -d '{
       "jsonrpc":"2.0",
       "method":"eth_sendTransaction",
       "params":[{
         "from":"0x0d1207fbdfea912219e8cd0e6ae336e841525a3e",
         "to":"0x60b4...ac36",
         "data":"'"$DATA_SCISSORS"'"
       }],
       "id":10
     }'
   ```

   After this, `storage[1]` is updated to match `storage[3]` (`"abandoned"`).

2. **Call `openDoor()`**

   Selector (from decompile): `0xdb0e127a`.

   ```bash
   DATA_OPEN="0xdb0e127a"

   curl -s -X POST http://10.240.3.250:8545      -H "Content-Type: application/json"      -d '{
       "jsonrpc":"2.0",
       "method":"eth_sendTransaction",
       "params":[{
         "from":"0x0d1207fbdfea912219e8cd0e6ae336e841525a3e",
         "to":"0x60b4...ac36",
         "data":"'"$DATA_OPEN"'"
       }],
       "id":11
     }'
   ```

   Now that `storage[1] == storage[3]`, this sets the solved flag in `storage[0]`.

### Verifying `isSolved()`

Selector from decompile: `0x64d98f6e`.

```bash
curl -s -X POST http://10.240.3.250:8545   -H "Content-Type: application/json"   -d '{
    "jsonrpc":"2.0",
    "method":"eth_call",
    "params":[{"to":"0x60b4...ac36","data":"0x64d98f6e"},"latest"],
    "id":12
  }'
```

The returned value ends in `...01`, which means `isSolved() == true`.

### Getting the flag

A separate TCP service exposes the challenge manager:

```bash
nc 10.240.3.250 31337
```

The service asks for the player address and queries the contract on chain. Once `isSolved()` is true for that address, the service prints the actual flag in the format:

```text
MCTF25{i_think_this_was_reversing_why_martins}
```

