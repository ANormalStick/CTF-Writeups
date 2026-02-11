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
Instead of all 250k traces, I used a subset (e.g., 20k–50k):

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
