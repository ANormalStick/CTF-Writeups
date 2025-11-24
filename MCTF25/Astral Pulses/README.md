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

/* Tables (if you use them) */
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

/* Links / code */
a { color: var(--accent); }
a:hover { text-decoration: none; }

code, pre {
  font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;
}
</style>


# Astral Pulses

> **Flag:** `MCTF25{1_t0O_can_533_50uNdwav35}`  

---

## Challenge description

We’re given a single file:

- `output.wav` – a short “looping message from deep space”.

We’re told that flags are always in the format:

```text
MCTF25{flag}
```

So the goal is to pull some kind of data out of the audio and recover the flag.

---

## 1. First look at the audio

On the Ubuntu VM:

```bash
# Optional: install tools
sudo apt update
sudo apt install audacity -y

# Open the file in Audacity
audacity output.wav &
```

Switch **Audacity** to a **spectrogram** view:

- Click the track name → `Spectrogram` → `Spectrogram`.
- Zoom in vertically on the high-frequency region.

You should now see:

- Several **horizontal lines** at fixed frequencies.
- Each line is **dashed into dots**.
- The dots of one particular high-frequency line are very regular (a constant “tick”).
- Other lines appear only at some of the ticks.

This strongly suggests a **multi-frequency digital encoding**:

- One tone is used as a **clock / symbol sync**.
- Several other tones represent **data bits** that are either ON (1) or OFF (0) at each clock tick.

---

## 2. Understanding the encoding

From the spectrogram:

- There is **one strong constant-spacing line** (the clock).
- There are **7 other lines** that appear/disappear at those clock positions.

So each symbol:

- Lasts for one clock “tick”.
- Encodes **7 bits** (presence/absence of each of the 7 data frequencies).

That gives us values from 0–127, i.e. **7-bit ASCII**.

If you look at the very start of the file, you can see a clearly structured pattern:

- The first section looks like **binary counting** – the dots move in a regular pattern.
- That’s consistent with a **training/learning block** where the sender transmits 0–127 in order.

So the plan:

1. Detect each symbol using the clock frequency.
2. For every symbol, measure the energy at each of the 7 data frequencies.
3. Convert those 7 bits to a value 0–127.
4. Interpret as ASCII.
5. Ignore the initial training block of 128 symbols.
6. Read the remaining plaintext and extract the flag.

---

## 3. Decoding with Python

Below is a simplified version of a Python script that does the decoding.  
(You might tweak the exact frequencies & thresholds based on your measurements.)

```python
import wave
import numpy as np

FILENAME = "output.wav"

# Parameters (measured from the spectrogram)
SAMPLE_RATE = 44100
SYMBOL_DURATION = 0.1      # seconds per symbol (time between clock ticks)
N_SAMPLES_PER_SYMBOL = int(SYMBOL_DURATION * SAMPLE_RATE)

# Frequencies (one clock + 7 data tones) – approximate values from the spectrogram
CLOCK_FREQ = 11250
DATA_FREQS = [
    9000,  9300,  9600,  9900,
    10200, 10500, 10800
]

def goertzel(samples, freq, sr):
    """Energy at a specific frequency using the Goertzel algorithm."""
    n = len(samples)
    k = int(0.5 + (n * freq / sr))
    w = 2.0 * np.pi * k / n
    cos_w = np.cos(w)
    sin_w = np.sin(w)
    coeff = 2.0 * cos_w

    s_prev = 0
    s_prev2 = 0
    for x in samples:
        s = x + coeff * s_prev - s_prev2
        s_prev2 = s_prev
        s_prev = s
    power = s_prev2**2 + s_prev**2 - coeff * s_prev * s_prev2
    return power

# --- Load WAV ---
with wave.open(FILENAME, "rb") as w:
    assert w.getnchannels() == 1
    assert w.getframerate() == SAMPLE_RATE
    raw = w.readframes(w.getnframes())

# Convert 8-bit unsigned PCM to float centered at 0
data = np.frombuffer(raw, dtype=np.uint8).astype(np.float32) - 128.0

# --- Split into symbols and decode ---
symbols = len(data) // N_SAMPLES_PER_SYMBOL

values = []
for i in range(symbols):
    start = i * N_SAMPLES_PER_SYMBOL
    end = start + N_SAMPLES_PER_SYMBOL
    chunk = data[start:end] * np.hanning(N_SAMPLES_PER_SYMBOL)

    # (Optional) Check clock power to verify this is a valid symbol
    clock_power = goertzel(chunk, CLOCK_FREQ, SAMPLE_RATE)
    if clock_power < 1e6:  # adjust threshold as needed
        continue

    bits = []
    for f in DATA_FREQS:
        p = goertzel(chunk, f, SAMPLE_RATE)
        bits.append(1 if p > 1e6 else 0)   # threshold tuned by trial

    # Convert 7 bits (LSB first) to integer
    val = 0
    for idx, b in enumerate(bits):
        val |= (b << idx)
    values.append(val)

# First 128 symbols are training (0..127)
payload_vals = values[128:]

# Convert to text
text = "".join(chr(v) for v in payload_vals)
print(text)
```

Running this gives a long text starting with a warped *Lorem ipsum*-style paragraph.  
Somewhere in the middle you’ll see:

```text
... lorem ipsum bla bla MCTF25{1_t0O_can_533_50uNdwav35} ...
```

So the flag is clearly embedded as a normal ASCII substring.

---

## 4. Final flag

Extracting the `{...}` part gives us the final answer:

```text
MCTF25{1_t0O_can_533_50uNdwav35}
```

