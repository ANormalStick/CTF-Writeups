<style> :root { color-scheme: dark; --bg: #020617; --fg: #e5e7eb; --fg-muted: #9ca3af; --accent: #38bdf8; --border-subtle: #1f2937; } /* Base page */ html, body { margin: 0; padding: 0; background: radial-gradient(circle at top left, #020617, #020617 40%, #020617 100%); color: var(--fg); font-family: system-ui, -apple-system, BlinkMacSystemFont, "SF Pro Text", "Segoe UI", sans-serif; } /* GitHub Pages wrappers */ .page-content, .wrapper, article, .post { background: transparent !important; max-width: 960px; margin: 0 auto; padding: 2.5rem 1.5rem 4rem; } /* Headings */ .post h1, .page-content h1, article h1 { font-size: 1.6rem; margin-bottom: 0.6rem; } .post h2, .page-content h2, article h2 { font-size: 1.1rem; margin-top: 1.8rem; margin-bottom: 0.6rem; color: var(--fg-muted); } /* Tables (optional, if you use them) */ table { border-collapse: collapse; width: 100%; font-size: 0.85rem; margin: 0.4rem 0 0.8rem; border-radius: 0.6rem; overflow: hidden; } th, td { padding: 0.5rem 0.75rem; border-bottom: 1px solid var(--border-subtle); background-color: rgba(15, 23, 42, 0.9); } th { text-align: left; font-weight: 500; color: var(--fg-muted); } tbody tr:nth-child(even) td { background-color: rgba(15, 23, 42, 0.75); } tbody tr:last-child td { border-bottom: none; } /* Links */ a { color: var(--accent); } a:hover { text-decoration: none; } /* Code blocks */ pre, code, pre code, .highlight, .highlight pre, .highlight code { background-color: rgba(15, 23, 42, 0.96) !important; color: var(--fg); } pre { border: 1px solid var(--border-subtle); padding: 0.85rem 1rem; border-radius: 0.5rem; overflow-x: auto; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } code { padding: 0.1rem 0.25rem; border-radius: 0.25rem; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } </style>

# Lines of Contact - CTF Challenge

**Category:** Signal Processing / Misc  
**Difficulty:** Hard  

## 📥 Download Challenge

**[Download Lines of Contact.rar](https://media.githubusercontent.com/media/ANormalStick/CTF-Writeups/main/ChallangesIMade/LinesOfContact/Lines%20of%20Contact.rar)** - Try to solve it yourself before reading the writeup!

---

## Challenge Summary

You're aboard a deep-space relay station when an incoming capture hits your buffer: a mono audio recording labeled "0xfun". The analysts insist it's alien. But the encoding style feels familiar — like something humanity would send when it wanted to be understood without sharing a language. Dig into the signal, recover what it's really carrying, and extract the flag.

This is an analog "pictures in audio" challenge (Golden Record–style raster), but you're not told that up front.

---

## 1. First Triage: "Is This Really Just Noise?"

Open `ctf_record.wav` in Audacity (or any waveform viewer) and zoom in.

There are repeating deep negative spikes at regular-ish intervals. That pattern is your first clue: real noise doesn't produce clean periodic "events" like that.

Those spikes look like a **line sync pulse**: something that marks "start of a line."

## 2. Hypothesis: It's a Raster Stream (Audio → Image)

If there's a repeating sync, the classic structure is:

```
[sync pulse][signal payload][sync pulse][signal payload]...
```

If you stack payloads line-by-line, you get an image. This is the "aha": treat the audio as scanlines.

## 3. Extract Line Starts (Sync Detection)

Take the audio samples and look at their distribution. Sync pulses are the most negative plateau in the whole file.

A robust approach:

1. Let `m = min(audio)`
2. Choose a threshold like `thr = 0.7 * m`
3. Mark runs where `audio < thr`
4. Filter by run length: compute run lengths of consecutive samples below threshold. Keep the long ones (e.g., top 10–20% or > median of the longest cluster).

You get a list of sync start indices: `starts[]`.

## 4. Estimate Line Spacing (Samples Per Line)

Compute:

```
diffs = starts[i+1] - starts[i]
```

During picture regions, diffs will cluster tightly around a nominal value (small drift is normal).

Use: `nominal_line = median(diffs)` (after ignoring huge gaps)

## 5. Find Image Boundaries

There will be big gaps in diffs between picture segments (preamble / marker / separator).

Find indices where `diffs > 3 * nominal_line` and split `starts[]` into chunks (segments). Each chunk with lots of lines is a "picture."

## 6. Decode Each Line into Pixels

For each line:

1. Take samples from `start + sync_len` to `next_start` as the "video payload"
2. Infer `sync_len` from the tight cluster of long run lengths: `sync_len = median(long_run_lengths)`
3. Slice: `payload = audio[start + sync_len : next_start]`
4. Resample each payload to a fixed width = `median(payload_length)` to handle slight time drift

## 7. Stack Lines into an Image

Convert each resampled payload into a row. Stack rows in order → 2D array. Normalize brightness robustly using percentiles (1%–99%). Save as PNG.

One segment should look like a calibration image (circle or obvious shape). The next contains the flag.

## 8. Read the Flag

Once the flag image appears, read it and submit.

---

[← Back to Blog](../)
