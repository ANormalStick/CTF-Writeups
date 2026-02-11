# Void Bound Blade — Writeup

## TL;DR
The Merkle proof verifier (`VoidboundMerkle._merkleProof`) does not enforce the required proof length.  
By choosing a **blade id with a 1-bit in the right place** and crafting a Blade whose leaf equals an
**internal node** of the relic tree, you can produce a *short* proof that still validates against the
current world root. Then bind a blade with edge = `relic0.sigil` (3e12) and one-shot the Shogun.

This writeup matches the live instance used at:
`0xe6510CC3cD4E1a2ae336c074E1d6015994e5b24f` (Sepolia)

---

## Key Bug
`VoidboundMerkle._merkleProof` accepts **any** proof length:
```solidity
for (uint256 i; i < proof.length; i++) { ... }
return root == hashed;
```
So if we can make `hashed` equal the root after fewer steps, the proof passes.

We exploit this by:
1. Making the blade leaf equal the **right half** of relic0’s Merkle leaf.
2. Choosing a blade id whose path bits align with that internal node.

That lets us “skip” most of the Merkle tree.

---

## Attack Outline
1. Get whitelisted (enter sanctum).
2. Get a Ronin and a clan.
3. Increase relic0 attunement to **129**.
4. Ensure **blade id 129** exists.
5. Use a short Merkle proof to bind blade id 129 with a massive edge.
6. Duel Shogun (one-shot).

---

## 1) Whitelist Bypass (enter sanctum)
`performKata(bytes)` forbids a direct call to `enterSanctum()` by selector, but **does not check
calldata length vs payload length**. We can call `performKata()` with a payload that **contains**
`enterSanctum()` later in calldata and still passes `shadowTorii`.

Minimal helper contract (GatePass) that calls `performKata()` with a crafted payload works.
Once `enterSanctum()` runs, `whitelist[tx.origin] = true`.

---

## 2) Setup
Call:
```
pledgeClan(0)
awakenRonin()
```

---

## 3) Raise relic0 attunement to 129
`attuneRelic()` adds `blade.tempo` to `relics[0].attunement`.

Use `mirrorRite` to call `voidAttuneBatch` in one go. For example:
- Forge 12 blades with tempo 10 and 4 blades with tempo 1 (total +124).

Then call:
```
mirrorRite(abi.encodeWithSelector(
    voidAttuneBatch.selector,
    [6..21]
))
```

Verify:
```
getRelic(0).attunement == 129
```

---

## 4) Ensure blade id 129 exists
`bindBlade` requires the blade id already exists and is owned.
So create blades until:
```
getBladeCount() == 130
```

---

## 5) Merkle proof trick
Goal: make `merkleizeBlade(blade) == b1`, the right half of relic0’s Merkle leaf.

Relic leaf structure:
```
R0 = hash(b0, b1)
b1 = hash(a2, a3)
a2 = hash(hash(attunement), hash(sigil))
a3 = hash(hash(isSealed), hash(isSealed))
```

To match this with a Blade:
```
Blade.id      = attunement = 129
Blade.edge    = relic0.sigil (3_000_000_000_000)
Blade.tempo   = 1
Blade.roninId = 1   // because bool true hashes same as uint256(1)
```

So:
```
merkleizeBlade(Blade) == b1
```

Now choose **blade id 129**.  
Its Merkle path bits match the internal-node shortcut, allowing a short proof.

---

## 6) Proof generation
The proof depends on **current on-chain state** (blades_root changes as you forge).
Use the provided script:

```
python E:\ctf\compute_proof.py
```

It prints:
- the 9 proof hashes
- the crafted Blade struct

---

## 7) bindBlade + duel
Bind:
```
bindBlade(
  [129, 3000000000000, 1, 1],
  [<9 hashes from script>]
)
```

Then:
```
duelShogun()
```

Damage = `edge * tempo + level` = 3e12, so the Shogun dies instantly.

---

## Files
- `E:\ctf\compute_proof.py` — generates the correct proof for the current state.

---

## Notes
- The proof hashes **must** be regenerated if the sanctum state changes
  (any blade or relic updates change the root).
- If gas estimation fails for `mirrorRite`, raise gas limit (~1,000,000).

