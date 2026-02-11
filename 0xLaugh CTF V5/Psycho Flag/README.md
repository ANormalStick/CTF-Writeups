# Psycho Flag (Hard) ? Writeup

## Overview
We?re given a stripped x86_64 ELF named `chall`. It uses anti-debug/anti-emulation tricks (fork + ptrace + segment register writes) and self-modifies code. The flag is validated by transforming the input into a 38-byte buffer and comparing it against a 38-byte blob in `.data` using `repe cmpsb`.

The key idea: stop at the compare instruction in the child process, read the transformed input buffer and the target blob, then invert the per-byte transform to recover the original input (the flag).

## Files
- Binary: `chall`

## 1) Quick triage
```bash
# strings
python - <<'PY'
import re
from pathlib import Path
p=Path('chall').read_bytes()
for s in re.findall(rb'[ -~]{4,}', p):
    print(s.decode('ascii','ignore'))
PY
```
Notable strings: `Correct!`, `Wrong Flag.`, `Usage: ./chall <flag>`.

## 2) Self-modifying behavior (ptrace)
`strace` shows the binary uses `ptrace(PTRACE_POKETEXT, ...)` to patch code at runtime:
```bash
wsl strace -i -o /tmp/trace_i.txt /mnt/e/ctf3/chall AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
wsl grep ptrace /tmp/trace_i.txt
```
Example output:
```
ptrace(PTRACE_POKETEXT, 1007, 0x401920, 0xaaea944412e289c0) = 0
ptrace(PTRACE_POKETEXT, 1007, 0x401928, 0x3a61e2faaaaaaaaa) = 0
ptrace(PTRACE_POKETEXT, 1007, 0x40c5b8, 0x61aaea6f63c299c0) = 0
```

To simplify debugging, patch the binary with those 8-byte values and then disable ptrace syscalls (so a debugger can attach to the child cleanly).

### Patch helper (Python)
```bash
python - <<'PY'
from elftools.elf.elffile import ELFFile
from pathlib import Path
import struct

# Start from original
src=Path('chall')
patched=Path('chall_unpacked')
patched.write_bytes(src.read_bytes())

patches={
    0x401920: 0xaaea944412e289c0,
    0x401928: 0x3a61e2faaaaaaaaa,
    0x40c5b8: 0x61aaea6f63c299c0,
}

with open(patched,'rb') as f:
    elf=ELFFile(f)
    segments=[seg for seg in elf.iter_segments() if seg['p_type']=='PT_LOAD']

def vaddr_to_offset(vaddr):
    for seg in segments:
        start=seg['p_vaddr']
        end=start+seg['p_memsz']
        if start <= vaddr < end:
            return seg['p_offset'] + (vaddr - start)
    return None

b=bytearray(patched.read_bytes())
for addr,val in patches.items():
    off=vaddr_to_offset(addr)
    b[off:off+8]=struct.pack('<Q', val)

patched.write_bytes(b)
PY
```

Disable ptrace syscalls (replace `syscall` with `xor eax,eax` at the ptrace call sites):
```bash
python - <<'PY'
from elftools.elf.elffile import ELFFile
from pathlib import Path

src=Path('chall_unpacked')
dst=Path('chall_unpacked_noptrace')
dst.write_bytes(src.read_bytes())

patch_addrs=[0x40209c,0x4020f9,0x402c18,0x403ea5]  # ptrace syscalls

with open(dst,'rb') as f:
    elf=ELFFile(f)
    segments=[seg for seg in elf.iter_segments() if seg['p_type']=='PT_LOAD']

def vaddr_to_offset(vaddr):
    for seg in segments:
        start=seg['p_vaddr']
        end=start+seg['p_memsz']
        if start <= vaddr < end:
            return seg['p_offset'] + (vaddr - start)
    return None

b=bytearray(dst.read_bytes())
for addr in patch_addrs:
    off=vaddr_to_offset(addr)
    b[off:off+2]=bytes([0x31,0xC0])  # xor eax,eax

dst.write_bytes(b)
PY
```

## 3) Find the compare point
There?s a `repe cmpsb` at `0x406270` that compares the transformed input against the 38-byte blob. We break there in the child process and read the two buffers.

### GDB script
Create `gdb_cmds.txt`:
```
set pagination off
set follow-fork-mode child
set detach-on-fork off
handle SIGSTOP noprint nostop pass
b *0x406270
run AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
info reg rsi rdi rcx
x/38bx $rsi
x/38bx $rdi
quit
```
Run:
```bash
wsl gdb -q /mnt/e/ctf3/chall_unpacked_noptrace -x /mnt/e/ctf3/gdb_cmds.txt
```
At the breakpoint:
- `RSI` points to the transformed input buffer (38 bytes)
- `RDI` points to the target blob

## 4) Build the per-byte mapping
The transform is per-byte and positional-independent. For any input where all 38 bytes are the same, the output buffer becomes 38 identical bytes. We recover a mapping by feeding chunks of printable ASCII and reading the first 38 output bytes at `$rsi`.

### Mapping script
```bash
python - <<'PY'
import subprocess, re

BINARY='/mnt/e/ctf3/chall_unpacked_noptrace'

# printable ASCII excluding space and backtick
printables = ''.join(chr(i) for i in range(0x21, 0x7f) if i!=0x60)
chunks=[printables[i:i+38] for i in range(0,len(printables),38)]

mapping={}

def escape_gdb(s):
    s=s.replace('\\','\\\\')
    s=s.replace('"','\\"')
    s=s.replace('$','\\$')
    return s

for chunk in chunks:
    inp = chunk + 'A'*(38-len(chunk))
    esc = escape_gdb(inp)
    script = f"""\
set pagination off
set follow-fork-mode child
set detach-on-fork off
handle SIGSTOP noprint nostop pass
b *0x406270
set args \"{esc}\"
run
x/38bx $rsi
quit
"""
    with open('E:/ctf3/gdb_tmp.txt','w') as f:
        f.write(script)
    cmd=['wsl','gdb','-q',BINARY,'-x','/mnt/e/ctf3/gdb_tmp.txt']
    out=subprocess.check_output(cmd, text=True)
    bytes_list=[]
    for line in out.splitlines():
        if line.startswith('0x') and ':' in line:
            _, data = line.split(':',1)
            parts=re.findall(r'0x([0-9a-fA-F]{2})', data)
            for p in parts:
                bytes_list.append(int(p,16))
            if len(bytes_list)>=38:
                break
    for i,ch in enumerate(chunk):
        mapping[bytes_list[i]] = ch

blob = bytes([
    0xd9,0x91,0x0d,0xb5,0xa4,0x8e,0xc1,0x92,0x39,0x64,0x90,0x5a,0xc1,0xd9,
    0x66,0xe3,0x2d,0x68,0x8e,0x66,0xe1,0xc0,0xa5,0xca,0x8a,0x66,0xe0,0x3b,
    0x66,0x55,0xc1,0xb4,0x66,0xee,0x68,0x75,0xca,0x8c
])

out=''
for b in blob:
    out += mapping[b]

print(out)
PY
```

## 5) Flag
```
0xL4ugh{P5ych0_Flag_Hid3s_In_The_Gat3}
```

## Validation
```bash
wsl /mnt/e/ctf3/chall "0xL4ugh{P5ych0_Flag_Hid3s_In_The_Gat3}"
```
Output:
```
Correct!
```

## Notes
- The binary forks; always debug the child when the compare executes.
- The ptrace self-modification must be neutralized or applied offline to avoid anti-debug behavior.
- The transform is a 1-byte substitution (monoalphabetic), so a simple mapping is enough once the compare buffers are exposed.