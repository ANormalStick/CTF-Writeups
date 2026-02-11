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

/* Tables */
table {
  border-collapse: collapse;
  width: 100%;
  font-size: 0.85rem;
  margin: 0.4rem 0 0.8rem;
  border-radius: 0.6rem;
  overflow: hidden;
}

th, td {
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
pre, code, pre code, .highlight, .highlight pre, .highlight code {
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

# SCA2 Write-up

## Overview
Goal: recover the 16-byte key from the large `.npz` trace set and wrap it as `0xL4ugh{...}`.

The file is ~2.59?GB, so I avoided full download by using HTTP range requests to parse the ZIP and stream only the needed data.

---

## 1) Inspect the remote ZIP with range requests
Steps:
- Read the end of the file to locate the End Of Central Directory (EOCD).
- Parse EOCD to get the central directory offset/size.
- Parse the central directory entries.
- Handle Zip64 fields for large offsets.

This revealed two entries:
- `traces.npy`: shape `(250000, 1280)`, dtype `<f8`
- `plaintexts.npy`: shape `(250000, 16)`, dtype `<i8`

From the local file headers:
- `traces.npy` data offset = **188**
- `plaintexts.npy` data offset = **2560000380**

---

## 2) Stream a slice of the dataset
Instead of all 250k traces, I used a subset (e.g., 20k�50k):

```
plaintexts = np.frombuffer(buf, dtype=np.int64).reshape(N,16).astype(np.uint8)
traces     = np.frombuffer(buf, dtype=np.float64).reshape(N,1280).astype(np.float32)
```

---

## 3) First-order CPA failed
Classic CPA with the standard model:

```
HW(Sbox(P ? K))
```

did not yield a correct key (even with more traces/downsampling and other simple models). That strongly suggests a **masked / higher-order** leakage setup.

---

## 4) Second-order CPA (squared traces)
For masked DPA, a common approach is to correlate the model with **centered squared traces**:

```
T   = traces - mean(traces)
T2  = (T**2) - mean(T**2)
```

Then correlate `T2` with the standard HW S-box model:

```
hyp = HW(Sbox(P ? K))
correlate(hyp, T2)
```

This cleanly recovered the key.

---

## 5) Key and flag
Recovered key (hex):

```
4d61736b656444504131734330306c21
```

ASCII:

```
MaskedDPA1sC00l!
```

Final flag:

```
0xL4ugh{MaskedDPA1sC00l!}
```
