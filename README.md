# Math Striker

A vertical arcade shooter (shmup) set inside a quantum computer. The enemies are
**numbers and operators**, and your **running score is your health**: hit zero and
you are deleted.

Built with plain HTML5 Canvas and ES modules. No framework, no bundler, no build step.

## Play (phones only)

Math Striker is built for **phones**. On a computer the page just asks you to open it on
your phone. (To test on a computer, add `?desktop=1` to the address; dragging with the
mouse works like a finger.)

| Action | Gesture |
| --- | --- |
| Pick a pilot / start | Tap a pilot card |
| Move | Drag one finger anywhere (the ship follows your finger's movement) |
| Shoot | Automatic while your finger is down |
| Prime Bomb | Tap the **BOMB** button (bottom left) |
| Pause | Tap the **II** button (top right), tap anywhere to resume |
| Play again | Tap the game-over screen (tap the bottom line to change pilot) |

On Android the game goes fullscreen and locks to portrait on your first tap. On iPhone,
use Share, then **Add to Home Screen**, to remove the browser bars.

### Pilots

| | Axiom (male pilot) | Calcula (female pilot) |
| --- | --- | --- |
| Focus | Defense | Mobility and burst damage |
| Start score (health) | 150 | 100 |
| Damage taken | 65% (35% reduction) | 100% |
| Speed | 280 | 340 |
| Weapon | Single shot | Twin shot, plus a 5-way 2x-damage fan every 6th volley |
| Prime Bombs | 3 | 2 |

### Stages (45 seconds each, then they loop harder)

| Stage | Theme | What to do |
| --- | --- | --- |
| 1 | Addition (+) | Collect `+n`. Dodge `−n` numbers and `−n` shots. |
| 2 | Subtraction (−) | Enemies fire `−` operators in fans. Negative integers drain you. |
| 3 | Multiplication (×) | Grab `×2 ×3 ×5`. Avoid `0` (resets your multiplier) and `×−2 ×−3` (cost a share of your score). |
| 4 | Division (÷) and fractions | `÷2 ÷3 ÷4` and `1/2 2/3 3/4` shrink your score, your multiplier and your ship (weaker shots). |

Clearing a stage gives a score bonus and one extra Prime Bomb. After stage 4 the game
loops back to stage 1 with higher difficulty, forever.

Shapes help when colors are hard to tell apart: **circles are pickups, diamonds are enemies.**

### Power-ups (drop every 14 to 22 seconds, last 8 seconds)

| Power-up | Effect |
| --- | --- |
| `\|x\|` Absolute Value Shield | Negative numbers and `−` bullets become **positive** score bonuses. `0` is nullified, `×−k` becomes `×k`. It does *not* stop division, because division is not negative. |
| `π` Pi Magnet | Pulls every positive pickup toward you. |
| `x²` Exponent | Squares your multiplier (minimum ×2, capped at ×64). |

### Prime Bomb (`B`)

Destroys every hostile token whose value is **not prime** and clears enemy bullets.
Prime-valued enemies (2, 3, 5, 7 ...) survive and glow gold. Kills from a bomb pay out normally.
Set `bombSparesPrimeBullets: true` in `src/config.js` for the stricter reading where prime-valued bullets survive too.

## Scoring rules (in one place)

All of this lives in [`src/mathRules.js`](src/mathRules.js).

- Pickups add their value times your **multiplier**. Kill rewards are flat.
- Losses are **not** multiplied by your multiplier. They are scaled by your pilot's defense, by the stage danger (+35% per stage cleared), and a hit also takes a small percentage of your *current* score, so a huge score is a fragile one.
- The multiplier halves every 10 seconds unless you grab another `×k`.
- Division and fractions are softened on purpose: literally halving your score would be unfair, so the loss is `score × (1 − f) × 0.4`.
- Your score hitting 0 or below is game over.

## Run it locally

ES modules are blocked on `file://` URLs, so serve the folder with any static server.
Pick whichever you have installed:

```bash
# Python 3 (nothing to install)
python3 -m http.server 8000

# or Node
npm start            # runs: npx serve . -l 8000
```

Then open <http://localhost:8000/?desktop=1> on your computer, or use your computer's
network address on your phone (same Wi-Fi) for the real thing.

## Test it

```bash
npm test             # runs: node tests/smoke.mjs  (Node 18+)
```

No browser is needed. The suite checks every math rule from the design (shield,
magnet, exponent, prime bomb, zero, negative multipliers, division, stage danger,
multiplier decay) and then simulates thousands of frames of random play for both
pilots, asserting the game never crashes or enters an invalid state.

## Project layout

```
math-striker/
├── index.html            page shell and canvas
├── css/style.css         centers and scales the canvas
├── src/
│   ├── main.js           browser entry: wires everything, runs the frame loop
│   ├── game.js           game states, update loop, collisions, bomb, rendering
│   ├── mathRules.js      Math Rules Manager: how every token changes your score
│   ├── entities.js       Player, Bullet, Token, PowerUp, particles
│   ├── stages.js         data-driven stage tables (add a stage = add an object)
│   ├── spawner.js        weighted random spawning of numbers and power-ups
│   ├── config.js         every tunable number plus the two pilot definitions
│   ├── input.js          keyboard with held / just-pressed queries
│   ├── ui.js             HUD, menu, banner, pause and game-over screens
│   ├── draw.js           canvas helpers (text, rounded rects, ship drawing)
│   └── audio.js          tiny WebAudio sound effects, no asset files
└── tests/smoke.mjs       headless unit and simulation tests
```

Design rule: `mathRules.js` has no canvas or DOM code, so game rules can be tested and
tweaked without touching rendering.

## Tuning and extending

- **Balance:** edit `src/config.js` (multiplier caps, penalties, power-up length, aimed spawns) and `src/stages.js` (spawn rate, fall speed, shooter chance, token weights).
- **New stage:** add an object to `STAGES` in `src/stages.js`. If it needs a new kind of token, add a case in `Token` (entities.js), `Spawner._makeToken` and `MathRules.resolveTouch`.
- **New pilot:** add an entry to `CHARACTERS` in `src/config.js`. A new `weapon` type goes in `Player.tryFire`; a new ship shape goes in `drawShip` (draw.js).
- Open the browser console while playing and use `__game` to inspect or change state, for example `__game.rules.score = 9999`.

## Publish on GitHub Pages

1. Push this folder to a GitHub repository (the files above at the repo root).
2. Repo **Settings → Pages → Build and deployment**: Source = *Deploy from a branch*, Branch = `main`, folder = `/ (root)`.
3. After a minute the game is live at `https://<your-username>.github.io/<repo-name>/`.

All asset paths are relative, so it works under a repo subpath without changes.

## Ideas for next steps

- Boss at the end of each stage (a giant number you must factor or reduce)
- Touch controls for phones (pointer-drag to move, auto-fire)
- Gamepad support
- Music and richer sound
- A leaderboard
