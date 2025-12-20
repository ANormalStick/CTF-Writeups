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

# Genie — Writeup

**Flag:** `shellmates{Weil_Pairing_and_Frobenius_u_get_torsion_group_Kernel_Sign}`

This challenge implements a CSIDH-style group action and uses it inside a toy signature scheme (“CSI-FISH”). The flag is encrypted with a key derived from the secret key list `scheme.SK`. The vulnerability is that the public key **leaks images of torsion basis points**, which is enough to recover the CSIDH exponents prime-by-prime (including their signs via Frobenius).

---

## 1. What the server does

The server constructs a CSIDH setting:

- Prime list `primes = [ℓ₀, ℓ₁, …, ℓₙ₋₁]` (here `n = 74`)
- Large prime  
  \[
  p = 4\cdot\prod_i \ell_i - 1
  \]
- Base curve  
  \[
  E_0: y^2 = x^3 + x
  \]
- Work over \(\mathbb{F}_{p^2}\), but use the Frobenius split to choose kernel “signs”.

Then it builds a *list* of 16 related secret vectors:

- `SK[0] = 0`
- `SK[1] = vec`  (short vector from lattice reduction)
- for `j = 2..15`:
  - `quick[i] = -sign(vec[i])`
  - `SK[j] = SK[j-1] + quick`

So each coordinate walks by ±1 each step.

Finally it encrypts the flag as:

```python
key = sha256(str(scheme.SK).encode()).digest()
iv_ct = iv || AES-CBC(key).encrypt(pad(flag))
```

So the whole goal is: **recover `scheme.SK`**.

---

## 2. What gets printed (the leak)

The keygen prints:

```python
Points_list = [(P.xy(), PP.xy()) for _, P, PP in self.PK]
Params = { "n": 74, "S": 16 }
Hex result = IV || ciphertext
```

Each public key entry includes **two points** `(P, PP)` on the curve corresponding to that key, where:

- \(P = \varphi(Q_1)\)
- \(PP = \varphi(Q_2)\)

and \(Q_1, Q_2\) are fixed points on \(E_0\), and \(\varphi\) is the secret isogeny chain for that key.

> In real CSIDH you would publish *only the curve* (or its class-group representative), not the images of torsion generators. Publishing torsion point images leaks enough structure to recover the secret.

---

## 3. Reconstructing each curve from two points

The server does not print curve coefficients, but it prints two points on each curve. A short Weierstrass curve is:

\[
y^2 = x^3 + a_4 x + a_6
\]

Given two points \((x_1,y_1)\), \((x_2,y_2)\), we can solve:

\[
y_1^2 - x_1^3 = a_4 x_1 + a_6
\]
\[
y_2^2 - x_2^3 = a_4 x_2 + a_6
\]

Thus:

\[
a_4 = \frac{(y_1^2 - x_1^3) - (y_2^2 - x_2^3)}{x_1 - x_2},\quad
a_6 = (y_1^2 - x_1^3) - a_4x_1
\]

This reconstructs the curve for each entry in `Points_list`, allowing us to instantiate the printed points and do group arithmetic.

Also, because `SK[0]=0`, the first entry corresponds to \(\varphi = \mathrm{id}\). Therefore:

- The curve is exactly \(E_0\)
- The printed points are exactly the original basis points \(Q_1, Q_2\)

So we recover \(Q_1, Q_2\) for free.

---

## 4. Recovering each coordinate |vec[i]| (magnitude)

Let ℓ be one of the CSIDH primes. Define cofactor:

\[
h = \frac{p+1}{\ell}
\]

For each printed key index \(j\), we have points \(P_j, PP_j\). Project them to ℓ-torsion by multiplying by \(h\):

\[
A_j = hP_j,\quad B_j = hPP_j
\]

Key fact:

- If ℓ does **not** divide \(\deg(\varphi_j)\), the map is invertible on \(E_0[\ell]\), so the images of two independent generators remain independent ⇒ \(A_j, B_j\) span 2D ℓ-torsion.
- If ℓ **does** divide \(\deg(\varphi_j)\), the ℓ-part of the map has a nontrivial kernel on ℓ-torsion, so the images collapse into a 1D subspace ⇒ \(A_j, B_j\) become dependent.

In this challenge, the 16 secret keys are related by walking each exponent toward 0:

\[
SK[j][i] = vec[i] - (j-1)\cdot \mathrm{sign}(vec[i])
\]

So there is an index:

\[
j_0 = |vec[i]| + 1
\]

where that coordinate becomes 0. At that index, ℓ no longer divides the degree contributed by that coordinate, so the ℓ-torsion action becomes invertible again.

**Algorithm (magnitude):**

For each ℓᵢ:
1. For j = 1..15:
   - compute \(A_j = hP_j\), \(B_j = hPP_j\)
   - test if \(A_j, B_j\) are independent in \(E_j[\ell]\)
2. The first j where they’re independent gives:
   \[
   |vec[i]| = j-1
   \]

**Independence test:** since ℓ is small, brute force check whether \(B_j \in \langle A_j\rangle\) by trying \(kA_j\) for k=0..ℓ-1.

---

## 5. Recovering sign(vec[i]) using Frobenius split

The CSIDH implementation chooses kernel direction using whether a kernel point’s y-coordinate lies in \(\mathbb{F}_p\):

```python
s = 1 if P[1] in self.Fp else -1
S = [i for i,e in enumerate(es) if sign(e) == s and e != 0]
```

So:
- “+” kernels correspond to points with \(y \in \mathbb{F}_p\)
- “−” kernels correspond to points with \(y \notin \mathbb{F}_p\)

When ℓ divides the degree, the projected points are dependent, giving a relation:

\[
uA_j + vB_j = 0
\]

Because \(A_j = \varphi_j(hQ_1)\) and \(B_j = \varphi_j(hQ_2)\), we get:

\[
\varphi_j(u(hQ_1) + v(hQ_2)) = 0
\]

So the kernel generator on \(E_0[\ell]\) is:

\[
K = uQ_{1,\ell} + vQ_{2,\ell},\quad Q_{1,\ell}=hQ_1,\ Q_{2,\ell}=hQ_2
\]

We can find `(u,v)` by brute forcing the scalar relation between \(A_j\) and \(B_j\) (again ℓ small). Then check:

- if \(K_y \in \mathbb{F}_p\) ⇒ sign +1
- else ⇒ sign −1

Finally:

\[
vec[i] = \mathrm{sign}(vec[i])\cdot |vec[i]|
\]

---

## 6. Rebuilding scheme.SK and decrypting

Once `vec` is known, reconstruct the full list:

- `SK[0] = 0`
- `SK[1] = vec`
- `quick[i] = -sign(vec[i])`
- `SK[j] = SK[j-1] + quick`

Then:

```python
key = sha256(str(SK).encode()).digest()
pt  = unpad(AES.new(key, AES.MODE_CBC, iv).decrypt(ct), 16)
```

This reveals the flag.

---

## 7. Full Sage reference solver

Save as `solve.sage` next to `out.txt`:

```python
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad
import hashlib

# primes from server.sage
primes = [3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47, 53, 59, 61, 67, 71,
          73, 79, 83, 89, 97, 101, 103, 107, 109, 113, 127, 131, 137, 139, 149, 151,
          157, 163, 167, 173, 179, 181, 191, 193, 197, 199, 211, 223, 227, 229, 233,
          239, 241, 251, 257, 263, 269, 271, 277, 281, 283, 293, 307, 311, 313, 317,
          331, 337, 347, 349, 353, 359, 367, 373, 587]

n = len(primes)
p = 4*prod(primes) - 1
Fp = GF(p)
F  = GF(p^2, modulus=x^2 + 1, names='i')
i  = F.gen()

# --- parse out.txt ---
lines = open("out.txt","r").read().splitlines()

def grab_block(header, next_header):
    a = lines.index(header) + 1
    b = lines.index(next_header)
    return "\\n".join(lines[a:b]).strip()

points_str = grab_block("Points_list:", "Params:")

# last block is Hex result:
hex_str = lines[lines.index("Hex result:")+1].strip()

Points_list = sage_eval(points_str, locals={"i": i})

def curve_from_points(Pxy, Qxy):
    x1,y1 = Pxy
    x2,y2 = Qxy
    a4 = ((y1^2 - x1^3) - (y2^2 - x2^3)) / (x1 - x2)
    a6 = (y1^2 - x1^3) - a4*x1
    return EllipticCurve(F, [a4, a6])

# rebuild curves + points
Es, Ps, Qs = [], [], []
for (Pxy, Qxy) in Points_list:
    E = curve_from_points(Pxy, Qxy)
    P = E(Pxy)
    Q = E(Qxy)
    Es.append(E); Ps.append(P); Qs.append(Q)

# E0 and basis points are entry 0
E0 = Es[0]
Q1 = Ps[0]
Q2 = Qs[0]

def in_Fp(y):
    return y == y^p

def dependent(A, B, ell):
    if A.is_zero() or B.is_zero():
        return True
    for k in range(ell):
        if k*A == B:
            return True
    return False

def ratio_B_eq_tA(A, B, ell):
    for t in range(ell):
        if t*A == B:
            return t
    return None

vec = [0]*n

for idx, ell in enumerate(primes):
    h = (p+1)//ell

    # magnitude
    mag = None
    for j in range(1, 16):
        Aj = h*Ps[j]
        Bj = h*Qs[j]
        if not dependent(Aj, Bj, ell):
            mag = j-1
            break
    if mag is None:
        mag = 15

    if mag == 0:
        vec[idx] = 0
        continue

    # sign from dependency at j=1
    A1 = h*Ps[1]
    B1 = h*Qs[1]

    t = ratio_B_eq_tA(A1, B1, ell)

    Q1l = h*Q1
    Q2l = h*Q2

    if t is not None:
        K = t*Q1l - Q2l
    else:
        t2 = ratio_B_eq_tA(B1, A1, ell)
        if t2 is None:
            raise ValueError("no dependency found for ell=%d" % ell)
        K = Q1l - t2*Q2l

    sgn = 1 if in_Fp(K[1]) else -1
    vec[idx] = sgn * mag

# rebuild SK list
def sgn_int(x): return 1 if x>0 else (-1 if x<0 else 0)
quick = [-sgn_int(v) for v in vec]

SK = []
SK.append([0]*n)
SK.append(vec[:])
for j in range(2, 16):
    SK.append([SK[-1][k] + quick[k] for k in range(n)])

key = hashlib.sha256(str(SK).encode()).digest()
blob = bytes.fromhex(hex_str)
iv, ct = blob[:16], blob[16:]
pt = unpad(AES.new(key, AES.MODE_CBC, iv).decrypt(ct), 16)
print(pt.decode())
```

---
