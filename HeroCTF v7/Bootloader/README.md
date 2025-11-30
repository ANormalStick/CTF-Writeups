<style> :root { color-scheme: dark; --bg: #020617; --fg: #e5e7eb; --fg-muted: #9ca3af; --accent: #38bdf8; --border-subtle: #1f2937; } /* Base page */ html, body { margin: 0; padding: 0; background: radial-gradient(circle at top left, #020617, #020617 40%, #020617 100%); color: var(--fg); font-family: system-ui, -apple-system, BlinkMacSystemFont, "SF Pro Text", "Segoe UI", sans-serif; } /* GitHub Pages wrappers */ .page-content, .wrapper, article, .post { background: transparent !important; max-width: 960px; margin: 0 auto; padding: 2.5rem 1.5rem 4rem; } /* Headings */ .post h1, .page-content h1, article h1 { font-size: 1.6rem; margin-bottom: 0.6rem; } .post h2, .page-content h2, article h2 { font-size: 1.1rem; margin-top: 1.8rem; margin-bottom: 0.6rem; color: var(--fg-muted); } /* Tables (optional, if you use them) */ table { border-collapse: collapse; width: 100%; font-size: 0.85rem; margin: 0.4rem 0 0.8rem; border-radius: 0.6rem; overflow: hidden; } th, td { padding: 0.5rem 0.75rem; border-bottom: 1px solid var(--border-subtle); background-color: rgba(15, 23, 42, 0.9); } th { text-align: left; font-weight: 500; color: var(--fg-muted); } tbody tr:nth-child(even) td { background-color: rgba(15, 23, 42, 0.75); } tbody tr:last-child td { border-bottom: none; } /* Links */ a { color: var(--accent); } a:hover { text-decoration: none; } /* Code blocks */ pre, code, pre code, .highlight, .highlight pre, .highlight code { background-color: rgba(15, 23, 42, 0.96) !important; color: var(--fg); } pre { border: 1px solid var(--border-subtle); padding: 0.85rem 1rem; border-radius: 0.5rem; overflow-x: auto; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } code { padding: 0.1rem 0.25rem; border-radius: 0.25rem; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } </style>

# HeroCTF 2025 – Misc / Bootloader Writeup

> **Flag:** `Hero{SUCC3SSFULLY_3XPL0173D_B00TL04D3R}`

## Challenge Overview

- **Category:** Misc (RE / Crypto)
- **Binary:** `u-boot.bin` (custom U-Boot image)
- **Remote:**  
  - (initial) `nc dyn08.heroctf.fr 12324`  
  - (final) `nc dyn05.heroctf.fr 11076`
- **Arch:** ARMv7 32-bit, little-endian
- **Crypto:** AES-128 ECB (mentioned in the description)
- **Flag format:** `Hero{...}`

The challenge claims the bootloader uses **AES-128-ECB**, and your goal is to recover the flag that it prints.

---

## 1. Reconnaissance on `u-boot.bin`

First we look at the binary locally:

```bash
file u-boot.bin
```

We confirm it’s a U-Boot binary for ARM.

Then we search for anything related to boot prompts or secrets:

```bash
strings -n 6 u-boot.bin | grep -B1 "Press passphrase"
```

Output:

```text
bFoL^GMSThEpSPVX2LU@Zx^58M$h!nMaoRBzu7Wa
Press passphrase to stop autoboot, or wait %d seconds to start boot
```

So the **hidden passphrase** that stops autoboot is:

```text
bFoL^GMSThEpSPVX2LU@Zx^58M$h!nMaoRBzu7Wa
```

We’ll need this on the remote instance.

---

## 2. Getting a U-Boot Shell on the Remote

Connect to the remote bootloader:

```bash
nc dyn05.heroctf.fr 11076
```

You see:

```text
Press ENTER to start boot
```

Press **Enter**.

Then U-Boot starts and shows:

```text
Press passphrase to stop autoboot, or wait 5 seconds to start boot
```

Paste the passphrase:

```text
bFoL^GMSThEpSPVX2LU@Zx^58M$h!nMaoRBzu7Wa
```

Now you drop into a U-Boot shell:

```text
=>
```

Check the environment:

```text
=> printenv
```

Relevant entries:

```text
fwaddr=0x40200000
fwsize=0x00000B30
bootcmd=go 0x40200000
```

So the interesting payload (the custom bootloader code) is at:

- Address: `0x40200000`
- Size: `0xB30` bytes

And `go 0x40200000` is what U-Boot would normally execute.

---

## 3. Dumping the Bootloader Payload

We use U-Boot’s memory-dump command to read out the firmware:

```text
=> md.b 0x40200000 0xB30
```

This prints a full hex dump of the 0xB30 bytes starting at `0x40200000`.  
Among the output, near the end, we see:

```text
40200af0: 1e ff 2f e1 42 6f 6f 74 20 63 6f 6d 70 6c 65 74  ../.Boot complet
40200b00: 65 64 2c 20 68 65 72 65 20 69 73 20 79 6f 75 72  ed, here is your
40200b10: 20 66 6c 61 67 20 3a 20 00 00 00 00 30 31 32 33   flag : ....0123
40200b20: 34 35 36 37 38 39 41 42 43 44 45 46 00 00 00 00  456789ABCDEF....
```

So the firmware prints:

> **“Boot completed, here is your flag : ”**  

followed by a hex string (initially the placeholder `"0123456789ABCDEF"`).

Copy the entire `md.b` output into a local file `dump.txt`:

```bash
cd ~/bootloader

cat > dump.txt
# paste all md.b lines here
# Ctrl+D to finish
```

Convert to a binary `fw.bin`:

```bash
cat > parse_dump.py << 'EOF'
import re

data = bytearray()
with open("dump.txt") as f:
    for line in f:
        m = re.search(r':\s+((?:[0-9a-fA-F]{2}\s+)+)', line)
        if not m:
            continue
        for b in m.group(1).strip().split():
            data.append(int(b, 16))

with open("fw.bin", "wb") as out:
    out.write(data)

print("Wrote", len(data), "bytes to fw.bin")
EOF

python3 parse_dump.py
# -> Wrote 2864 bytes to fw.bin
```

2864 decimal is `0xB30`, so the dump is complete.

---

## 4. Static Analysis of `fw.bin` (ARM blob)

Load the firmware at its runtime base address (`0x40200000`) using radare2:

```bash
r2 -a arm -b 32 -m 0x40200000 fw.bin
```

Inside r2:

```r2
aaa
afl
```

Among the functions we see:

- `fcn.4020000c` – main function of the tiny bootloader
- `fcn.40200830` – context/key setup
- `fcn.40200874` – block cipher (AES-128 ECB)
- `fcn.402000cc`, `fcn.402000e4` – printing helpers
- `fcn.402009b0` – hex encoder using `"0123456789ABCDEF"`

Disassemble `fcn.4020000c`:

```r2
pdf @ fcn.4020000c
```

Important excerpt:

```asm
fcn.4020000c ():
    push {r4, r5, r6, r7, lr}
    sub  sp, sp, 0xf4

    ; 1) initialize context using data at 0x40200b34
    ldr  r1, [0x402000b8]        ; r1 = 0x40200b34
    add  r0, sp, 0x30            ; r0 = &var_30 (context)
    bl   fcn.40200830            ; init(ctx, 0x40200b34)

    ; 2) prepare buffer at [sp..sp+0x30]
    sub  r4, sp, 1
    mov  r2, 0x30
    mov  r1, 0
    mov  r0, sp
    bl   fcn.40200a68            ; fill 0x30 bytes

    ; 3) XOR-decode bytes from 0x40200b43 into [sp..]
    mov  r1, r4
    ldr  r2, [0x402000bc]        ; r2 = 0x40200b43
    add  r0, r2, 0x27            ; r0 = r2 + 0x27 (end)
loop_xor:
    ldrb r3, [r2, 1]!
    eor  r3, r3, 0x23
    cmp  r0, r2
    strb r3, [r1, 1]!
    bne  loop_xor

    ; 4) call AES on 3×16-byte blocks
    mov  r1, sp
    add  r0, sp, 0x30
    bl   fcn.40200874            ; AES(ctx, sp)

    add  r0, sp, 0x30
    add  r1, sp, 0x10
    bl   fcn.40200874            ; AES(ctx, sp+0x10)

    add  r0, sp, 0x30
    add  r1, sp, 0x20
    bl   fcn.40200874            ; AES(ctx, sp+0x20)

    ; 5) print message
    ldr  r0, =0x40200af4         ; "Boot completed, here is your flag : "
    bl   fcn.402000e4

    ; 6) hex-encode the ciphertext using "0123456789ABCDEF"
    ldr  r5, =0x40200b1c         ; "0123456789ABCDEF"
    add  r7, sp, 0x2f
hex_loop:
    ldrb r3, [sp], #1
    and  r2, r3, 0xf
    ldrb r6, [r5, r2]
    ldrb r0, [r5, r3, lsr 4]
    bl   fcn.402000cc            ; putchar(high)
    mov  r0, r6
    bl   fcn.402000cc            ; putchar(low)
    cmp  r7, r4
    bne  hex_loop

    ; 7) newline & loop forever
    ldr  r0, [0x402000c8]
    bl   fcn.402000e4

    b    .
```

Conceptually:

1. The firmware reads some constant bytes at `0x40200b43`.
2. XORs them with `0x23` into a 0x27-byte buffer.
3. Encrypts those 3 blocks with AES-128-ECB.
4. Hex-encodes the result and prints it after the “Boot completed…” string.

On the remote, running:

```text
=> go 0x40200000
```

prints:

```text
Boot completed, here is your flag : CE04188B3AA1F39921E5ABBCB0BD7531BB72...
```

That long hex string is **AES ciphertext**, not the final CTF flag.

The true flag is the XOR-decoded 0x27-byte plaintext **before** AES.

---

## 5. Dumping the Hidden Data Region (`0x40200b30` onward)

Notice that:

- `fwsize = 0xB30`, so `0x40200000 .. 0x40200b2f` was dumped into `fw.bin`.
- The code uses pointers around `0x40200b34` and `0x40200b43`, which are **just beyond** the dumped region.

So we perform another dump on the remote:

```text
=> md.b 0x40200b30 0x60
```

This gives us the extra 0x60 bytes containing:

- AES key/context data at `0x40200b34`
- XOR-obfuscated flag bytes at `0x40200b43`

We copy that into `dump2.txt` and parse it:

```bash
cd ~/bootloader

cat > dump2.txt
# paste md.b 0x40200b30 0x60 output
# Ctrl+D to finish
```

Parsing:

```bash
cat > parse_dump2.py << 'EOF'
import re

data = bytearray()
with open("dump2.txt") as f:
    for line in f:
        m = re.search(r':\s+((?:[0-9a-fA-F]{2}\s+)+)', line)
        if not m:
            continue
        for b in m.group(1).strip().split():
            data.append(int(b, 16))

with open("extra.bin", "wb") as out:
    out.write(data)

print("Wrote", len(data), "bytes to extra.bin")
EOF

python3 parse_dump2.py
# -> Wrote 96 bytes to extra.bin (0x60)
```

So `extra.bin` contains exactly the bytes from `0x40200b30` to `0x40200b8f`.

---

## 6. Recovering the Flag by Undoing the XOR

From the disassembly:

- The XOR loop starts at `0x40200040`.
- `r2` is initialized from `[0x402000bc]`, which is `0x40200b43`.
- The loop uses `ldrb r3, [r2, 1]!` and stops when `r2 == r0`, where `r0 = r2 + 0x27`.

So the obfuscated bytes live at:

- Start: `0x40200b44` (because of the pre-increment)
- Length: `0x27` bytes

`extra.bin` is the region starting at `0x40200b30`, so:

- Offset of the start inside `extra.bin`:

  ```text
  0x40200b44 - 0x40200b30 = 0x14
  ```

Now we can just undo the XOR of `0x23` in Python:

```bash
cat > recover_flag.py << 'EOF'
data = open("extra.bin", "rb").read()

start = 0x14        # offset of 0x40200b44 into extra.bin
length = 0x27
enc = data[start:start+length]

plain = bytes(b ^ 0x23 for b in enc)
print("Decoded bytes:", plain)
print("As ASCII:", plain.decode("ascii"))
EOF

python3 recover_flag.py
```

Output:

```text
Decoded bytes: b'Hero{SUCC3SSFULLY_3XPL0173D_B00TL04D3R}'
As ASCII: Hero{SUCC3SSFULLY_3XPL0173D_B00TL04D3R}
```

This is the real flag.

We never needed to actually compute the AES key or decrypt the ciphertext. The “crypto” is pure theater: the flag is present in (lightly) obfuscated form in memory before the AES step.

---

## 7. Final Flag

**Flag:**

```text
Hero{SUCC3SSFULLY_3XPL0173D_B00TL04D3R}
```

---
