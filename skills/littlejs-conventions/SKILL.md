---
name: littlejs-conventions
description: Use when writing, reviewing, or debugging ANY code that uses the LittleJS game engine (littlejsengine) — gameplay, rendering, input, physics, particles, sound, camera, or engineInit setup. Loads engine conventions and pitfalls that prevent silent bugs (sizes are diameters, particle speed is per-frame, lerp takes percent last, engine-global name collisions, no super.update). Fire this before writing the first line of LittleJS code, even for tiny changes.
---

# LittleJS engine conventions

Rules for the LittleJS engine's **global API** style: call `engineInit`, `vec2`, `drawTile`, etc. directly — no `LJS.` prefix, no ES-module imports (unless the project already uses the ESM build).

## Startup and structure

- `engineInit(gameInit, gameUpdate, gameUpdatePost, gameRender, gameRenderPost, imageSources)` starts the engine — define all five callbacks even if some are empty. `gameInit` may be `async`; the engine awaits it (put `await box2dInit()` or a three.js `await import(...)` at the top).
- Model entities as classes extending `EngineObject` — the engine updates, moves, collides, and renders them automatically every frame:

```javascript
class Player extends EngineObject
{
    constructor(pos)
    {
        super(pos, vec2(1), tile(0), 0, CYAN); // pos, size, tileInfo, angle, color
    }
    update()
    {
        this.velocity = keyDirection().scale(.2); // arrows/WASD move the player
        // engine applies physics (velocity, gravity, collision) after update()
        // no super.update() call is needed
    }
}
```

Spawn once in `gameInit` with `new Player(vec2(0))`; it draws itself — no manual draw call. Customize visuals via `this.tileInfo` / `this.color` / `this.angle`, or override `render()`.
- Persisted settings/stats: use `readSaveData`/`writeSaveData` (localStorage-backed, JSON-serialized) — don't hand-roll localStorage.

## Never shadow engine globals

Engine globals share top-level scope with game scripts. A top-level `let`/`const` named after one throws `already declared` at load: `gravity`, `time`, `frame`, `paused`, `mousePos`, `cameraPos`, `cameraScale`, math shortcuts (`sin`, `cos`, `min`, `max`, `lerp`, `clamp`, `percent`, `rand`, `randInt`, `PI`), and color constants (`RED`, `GREEN`, `BLUE`, `CYAN`, `YELLOW`, `WHITE`, `BLACK`, ...). Prefix your own globals (`playerGravity`, `gameTime`); names like `score` and `level` are fine. When unsure, grep the engine file for `let <name>`.

## Use the engine's built-ins — don't reinvent

- `keyDirection()` for ALL arrow/WASD directional input (returns a vec2) — never write manual arrow-key OR chains. `keyIsDown()` is for non-directional actions (jump, interact).
- `gamepadStick(0)` for analog movement/aim; `mousePos` (world-space), `mouseWasPressed(0)` / `mouseIsDown(0)` for the mouse.
- `isOverlapping(posA, sizeA, posB, sizeB)` for AABB hit tests; `isOnScreen()` for culling; `screenToWorld()` / `worldToScreen()` for coordinate conversion; `Timer` for timed events.
- Math: `clamp`, `lerp`, `percent`, `rand`, `randInt`, and `Vector2` methods (`add`, `scale`, `distance`, `normalize`, `rotate`, ...) — don't rewrite them.
- Sound: `Sound` / `zzfx()` (ZzFX) — never write custom WebAudio code.
- FX: `ParticleEmitter` for explosions/trails/sparkles.
- Tile-based collision: `TileCollisionLayer` and the engine's tile-collision flow; put tile reactions in `collideWithTile(tileData, pos)`. Don't build a custom tile-collision engine.
- Prefer world-space drawing (`drawTile`, `drawRect`, `drawText`, ...); most draw functions take a `screenSpace` parameter if needed.

## Pitfalls (each of these causes silent bugs)

- `drawCircle` / `drawEllipse` size is the **diameter**, not the radius.
- `lerp(valueA, valueB, percent)` — percent comes **last**, not first.
- `ParticleEmitter` speed values are **per frame**, not per second (typical range 0.1–0.5, not 10+).
- Angles: **clockwise is positive** in LittleJS (Box2D is the opposite — counterclockwise positive).
- Y-axis is **up-positive** in world space: falling gravity is negative Y.
- `drawText` is world-space (size ~3 is normal); `drawTextScreen` is pixel/screen-space (size ~80 is normal). Don't mix them up.
- Spin uses `angleVelocity` / `angleDamping` — the standard-sounding `angularVelocity` is a silent no-op.
- For additive glow, the `additiveColor` argument needs **alpha 0** (e.g. `new Color(1,1,0,0)`); non-zero alpha thickens the silhouette.
- Keep `\n` as a two-character escape inside string literals; don't convert to real line breaks.
- Don't redefine the engine's math shortcuts or color constants (see the shadowing list above).
