<style> :root { color-scheme: dark; --bg: #020617; --fg: #e5e7eb; --fg-muted: #9ca3af; --accent: #38bdf8; --border-subtle: #1f2937; } /* Base page */ html, body { margin: 0; padding: 0; background: radial-gradient(circle at top left, #020617, #020617 40%, #020617 100%); color: var(--fg); font-family: system-ui, -apple-system, BlinkMacSystemFont, "SF Pro Text", "Segoe UI", sans-serif; } /* GitHub Pages wrappers */ .page-content, .wrapper, article, .post { background: transparent !important; max-width: 960px; margin: 0 auto; padding: 2.5rem 1.5rem 4rem; } /* Headings */ .post h1, .page-content h1, article h1 { font-size: 1.6rem; margin-bottom: 0.6rem; } .post h2, .page-content h2, article h2 { font-size: 1.1rem; margin-top: 1.8rem; margin-bottom: 0.6rem; color: var(--fg-muted); } /* Tables (optional, if you use them) */ table { border-collapse: collapse; width: 100%; font-size: 0.85rem; margin: 0.4rem 0 0.8rem; border-radius: 0.6rem; overflow: hidden; } th, td { padding: 0.5rem 0.75rem; border-bottom: 1px solid var(--border-subtle); background-color: rgba(15, 23, 42, 0.9); } th { text-align: left; font-weight: 500; color: var(--fg-muted); } tbody tr:nth-child(even) td { background-color: rgba(15, 23, 42, 0.75); } tbody tr:last-child td { border-bottom: none; } /* Links */ a { color: var(--accent); } a:hover { text-decoration: none; } /* Code blocks */ pre, code, pre code, .highlight, .highlight pre, .highlight code { background-color: rgba(15, 23, 42, 0.96) !important; color: var(--fg); } pre { border: 1px solid var(--border-subtle); padding: 0.85rem 1rem; border-radius: 0.5rem; overflow-x: auto; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } code { padding: 0.1rem 0.25rem; border-radius: 0.25rem; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } </style>

# Andor – HeroCTF Crypto (Easy, 50 pts)

- **Category:** Crypto  
- **Points:** 50  
- **Author:** Alol (challenge: *Andor*)  
- **Remote:** `nc crypto.heroctf.fr 9000`  
- **Flag format:** `^Hero{\S+}$`  
- **File given:** `andor.zip`  

---

## Challenge description

> Would you rather be inside solving challenges AND getting flags OR outside touching grass?

Connecting to the service gives us two hexadecimal strings on each round:

```text
a = <hex>
o = <hex>
>
```

Sending an empty line makes the server generate a new pair `a, o` based on the same secret flag.

From the challenge source (in `andor.zip`) we learn that:

- The flag is split into two halves (byte-wise).
- One half is **AND**-ed with a random secret key.
- The other half is **OR**-ed with another random secret key.
- This process is repeated with new random keys every time we query the server.

Our job is to reconstruct the original flag bits from these noisy AND/OR views.

---

## Bitwise behaviour

Consider one bit of the flag `f` and the corresponding bit of the random key `k`.  
We observe either `f & k` or `f | k`.

The possibilities are:

| f | k | f & k | f &#124; k |
|---|---|-------|------------|
| 0 | 0 |   0   |     0      |
| 0 | 1 |   0   |     1      |
| 1 | 0 |   0   |     1      |
| 1 | 1 |   1   |     1      |

Key observations:

- For the **AND** result `a = f & k`  
  - When we see a `1`, the flag bit *must* be `1`.  
  - When we see a `0`, the flag bit could be `0` **or** `1` (we don't know if `k` was 0 or 1).

- For the **OR** result `o = f | k`  
  - When we see a `0`, the flag bit *must* be `0`.  
  - When we see a `1`, the flag bit could be `0` **or** `1`.

So each query gives us **partial** information: some bits are determined, others ambiguous.

---

## Using many queries to remove ambiguity

The server lets us ask for as many `(a, o)` pairs as we want, each time with **fresh random keys** but the **same flag**.

For each bit position:

- If at least once we see that bit of `a` equal to `1`, we know the flag bit is `1` there.
- If at least once we see that bit of `o` equal to `0`, we know the flag bit is `0` there.

To aggregate information across queries we can use:

- Bitwise **OR** across all `a` values  
  - A bit becomes `1` as soon as any query proves it must be `1`.
- Bitwise **AND** across all `o` values  
  - A bit becomes `0` as soon as any query proves it must be `0`.

Thanks to randomness, after enough iterations every bit will eventually land in a “certain” case (either witnessed as 1 in the AND stream or 0 in the OR stream). When two consecutive iterations no longer change our accumulated values, we can assume the flag is fully recovered.

---

## Exploit strategy

1. Connect to the service and read the first `a` and `o`.
2. Treat them as our current best guesses:
   - `flag_and` – half of the flag learnt from AND-ed bits.
   - `flag_or` – half of the flag learnt from OR-ed bits.
3. In a loop:
   - Request another round of `a, o`.
   - Update:
     - `flag_and = flag_and | a` (bitwise OR, byte by byte)
     - `flag_or  = flag_or  & o` (bitwise AND, byte by byte)
   - Concatenate `flag_and + flag_or` as a candidate flag.
   - Stop when the candidate flag stops changing between iterations.
4. Print the final candidate as ASCII: this is our flag.

---

## Exploit script (no comments)

```python
from pwn import *

HOST = "crypto.heroctf.fr"
PORT = 9000


def b_and(x: bytes, y: bytes) -> bytes:
    return bytes(a & b for a, b in zip(x, y))


def b_or(x: bytes, y: bytes) -> bytes:
    return bytes(a | b for a, b in zip(x, y))


def read_pair(io):
    line_a = io.recvlineS().strip()
    line_o = io.recvlineS().strip()
    a_hex = line_a.split("a = ")[-1]
    o_hex = line_o.split("o = ")[-1]
    return bytes.fromhex(a_hex), bytes.fromhex(o_hex)


def recover_flag(io, verbose=False) -> bytes:
    flag_a, flag_o = read_pair(io)
    current = flag_a + flag_o
    previous = None
    io.sendline(b"")
    while current != previous:
        previous = current
        a, o = read_pair(io)
        flag_a = b_or(flag_a, a)
        flag_o = b_and(flag_o, o)
        current = flag_a + flag_o
        if verbose:
            print(current)
        io.sendline(b"")
    return current


def main():
    import argparse

    parser = argparse.ArgumentParser(description="Exploit script for HeroCTF Andor")
    parser.add_argument(
        "--local",
        action="store_true",
        help="Run against local chall.py instead of remote service",
    )
    parser.add_argument(
        "--verbose",
        action="store_true",
        help="Print intermediate candidate flags",
    )
    args = parser.parse_args()

    if args.verbose:
        context.log_level = "debug"

    if args.local:
        io = process(["python3", "chall.py"])
    else:
        io = remote(HOST, PORT)

    flag_bytes = recover_flag(io, verbose=args.verbose)
    try:
        flag_str = flag_bytes.decode()
    except UnicodeDecodeError:
        flag_str = flag_bytes.decode(errors="replace")

    print("Recovered flag:", flag_str)


if __name__ == "__main__":
    main()
```

Running this script eventually stabilises and prints something like:

```text
Recovered flag: Hero{y0u_4nd_5l33p_0r_y0u_4nd_c0ff33_3qu4l5_fl4g_4nd_p01n75}
```
