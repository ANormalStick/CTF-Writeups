<style> :root { color-scheme: dark; --bg: #020617; --fg: #e5e7eb; --fg-muted: #9ca3af; --accent: #38bdf8; --border-subtle: #1f2937; } /* Base page */ html, body { margin: 0; padding: 0; background: radial-gradient(circle at top left, #020617, #020617 40%, #020617 100%); color: var(--fg); font-family: system-ui, -apple-system, BlinkMacSystemFont, "SF Pro Text", "Segoe UI", sans-serif; } /* GitHub Pages wrappers */ .page-content, .wrapper, article, .post { background: transparent !important; max-width: 960px; margin: 0 auto; padding: 2.5rem 1.5rem 4rem; } /* Headings */ .post h1, .page-content h1, article h1 { font-size: 1.6rem; margin-bottom: 0.6rem; } .post h2, .page-content h2, article h2 { font-size: 1.1rem; margin-top: 1.8rem; margin-bottom: 0.6rem; color: var(--fg-muted); } /* Tables (optional, if you use them) */ table { border-collapse: collapse; width: 100%; font-size: 0.85rem; margin: 0.4rem 0 0.8rem; border-radius: 0.6rem; overflow: hidden; } th, td { padding: 0.5rem 0.75rem; border-bottom: 1px solid var(--border-subtle); background-color: rgba(15, 23, 42, 0.9); } th { text-align: left; font-weight: 500; color: var(--fg-muted); } tbody tr:nth-child(even) td { background-color: rgba(15, 23, 42, 0.75); } tbody tr:last-child td { border-bottom: none; } /* Links */ a { color: var(--accent); } a:hover { text-decoration: none; } /* Code blocks */ pre, code, pre code, .highlight, .highlight pre, .highlight code { background-color: rgba(15, 23, 42, 0.96) !important; color: var(--fg); } pre { border: 1px solid var(--border-subtle); padding: 0.85rem 1rem; border-radius: 0.5rem; overflow-x: auto; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } code { padding: 0.1rem 0.25rem; border-radius: 0.25rem; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } </style>

# Symbol of Hope

## Summary

The provided `checker` binary is UPX packed. After unpacking, it reads a 42-byte input, applies a long chain of byte-wise transformations (`f_0` .. `f_4200`), and compares the result to a 42-byte constant in `.rodata`. Each function mutates exactly one byte at a fixed offset, so the whole transform is a per-byte permutation. The flag is recovered by inverting these permutations in reverse order.

## Recon and Unpacking

```
file checker
strings -a checker | head
```

The `UPX!` marker indicates packing. Unpack:

```
upx -d checker -o checker.unpacked
```

## Static Analysis

`main` logic (via `objdump -d -Mintel`):

- Reads up to 0x2e bytes with `fgets` into a stack buffer.
- Uses `strcspn` with `\"\\n\\r\"` and requires length `0x2a` (42).
- Copies 42 bytes to a second buffer and calls `f_0`.

At the end of the chain:

```
f_4200:
  memcmp(buf, expected, 0x2a)
  prints \"Yes\" or \"No\"
```

The `expected` array is a 42-byte constant in `.rodata`.

Each `f_i` (for i < 4200) does:

1. Read one byte at a fixed offset.
2. Apply a reversible byte operation (add/sub, xor, not, neg, rol/ror, nibble swap, multiply by odd constant, etc.).
3. Write the byte back.
4. Call the next function.

So the full transform is a sequence of 4200 bijective byte transforms, each touching exactly one index.

## Solve Strategy

Because each function affects only one byte, we can invert the chain one function at a time.

For each `f_i`:

- Identify the byte offset it reads and writes.
- Build the mapping `x -> y` by running the function on a buffer where that offset is set to all 0..255.
- Invert the mapping to get `y -> x`.

Apply the inverted mappings in reverse order to the 42-byte `expected` array to recover the original input.

## Solve Script (Python)

This script uses `pyelftools` and `capstone` to disassemble the functions and emulate the instruction subset used in the transforms.

```python
from elftools.elf.elffile import ELFFile
from capstone import Cs, CS_ARCH_X86, CS_MODE_64
from capstone.x86 import X86_OP_REG, X86_OP_IMM, X86_OP_MEM

REG64 = ["rax","rbx","rcx","rdx","rsi","rdi","rbp","rsp","r8","r9","r10","r11","r12","r13","r14","r15"]
REG32 = ["eax","ebx","ecx","edx","esi","edi","ebp","esp","r8d","r9d","r10d","r11d","r12d","r13d","r14d","r15d"]
REG16 = ["ax","bx","cx","dx","si","di","bp","sp","r8w","r9w","r10w","r11w","r12w","r13w","r14w","r15w"]
REG8L = ["al","bl","cl","dl","sil","dil","bpl","spl","r8b","r9b","r10b","r11b","r12b","r13b","r14b","r15b"]

alias = {}
for r in REG64:
    alias[r] = (r, 0, 64)
for r in REG32:
    base = r[:-1] if r.startswith("r") and r.endswith("d") and len(r) > 2 else "r" + r[1:]
    alias[r] = (base, 0, 32)
for r in REG16:
    base = r[:-1] if r.startswith("r") and r.endswith("w") else "r" + r
    alias[r] = (base, 0, 16)
for r in REG8L:
    base = {"al":"rax","bl":"rbx","cl":"rcx","dl":"rdx","sil":"rsi","dil":"rdi","bpl":"rbp","spl":"rsp"}.get(r, r[:-1])
    alias[r] = (base, 0, 8)

def get_reg(regs, name):
    base, shift, bits = alias[name]
    return (regs[base] >> shift) & ((1 << bits) - 1)

def set_reg(regs, name, val):
    base, shift, bits = alias[name]
    mask = (1 << bits) - 1
    val &= mask
    if bits in (32, 64) and shift == 0:
        regs[base] = val
    else:
        cur = regs[base]
        cur &= ~(mask << shift)
        cur |= (val << shift)
        regs[base] = cur

def mem_read(stack_mem, buf, addr, size):
    val = 0
    for i in range(size):
        a = addr + i
        b = buf[a] if 0 <= a < len(buf) else stack_mem.get(a, 0)
        val |= (b & 0xff) << (8 * i)
    return val

def mem_write(stack_mem, buf, addr, size, val):
    for i in range(size):
        b = (val >> (8 * i)) & 0xff
        a = addr + i
        if 0 <= a < len(buf):
            buf[a] = b
        else:
            stack_mem[a] = b

def eval_mem_addr(regs, memop, md):
    addr = memop.disp
    if memop.base:
        addr += get_reg(regs, md.reg_name(memop.base))
    if memop.index:
        addr += get_reg(regs, md.reg_name(memop.index)) * memop.scale
    return addr

def emulate(instrs, rol_addr, ror_addr, input_bytes, md):
    regs = {r: 0 for r in REG64}
    regs["rsp"] = 0x100000
    regs["rbp"] = 0x100000
    regs["rdi"] = 0
    stack_mem = {}
    buf = bytearray(input_bytes)
    read_offsets = []
    write_offsets = []
    last_write = None

    for insn in instrs:
        mnem = insn.mnemonic
        ops = insn.operands

        def read_op(op):
            if op.type == X86_OP_REG:
                return get_reg(regs, md.reg_name(op.reg))
            if op.type == X86_OP_IMM:
                return op.imm
            if op.type == X86_OP_MEM:
                a = eval_mem_addr(regs, op.mem, md)
                if 0 <= a < len(buf):
                    read_offsets.append(a)
                return mem_read(stack_mem, buf, a, op.size)
            raise NotImplementedError

        def write_op(op, val):
            nonlocal last_write
            if op.type == X86_OP_REG:
                set_reg(regs, md.reg_name(op.reg), val)
                return
            if op.type == X86_OP_MEM:
                a = eval_mem_addr(regs, op.mem, md)
                if 0 <= a < len(buf):
                    write_offsets.append(a)
                    last_write = (a, val & 0xff)
                mem_write(stack_mem, buf, a, op.size, val)
                return
            raise NotImplementedError

        if mnem in ("endbr64", "nop"):
            continue
        if mnem == "push":
            val = read_op(ops[0])
            regs["rsp"] = (regs["rsp"] - 8) & 0xffffffffffffffff
            mem_write(stack_mem, buf, regs["rsp"], 8, val)
            continue
        if mnem == "pop":
            val = mem_read(stack_mem, buf, regs["rsp"], 8)
            regs["rsp"] = (regs["rsp"] + 8) & 0xffffffffffffffff
            write_op(ops[0], val)
            continue
        if mnem in ("mov", "movzx"):
            write_op(ops[0], read_op(ops[1]))
            continue
        if mnem == "lea":
            write_op(ops[0], eval_mem_addr(regs, ops[1].mem, md))
            continue
        if mnem in ("add", "sub", "xor", "or"):
            dst = read_op(ops[0])
            src = read_op(ops[1])
            size = ops[0].size
            mask = (1 << (size * 8)) - 1
            if mnem == "add":
                res = (dst + src) & mask
            elif mnem == "sub":
                res = (dst - src) & mask
            elif mnem == "xor":
                res = (dst ^ src) & mask
            else:
                res = (dst | src) & mask
            write_op(ops[0], res)
            continue
        if mnem == "imul":
            if len(ops) == 2:
                dst = read_op(ops[0])
                src = read_op(ops[1])
                size = ops[0].size
                res = (dst * src) & ((1 << (size * 8)) - 1)
                write_op(ops[0], res)
            else:
                src1 = read_op(ops[1])
                src2 = read_op(ops[2])
                size = ops[0].size
                res = (src1 * src2) & ((1 << (size * 8)) - 1)
                write_op(ops[0], res)
            continue
        if mnem == "not":
            dst = read_op(ops[0])
            size = ops[0].size
            write_op(ops[0], (~dst) & ((1 << (size * 8)) - 1))
            continue
        if mnem == "neg":
            dst = read_op(ops[0])
            size = ops[0].size
            write_op(ops[0], (-dst) & ((1 << (size * 8)) - 1))
            continue
        if mnem in ("shl", "shr", "sar"):
            dst = read_op(ops[0])
            count = read_op(ops[1])
            size = ops[0].size
            bits = size * 8
            if mnem == "shl":
                res = (dst << (count & (bits - 1))) & ((1 << bits) - 1)
            elif mnem == "shr":
                res = (dst >> (count & (bits - 1))) & ((1 << bits) - 1)
            else:
                shift = count & (bits - 1)
                sign_bit = 1 << (bits - 1)
                val = dst - (1 << bits) if dst & sign_bit else dst
                res = (val >> shift) & ((1 << bits) - 1)
            write_op(ops[0], res)
            continue
        if mnem == "call" and ops[0].type == X86_OP_IMM:
            target = ops[0].imm
            if target in (rol_addr, ror_addr):
                val = get_reg(regs, "edi") & 0xff
                count = get_reg(regs, "esi") & 0x7
                if target == rol_addr:
                    res = ((val << count) | (val >> (8 - count))) & 0xff
                else:
                    res = ((val >> count) | (val << (8 - count))) & 0xff
                set_reg(regs, "eax", res)
            continue
        if mnem in ("leave", "ret", "jmp", "jne", "test"):
            continue
        raise NotImplementedError(f\"Unhandled {mnem} {insn.op_str}\")

    return read_offsets, write_offsets, last_write

def addr_to_offset(elf, addr):
    for sec in elf.iter_sections():
        start = sec["sh_addr"]
        end = start + sec["sh_size"]
        if start <= addr < end:
            return sec, addr - start
    return None, None

with open(\"checker.unpacked\", \"rb\") as f:
    elf = ELFFile(f)
    symtab = elf.get_section_by_name(\".symtab\")
    text = elf.get_section_by_name(\".text\")
    text_data = text.data()
    text_addr = text[\"sh_addr\"]

    md = Cs(CS_ARCH_X86, CS_MODE_64)
    md.detail = True

    syms = {s.name: s for s in symtab.iter_symbols()}
    rol_addr = syms[\"rol8\"][\"st_value\"]
    ror_addr = syms[\"ror8\"][\"st_value\"]

    funcs = []
    for s in symtab.iter_symbols():
        if s.name.startswith(\"f_\") and s[\"st_value\"] != 0:
            idx = int(s.name.split(\"_\")[1])
            funcs.append([idx, s.name, s[\"st_value\"], s[\"st_size\"]])
    funcs.sort()

    for i in range(len(funcs)):
        if funcs[i][3] == 0:
            cur = funcs[i][2]
            nxt = next((funcs[j][2] for j in range(i + 1, len(funcs)) if funcs[j][2] > cur), None)
            funcs[i][3] = (text_addr + len(text_data) - cur) if nxt is None else (nxt - cur)

    def get_bytes(addr, size):
        start = addr - text_addr
        return text_data[start:start + size]

    exp_sym = syms[\"expected\"]
    exp_sec, exp_off = addr_to_offset(elf, exp_sym[\"st_value\"])
    expected = exp_sec.data()[exp_off:exp_off + 0x2a]

    func_instrs = []
    for idx, name, addr, size in funcs:
        instrs = list(md.disasm(get_bytes(addr, size), addr))
        func_instrs.append((idx, name, instrs))

    inv_maps = []
    for idx, name, instrs in func_instrs:
        if name == \"f_4200\":
            continue
        reads, writes, _ = emulate(instrs, rol_addr, ror_addr, [0] * 42, md)
        reads = sorted(set(r for r in reads if 0 <= r < 42))
        writes = sorted(set(w for w in writes if 0 <= w < 42))
        if len(reads) != 1 or reads != writes:
            raise RuntimeError(f\"Bad IO in {name}: {reads} {writes}\")
        off = reads[0]
        mapping = [0] * 256
        buf = [0] * 42
        for x in range(256):
            buf[off] = x
            _, _, last = emulate(instrs, rol_addr, ror_addr, buf, md)
            mapping[x] = last[1]
        inv = [0] * 256
        seen = [False] * 256
        for x, y in enumerate(mapping):
            inv[y] = x
            seen[y] = True
        if not all(seen):
            raise RuntimeError(f\"Non-bijective in {name}\")
        inv_maps.append((off, inv))

    out = bytearray(expected)
    for off, inv in reversed(inv_maps):
        out[off] = inv[out[off]]

    print(out.decode(\"utf-8\"))
```

## Flag

```
uoftctf{5ymb0l1c_3x3cu710n_15_v3ry_u53ful}
```
