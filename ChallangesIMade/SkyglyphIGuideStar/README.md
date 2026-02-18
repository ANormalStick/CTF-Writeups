<style> :root { color-scheme: dark; --bg: #020617; --fg: #e5e7eb; --fg-muted: #9ca3af; --accent: #38bdf8; --border-subtle: #1f2937; } /* Base page */ html, body { margin: 0; padding: 0; background: radial-gradient(circle at top left, #020617, #020617 40%, #020617 100%); color: var(--fg); font-family: system-ui, -apple-system, BlinkMacSystemFont, "SF Pro Text", "Segoe UI", sans-serif; } /* GitHub Pages wrappers */ .page-content, .wrapper, article, .post { background: transparent !important; max-width: 960px; margin: 0 auto; padding: 2.5rem 1.5rem 4rem; } /* Headings */ .post h1, .page-content h1, article h1 { font-size: 1.6rem; margin-bottom: 0.6rem; } .post h2, .page-content h2, article h2 { font-size: 1.1rem; margin-top: 1.8rem; margin-bottom: 0.6rem; color: var(--fg-muted); } /* Tables (optional, if you use them) */ table { border-collapse: collapse; width: 100%; font-size: 0.85rem; margin: 0.4rem 0 0.8rem; border-radius: 0.6rem; overflow: hidden; } th, td { padding: 0.5rem 0.75rem; border-bottom: 1px solid var(--border-subtle); background-color: rgba(15, 23, 42, 0.9); } th { text-align: left; font-weight: 500; color: var(--fg-muted); } tbody tr:nth-child(even) td { background-color: rgba(15, 23, 42, 0.75); } tbody tr:last-child td { border-bottom: none; } /* Links */ a { color: var(--accent); } a:hover { text-decoration: none; } /* Code blocks */ pre, code, pre code, .highlight, .highlight pre, .highlight code { background-color: rgba(15, 23, 42, 0.96) !important; color: var(--fg); } pre { border: 1px solid var(--border-subtle); padding: 0.85rem 1rem; border-radius: 0.5rem; overflow-x: auto; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } code { padding: 0.1rem 0.25rem; border-radius: 0.25rem; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } </style>

# Skyglyph I: Guide Star - CTF Challenge

**Category:** Misc  
**Difficulty:** Easy  

## 📥 Download Challenge

**[Download Skyglyph I Guide Star.zip](https://media.githubusercontent.com/media/ANormalStick/CTF-Writeups/main/ChallangesIMade/SkyglyphIGuideStar/Skyglyph%20I%20Guide%20Star.zip)** - Try to solve it yourself before reading the writeup!

---

## Challenge Summary

This challenge is a **star-tracker calibration** puzzle. You are given `tracker_dump.csv` containing many detections:

- `x_px, y_px` — pixel coordinates of detected star-like blobs
- `flux` — brightness proxy
- Plus a few **guide stars**: rows where `name`, `ra_h`, and `dec_deg` are present

The trick is: use those guide stars to calibrate the camera, then map *all* detections back into a 2D sky plane. When plotted, the stars form a readable message/flag.

---

## 1. Use Guide Stars as Correspondences

Guide-star rows give you a correspondence between:
- a pixel coordinate `(x, y)`
- a sky coordinate `(RA, Dec)`

RA is given in **hours** — convert to degrees: `ra_deg = 15 * ra_h`, then to radians.

## 2. Project RA/Dec into a 2D Tangent Plane

Pixels are 2D. RA/Dec live on a sphere. We project RA/Dec to a local 2D tangent plane using a **gnomonic projection**.

Pick a projection center `(ra0, dec0)` — a deterministic choice is:
- `ra0` = circular mean of guide-star RA (handles RA wraparound)
- `dec0` = mean of guide-star Dec

For each guide star:

```
Δra = wrap(ra - ra0) into [-π, π]

cosc = sin(dec0)*sin(dec) + cos(dec0)*cos(dec)*cos(Δra)
u    = (cos(dec) * sin(Δra)) / cosc
v    = (cos(dec0)*sin(dec) - sin(dec0)*cos(dec)*cos(Δra)) / cosc
```

## 3. Fit a Camera Model (Plane → Pixels)

Model how the tangent plane maps to pixel coordinates:

### Radial Distortion (k1, k2)

```
r² = u² + v²
f  = 1 + k1*r² + k2*r⁴
(u_d, v_d) = (u*f, v*f)
```

### Similarity Transform

```
[x, y] = s * R(θ) * [u_d, v_d] + [tx, ty]
```

Fit parameters `p = (s, θ, tx, ty, k1, k2)` by minimizing the reprojection error on the guide-star correspondences using a robust Gauss-Newton optimizer with Huber-like weighting.

## 4. Invert the Model (Pixels → Tangent Plane)

After fitting `p`, recover plane coordinates for *all* detections:

1. Undo translation / scale / rotation to get distorted plane coords `(u_d, v_d)`
2. Invert the radial distortion by solving `r_d = r * (1 + k1*r² + k2*r⁴)` for `r` using Newton iterations

## 5. Fix Orientation Deterministically

The recovered plane can be rotated or mirrored. The challenge defines a deterministic convention:

- Rotate so **Deneb** lies on `+X`
- Flip Y if needed so **Altair** lies on `+Y`

## 6. Render the "Blue Dots" Image

Plot the recovered plane coordinates. To reduce clutter, render only the brightest detections (keep points above a flux quantile, default: top 10%).

The solver writes a **zoomable SVG**: `recovered_full.svg`

Open it in a browser and zoom until the message is clear. If it's too dense, raise the quantile:

```bash
python3 solver_skyglyph1_fullsvg.py --dir . --q 0.95
```

---

[← Back to Blog](../)
