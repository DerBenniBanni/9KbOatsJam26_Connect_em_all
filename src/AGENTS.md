# Connections — webgame context

## Game
Single-file HTML canvas game (~2.8KB gzipped). 20 numbered circles placed randomly, player connects them in order 1→2→3→...→20 by drawing freehand lines without crossing any previous line.

## Key files
- `index.html` — the entire game

## Design decisions (per user)
- Must start dragging **from** the current circle (not anywhere)
- Connect in **numbered order** 1→2→...→20
- Crossing a line: **reject** the line (not game over) — path disappears, try again
- Shuffle button always available to regenerate layout
- Title "Connect em all!" fades on first draw, reappears on win
- Sound: `blip()` (Web Audio sine tone) only on successful connection — no draw loop sound

## Architecture (single-file)
**Canvas:** Fullscreen, HiDPI-aware (`devicePixelRatio`), dark background (#1a1a2e). Positions stored as fractions (0-1) of canvas size, converted to pixels on draw.

**State variables:**
- `cir[]` — circles {fx, fy, n}
- `paths[]` — completed connections, each is array of {x,y} pixel points
- `cur[]` — current in-progress drawing path
- `t` — current circle index (0-based), we're drawing FROM cir[t] TO cir[t+1]
- `down` — boolean, pointer is dragging
- `px,py` — last pointer position (for touchend fallback)
- `fT` — fail flash timer (frames)
- `fx[]` — ring effects {x, y, r, g, a, d, gold}
- `tA, tF` — title alpha and fade trigger
- `actx` — lazy AudioContext

**Key functions:**
- `gen()` — place 20 circles with min distance `Math.min(100, min(W,H)*.08)`, 200 retries
- `snap(tgt)` — called when pointer reaches target circle. Pushes target center to cur, calls `chk(cur)`. If crossing: fail flash, reset. If clean: push to paths, t++, effects + blip.
- `chk(pts)` — check pts against all completed paths for intersections. Skips segments sharing endpoint (<1px) to avoid false positives at circle centers.
- `si(a,b,c,d)` / `ori()` / `onSeg()` — line segment intersection via orientation test
- `start(x,y)` — shuffle button hit test, then proximity check to current circle (R+15px)
- `move(x,y)` — sample points every P=12px, auto-snap when within S=38px of target
- `end()` — fallback snap on release (S*1.5 range), cancel otherwise
- `addFx(x,y,r,g,d,gold)` — spawn expanding ring effect
- `aC()` / `blip()` — Web Audio, 520Hz ± 60Hz sine tone, 120ms, gain 0.22

**Event handlers (all on canvas):**
- Mouse: mousedown/move/up, mouseleave
- Touch: touchstart/move/end/cancel (with preventDefault, touch-action:none in CSS)

**Draw order (back to front):**
1. Clear
2. Completed paths (cyan, shadowBlur 8)
3. Active drawing (semi-transparent cyan)
4. Circles (connected=blue, current=gold glow, target=green, unvisited=gray)
5. Status counter (top-left)
6. Instruction text (bottom, only at start)
7. Shuffle button (top-right, cyan border)
8. "Crossed!" flash (red, brief)
9. Win overlay + text (semi-transparent black)
10. Ring effects (cyan for connections, gold for win)
11. Title "Connect em all!" (fades on first draw, reappears on win)

## Constants
```
T=20   circle count
R=18   circle radius (px)
S=38   snap distance (px)
D=80   min circle separation (px)
P=12   path sampling interval (px)
M=.06  margin fraction
L=3    line width
```
