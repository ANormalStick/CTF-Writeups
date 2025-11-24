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
