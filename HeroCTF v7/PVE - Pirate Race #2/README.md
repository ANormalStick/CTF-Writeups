<style> :root { color-scheme: dark; --bg: #020617; --fg: #e5e7eb; --fg-muted: #9ca3af; --accent: #38bdf8; --border-subtle: #1f2937; } /* Base page */ html, body { margin: 0; padding: 0; background: radial-gradient(circle at top left, #020617, #020617 40%, #020617 100%); color: var(--fg); font-family: system-ui, -apple-system, BlinkMacSystemFont, "SF Pro Text", "Segoe UI", sans-serif; } /* GitHub Pages wrappers */ .page-content, .wrapper, article, .post { background: transparent !important; max-width: 960px; margin: 0 auto; padding: 2.5rem 1.5rem 4rem; } /* Headings */ .post h1, .page-content h1, article h1 { font-size: 1.6rem; margin-bottom: 0.6rem; } .post h2, .page-content h2, article h2 { font-size: 1.1rem; margin-top: 1.8rem; margin-bottom: 0.6rem; color: var(--fg-muted); } /* Tables (optional, if you use them) */ table { border-collapse: collapse; width: 100%; font-size: 0.85rem; margin: 0.4rem 0 0.8rem; border-radius: 0.6rem; overflow: hidden; } th, td { padding: 0.5rem 0.75rem; border-bottom: 1px solid var(--border-subtle); background-color: rgba(15, 23, 42, 0.9); } th { text-align: left; font-weight: 500; color: var(--fg-muted); } tbody tr:nth-child(even) td { background-color: rgba(15, 23, 42, 0.75); } tbody tr:last-child td { border-bottom: none; } /* Links */ a { color: var(--accent); } a:hover { text-decoration: none; } /* Code blocks */ pre, code, pre code, .highlight, .highlight pre, .highlight code { background-color: rgba(15, 23, 42, 0.96) !important; color: var(--fg); } pre { border: 1px solid var(--border-subtle); padding: 0.85rem 1rem; border-radius: 0.5rem; overflow-x: auto; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } code { padding: 0.1rem 0.25rem; border-radius: 0.25rem; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } </style>

# HeroCTF 2025 – PVE: Pirate Race #2

Category: Prog  
Difficulty: Medium  
Tags: PVE  
Author: Log_s  

---

## Challenge Overview

This is the second **Pirate Race PVE** challenge.  
You must code your own bot that controls a ship on a 2D sea and race against a more advanced predefined bot.

Each turn:

- The server sends you a JSON **game state**.
- You send back your **move** as JSON:
  - `angle` (integer, 0–359, with `0°` = North, `90°` = East, `180°` = South, `270°` = West)
  - `acceleration` (integer in `[-100, 100]`)
  - `data` (short string used as persistent memory between turns, ≤ 64 chars)

The goal is to beat the predefined bot and obtain the flag.

---

## Access & Authentication

To access the platform, you need a **CTFd access token**:

1. Go to: <https://ctf.heroctf.fr/settings#tokens>  
2. Set the token expiration date to **1st December 2025** (day after the CTF ends).  
3. Generate a token, copy it and save it somewhere safe (you cannot view it again).  
4. Use this token to authenticate to the Pirate Race platform.

Game platform:

- HTTP base URL: `http://pirate.heroctf.fr`

The exact endpoints and protocol (e.g., `/state`, `/move`) are documented on the challenge platform.

---

## Game Model

A typical (simplified) game state looks like:

```jsonc
{
  "sea_size": 1000,
  "your_ship": {
    "position": { "x": 123, "y": 456 },
    "velocity": { "x": 1.2, "y": -0.4 },
    "angle": 90
  },
  "islands": [
    {
      "position": { "x": 100, "y": 200 },
      "radius": 30,
      "type": 1,
      "validated": false
    }
  ],
  "barrels": [
    {
      "position": { "x": 500, "y": 600 },
      "collected": false
    }
  ],
  "data": "..."  // our own memory, <= 64 chars
}
```

Each turn, we respond with:

```jsonc
{
  "acceleration": 100,
  "angle": 42,
  "data": "..."
}
```

---

## Creator’s Intended Solution

The author’s writeup for Race #2 describes a **simpler yet more efficient** bot than in Race #1.

### Key Ideas

1. For each island, compute a **validation point** slightly offset from its center depending on its `type` (which side we must pass):

   ```python
   def island_to_target(x, y, t):
       match t:
           case 1: return (x+30, y)  # East
           case 2: return (x, y+30)  # South
           case 3: return (x-30, y)  # West
           case 4: return (x, y-30)  # North
   ```

2. Build a list of these target points and compute a simple **nearest-neighbor path** over them from the starting position:

   ```python
   import math

   def distance(a, b):
       return math.dist(a, b)

   def nearest_neighbor_path(points, start):
       unvisited = set(points)
       if start in unvisited:
           unvisited.remove(start)

       path = [start]
       current = start

       while unvisited:
           next_point = min(unvisited, key=lambda p: distance(current, p))
           unvisited.remove(next_point)
           path.append(next_point)
           current = next_point

       return path[1:]
   ```

3. The bot keeps track (in the `data` field) of:
   - The starting point
   - The current **target index** along this path

4. Each turn it:
   - Checks how many islands are validated; if more than the current `target_index`, it increments `target_index`.
   - Takes the corresponding point from the path, computes the angle, and always accelerates at `100`.

   ```python
   def angle_from_A_to_B(x1, y1, x2, y2):
       dx = x2 - x1
       dy = y2 - y1
       angle_rad = math.atan2(dy, dx)
       angle_deg = math.degrees(angle_rad)
       angle_deg = angle_deg + 90
       return angle_deg % 360
   ```

This solution **still ignores rhum barrels**, but it is already enough to beat the predefined advanced bot: we directly target the correct validation points in a reasonable order rather than blindly circling.

---

## My Bot Approach

I implemented a more advanced bot that:

- Prioritizes **islands** with an angle-aware nearest choice.
- Uses a better decision process for **barrel detours**.
- Stores extra information in `data`:
  - current island index
  - a small list of **banned barrels**
  - the last barrel target and a **frustration counter** to detect “bad” barrels.

Below is a condensed explanation of the logic.

### 1. Geometry Helpers

```python
import math

def distance(p1, p2):
    return math.hypot(p1[0] - p2[0], p1[1] - p2[1])

def angle_to_target(ship_pos, target_pos):
    sx, sy = ship_pos
    tx, ty = target_pos
    dx = tx - sx
    dy = ty - sy
    if dx == 0 and dy == 0:
        return 0
    ang_rad = math.atan2(dx, -dy)
    ang_deg = (math.degrees(ang_rad) + 360) % 360
    return ang_deg

def normalize_angle_diff(a, b):
    # minimal signed difference a-b in degrees, in [-180, 180]
    return (a - b + 540) % 360 - 180
```

These helpers implement the game’s direction convention (`0°` = North) and let us compute both absolute angles and relative angle differences.

---

### 2. Island Validation Points & Selection

We compute a **validation point** slightly outside the island center on the required side, clamped to the map:

```python
def island_validation_point(island, sea_size):
    ix = island["position"]["x"]
    iy = island["position"]["y"]
    r = island["radius"]
    t = island["type"]
    margin = 5
    border_margin = 5
    offset = r + margin

    if t == 1:      # East
        x = min(ix + offset, sea_size - border_margin)
        y = iy
    elif t == 3:    # West
        x = max(ix - offset, border_margin)
        y = iy
    elif t == 2:    # South
        x = ix
        y = min(iy + offset, sea_size - border_margin)
    else:           # t == 4, North
        x = ix
        y = max(iy - offset, border_margin)

    return (x, y)
```

Then we choose the next island using a **distance + angle** score:

```python
def choose_next_island(ship_pos, ship_angle, islands, sea_size):
    best_idx = None
    best_score = float("inf")
    best_dist = float("inf")

    for idx, island in enumerate(islands):
        if island.get("validated"):
            continue

        vp = island_validation_point(island, sea_size)
        d = distance(ship_pos, vp)
        ang = angle_to_target(ship_pos, vp)
        angle_diff = abs(normalize_angle_diff(ang, ship_angle))

        # Favor closer islands and those roughly in front of us
        score = d * (1.0 + angle_diff / 90.0)

        if score < best_score:
            best_score = score
            best_dist = d
            best_idx = idx

    return best_idx, best_dist
```

This makes the path smoother, since we don’t aggressively pick islands that require a huge turn unless necessary.

---

### 3. Data Format & Memory

We reuse the `data` field to store:

- The current **island index** we target.
- A small set of **banned barrels**.
- The previous barrel and a counter.

Format:

```text
I<island_idx>|Bx1,y1;x2,y2|Ppx,py,c
```

- `I<number>` → the current target island index
- `B...`     → up to 2 banned barrels, stored as `"x,y"` pairs
- `P...`     → last barrel target position `(px, py)` and `c` = the number of turns we’ve been “close” to it

Parsing and encoding functions (`parse_data`, `encode_data`) ensure the total length stays within 64 characters and drop less important fields if needed.

---

### 4. Barrel Strategy

We select the **best barrel** as the closest non-collected, non-banned barrel:

```python
def is_banned_barrel(x, y, banned, ban_dist=20.0):
    for bx, by in banned:
        if distance((x, y), (bx, by)) <= ban_dist:
            return True
    return False

def choose_best_barrel(ship_pos, barrels, banned):
    best_idx = None
    best_dist = float("inf")
    best_angle = 0.0
    for idx, barrel in enumerate(barrels):
        if barrel.get("collected"):
            continue
        bx = barrel["position"]["x"]
        by = barrel["position"]["y"]
        if is_banned_barrel(bx, by, banned):
            continue
        d = distance(ship_pos, (bx, by))
        if d < best_dist:
            best_dist = d
            best_idx = idx
            best_angle = angle_to_target(ship_pos, (bx, by))
    return best_idx, best_dist, best_angle
```

Then, when we already have an island target, we decide whether to **detour**:

- Compute:
  - `direct` = distance ship → island_target
  - `via` = distance ship → barrel + barrel → island_target
- Compare `via` vs `direct` (only accept detours if `via <= direct * max_ratio`).
- Check barrel distance and angle difference between the barrel and island directions.
- Make these thresholds depend on our **progress** in the race (early vs late).

We also keep a **frustration counter**:

- If we stay within a small distance of a barrel for several turns (e.g. `FRUSTRATION_LIMIT = 7`) and it still isn’t collected, we add it to `banned` and stop targeting it.

---

### 5. Movement & Throttle Logic

Once we have chosen `target_pos` (either an island validation point or a barrel):

1. Compute:

   ```python
   angle = angle_to_target(ship_pos, target_pos)
   dist_to_target = distance(ship_pos, target_pos)
   ```

2. Pick acceleration based on distance and speed:

   ```python
   if dist_to_target > 350:
       acceleration = 100
   elif dist_to_target > 200:
       acceleration = 100
   elif dist_to_target > 100:
       acceleration = 80
   elif dist_to_target > 50:
       acceleration = 50
   else:
       if speed > 12:
           acceleration = -20   # brake when close and fast
       else:
           acceleration = 40    # small push
   ```

3. Limit acceleration when the angle difference is large, so we don’t accelerate hard while facing the wrong way.

Finally we return the decision:

```python
return {
    "acceleration": acceleration,
    "angle": int(angle) % 360,
    "data": data_str,
}
```

---

## Integration Sketch

A typical client loop (similar to Race #1) might look like:

```python
import requests
import time

TOKEN = "YOUR_CTFD_TOKEN"
BASE = "http://pirate.heroctf.fr"

headers = {"Authorization": f"Token {TOKEN}"}

while True:
    state = requests.get(f"{BASE}/state", headers=headers).json()
    move = make_move(state)  # your bot logic from above
    requests.post(f"{BASE}/move", json=move, headers=headers)
    time.sleep(0.05)  # adjust to challenge rules
```

Check the official documentation for the exact endpoints and rate limits.

---

## Flag

Once your bot reliably beats the predefined bot, the platform reveals the flag.

For the original instance:

```text
Hero{61d477d29f70b607d1128a5da511d96c}
```

Flag format:

```regex
^Hero{\S+}$
```

---

## Notes

- The **official** creator bot for Race #2 already exploits:
  - Validation points near each island, not just the center.
  - A simple nearest-neighbor order for visiting islands.
- My solution additionally:
  - Uses angle-aware island selection.
  - Supports barrel detours with a “via vs direct” heuristic.
  - Tracks bad barrels in `data` to avoid getting stuck.

This is more than enough to beat the PVE bot and obtain the flag.
