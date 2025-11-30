# HeroCTF 2025 – PVE: Pirate Race #1

Category: Prog  
Difficulty: Easy  
Tags: PVE  
Author: Log_s  

---

## Challenge Overview

This is the first **Pirate Race PVE** challenge.  
You must code your own bot that controls a ship on a 2D sea and race against a predefined bot.

Each turn:

- The server sends you a JSON **game state**.
- You send back your **move** as JSON:
  - `angle` (integer, 0–359)
  - `acceleration` (integer in [-100, 100])
  - `data` (short string used as persistent memory between turns)

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

- Coordinates `(x, y)` define positions on the map.
- **Angles:**
  - `0°`   = North  
  - `90°`  = East  
  - `180°` = South  
  - `270°` = West  

Each turn, we respond with:

```jsonc
{
  "acceleration": 100,
  "angle": 42,
  "data": "..."
}
```

---

## Official Bot Idea (Author's Writeup)

The author provides a reference bot that beats the first (spiral) bot.  
High-level idea:

1. **Stage 0 – Target selection**  
   - Find the **closest unvalidated island**.
   - Store its position `(tx, ty)` and radius `tr` in `data`.

2. **Stage 1 – Move towards island**  
   - Point directly at `(tx, ty)` using `_angle_from_A_to_B`.
   - Apply simple obstacle avoidance for nearby islands:
     - If an island is very close to our path, slightly adjust angle by ±15°.

3. **Stage 2 – Circle until validation**  
   - Once we’re close to the island (distance ≤ `radius + 20`), switch to circling:
     - Compute a **circling angle** around the island:
       - If too far, go towards center.
       - If close enough, move in a perpendicular direction to orbit the island.
   - When the island with coordinates `(tx, ty)` becomes `validated`, we:
     - Switch back to **Stage 0** and pick a new island.

The persistent `data` string format used is:

```text
"{stage}-{tx:04},{ty:04},{tr:02}"
```

This allows the bot to remember:

- Current stage (`0`, `1`, or `2`)
- Target island coordinates (`tx`, `ty`)
- Target island radius (`tr`)

Even though the author’s bot completely ignores **rhum barrels**, it is already sufficient to beat the initial spiral bot. Barrels are mostly introduced for the PVP version of the challenge.

---

## My Bot Approach

I implemented a slightly different bot that:

- Prioritizes **islands** (race checkpoints).
- Optionally detours to **nearby barrels** when it is efficient.
- Uses a more elaborate `data` memory format to:
  - Track **banned barrels** (barrels that wasted time).
  - Track the **previous barrel target** and how many turns we were “close” to it.

### 1. Geometry Helpers

```python
import math

def distance(p1, p2):
    return math.hypot(p1[0] - p2[0], p1[1] - p2[1])


def angle_to_target(ship_pos, target_pos):
    """Convert target vector into game angle (0° = North, 90° = East)."""
    sx, sy = ship_pos
    tx, ty = target_pos
    dx = tx - sx
    dy = ty - sy

    if dx == 0 and dy == 0:
        return 0

    # Game direction: vector(angle) = (sin(angle), -cos(angle))
    ang_rad = math.atan2(dx, -dy)
    ang_deg = (math.degrees(ang_rad) + 360) % 360
    return ang_deg


def normalize_angle_diff(a, b):
    # minimal signed difference a-b in degrees, in [-180, 180]
    return (a - b + 540) % 360 - 180
```

The key is matching the game’s angle convention (0° at North instead of East).

---

### 2. Islands: Validation Points

Each island has a `type`:

- Type **1** → must validate on the **East** side  
- Type **2** → **South** side  
- Type **3** → **West** side  
- Type **4** → **North** side  

Instead of going to the center, I compute a **validation point** slightly outside the island toward the required side:

```python
def island_validation_point(island, sea_size):
    ix = island["position"]["x"]
    iy = island["position"]["y"]
    r = island["radius"]
    t = island["type"]

    margin = 5
    border_margin = 5
    offset = r + margin  # a bit outside island radius

    if t == 1:  # East => x > ix
        x = min(ix + offset, sea_size - border_margin)
        y = iy
    elif t == 3:  # West => x < ix
        x = max(ix - offset, border_margin)
        y = iy
    elif t == 2:  # South => y > iy
        x = ix
        y = min(iy + offset, sea_size - border_margin)
    else:  # t == 4, North => y < iy
        x = ix
        y = max(iy - offset, border_margin)

    return (x, y)
```

Then I pick the closest unvalidated island by this validation point:

```python
def choose_next_island(ship_pos, islands, sea_size):
    best_idx = None
    best_dist = float("inf")

    for idx, island in enumerate(islands):
        if island.get("validated"):
            continue

        vp = island_validation_point(island, sea_size)
        d = distance(ship_pos, vp)

        if d < best_dist:
            best_dist = d
            best_idx = idx

    return best_idx, best_dist
```

---

### 3. Barrels + Memory

I use a small memory format that fits under 64 characters:

```text
"Bx1,y1;x2,y2|Ppx,py,c"
```

- `B...` → up to 3 **banned barrels** (coordinates).
- `P...` → previous barrel target `(px, py)` and `c` = how many turns we were close to it.

If we chase the same barrel for too long while being within a small distance and it still isn’t collected, we **ban** it so we never waste time on it again.

I only detour to a barrel if:

- It’s significantly closer than the next island  
  (`barrel_dist < 0.7 * island_dist`)
- And `barrel_dist < 150` in absolute terms.

---

### 4. Movement & Throttle Logic

Once a target is chosen:

1. Compute the target angle with `angle_to_target`.
2. Compute the distance to target and pick acceleration:
   - Far → strong thrust (`100`, `90`, `60`).
   - Close:
     - If speed is high → apply a bit of **negative** acceleration to brake.
     - Otherwise → use a small positive thrust.
3. If the angle difference to target is > 100°, limit acceleration (e.g., max 40) to avoid speeding in the wrong direction.

Finally, I return:

```python
return {
    "acceleration": acceleration,
    "angle": int(angle) % 360,
    "data": data_str,
}
```

---

## Integration Sketch

A typical client loop (pseudocode) looks like:

```python
import requests
import time
import math

TOKEN = "YOUR_CTFD_TOKEN"
BASE = "http://pirate.heroctf.fr"

headers = {"Authorization": f"Token {TOKEN}"}

while True:
    state = requests.get(f"{BASE}/state", headers=headers).json()
    move = make_move(state)  # your bot logic
    requests.post(f"{BASE}/move", json=move, headers=headers)
    time.sleep(0.05)  # adjust to challenge rules
```

Check the platform documentation for exact endpoints and rate limits.
