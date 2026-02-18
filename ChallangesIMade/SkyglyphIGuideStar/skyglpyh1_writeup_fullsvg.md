# Skyglyph I: Guide Star (Stars Easy) — Writeup (SVG “blue dots” solver)

This challenge is a **star‑tracker calibration** puzzle (not blind plate-solving).

You are given `tracker_dump.csv` containing many detections:
- `x_px, y_px` — pixel coordinates of detected star-like blobs
- `flux` — brightness proxy
- plus a few **guide stars**: rows where `name`, `ra_h`, and `dec_deg` are present

The trick is: use those guide stars to calibrate the camera, then map *all* detections back into a 2D sky plane.
When plotted, the stars form a readable message/flag.

This writeup matches the provided solver: `solver_skyglyph1_fullsvg.py`.

---

## 1) Use guide stars as correspondences

Guide-star rows give you a correspondence between:
- a pixel coordinate `(x, y)`
- a sky coordinate `(RA, Dec)`

RA is given in **hours**:
- convert to degrees: `ra_deg = 15 * ra_h`
- then to radians for trig

Dec is given in degrees → radians.

---

## 2) Project RA/Dec into a 2D tangent plane

Pixels are 2D. RA/Dec live on a sphere. We project RA/Dec to a local 2D tangent plane using a **gnomonic projection**.

Pick a projection center `(ra0, dec0)`. A deterministic choice is:
- `ra0` = circular mean of guide-star RA (handles RA wraparound)
- `dec0` = mean of guide-star Dec

For each guide star:

Let `Δra = wrap(ra - ra0)` into `[-π, π]`.

Then:

```
cosc = sin(dec0)*sin(dec) + cos(dec0)*cos(dec)*cos(Δra)
u    = (cos(dec) * sin(Δra)) / cosc
v    = (cos(dec0)*sin(dec) - sin(dec0)*cos(dec)*cos(Δra)) / cosc
```

This yields plane coordinates `(u, v)` for each guide star.

---

## 3) Fit a camera model (plane → pixels)

We model how the tangent plane maps to pixel coordinates:

### 3.1 Radial distortion (k1, k2)

```
r² = u² + v²
f  = 1 + k1*r² + k2*r⁴
(u_d, v_d) = (u*f, v*f)
```

### 3.2 Similarity transform

```
[x, y] = s * R(θ) * [u_d, v_d] + [tx, ty]
```

We fit parameters:

`p = (s, θ, tx, ty, k1, k2)`

by minimizing the reprojection error on the guide-star correspondences.

The reference solver uses a small custom **robust Gauss‑Newton** optimizer with a Huber-like weighting.
(So it doesn’t require SciPy.)

---

## 4) Invert the model (pixels → tangent plane)

After fitting `p`, we recover plane coordinates for *all* detections:

1) undo translation / scale / rotation to get distorted plane coords `(u_d, v_d)`
2) invert the radial distortion by solving:

`r_d = r * (1 + k1*r² + k2*r⁴)`

for `r` using a few Newton iterations, then divide by `f(r)`.

This yields `(u, v)` for every detection.

---

## 5) Fix orientation deterministically (no guessing)

The recovered plane can be rotated or mirrored. The challenge defines a deterministic convention:

- rotate so **Deneb** lies on `+X`
- flip Y if needed so **Altair** lies on `+Y`

If either guide star is missing, the solver skips this step.

---

## 6) Render the “blue dots” image

Finally, we plot the recovered plane coordinates.

To reduce clutter, render only the brightest detections:
- keep points above a flux quantile (default: top 10%)

Instead of Matplotlib, the solver writes a **zoomable SVG**:
- `recovered_full.svg`

Open it in a browser and zoom until the message is clear.
If it’s too dense, raise the quantile:

```bash
python3 solver_skyglyph1_fullsvg.py --dir . --q 0.95
```

---

## Solver

- `solver_skyglyph1_fullsvg.py`

Run:

```bash
python3 solver_skyglyph1_fullsvg.py --dir .
```

Output:
- `recovered_full.svg`
