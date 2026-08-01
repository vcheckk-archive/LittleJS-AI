---
name: new-littlejs-game
description: Use when the user wants to start a brand-new LittleJS game — "make a new game", "start a X game", "create a game", "let's build a Y", or invokes /new-littlejs-game. Scaffolds a complete playable project (engine included, opens from file://, no server), picks the right starter + feature template, and builds the smallest playable loop. Not for editing a game that already exists.
---

# new-littlejs-game

Scaffold a new playable LittleJS game by copying the closest **example game** out of this plugin, then pulling gameplay patterns from the closest **feature template**. The tables below are the answers — don't re-explore the plugin on each new game.

**Plugin root:** `${CLAUDE_SKILL_DIR}/../..` — referred to as `<plugin>` below. It contains `dist/` (engine builds), `examples/` (starter game folders), `templates/` (feature references + helper modules), and `build.mjs`. Treat `<plugin>` as **read-only**: never write into it, never scaffold inside it — it is replaced wholesale on plugin updates.

## When to invoke

- The user typed `/littlejs:new-littlejs-game` (or `/new-littlejs-game`).
- The user asks to start/create/build a NEW game ("make a card game", "let's build a platformer"). Offer it conversationally if they're clearly starting fresh.
- NOT for editing/extending an existing game — just edit it directly.

## Step 0 — Pick the scaffold mode

Two modes; check once:

- **Repo mode** — the working directory is a clone of the LittleJS-AI repo (it has `build.mjs`, `templates/`, and `dist/littlejs.js`). Scaffold into `examples/<name>/` with `../../` script paths, exactly as the repo's own games do. Do NOT copy engine/template files — the clone already has them.
- **Standalone mode** (the normal case for plugin users) — any other directory. Copy everything the game needs from `<plugin>` into the project so it is fully self-contained. If the working directory is empty (or only has dotfiles/README), scaffold directly into it; otherwise create `./<name>/` and scaffold there.

## Step 1 — Ask up to 3 quick questions (only what's needed)

1. **Game name** (camelCase, e.g. `memoryMatch`) — the folder name in repo mode, the zip/title name in standalone mode. If they don't care, propose one.
2. **Core mechanic / genre** in one line — enough to pick the template (Step 2).
3. **Does it need a title/pause menu now or later?** (decides whether to wire `menus.js` from the start.)

If the request already answers these, skip straight to Step 2 and just confirm the name.

## Step 2 — Pick the starter + template

**Two decisions:** which example game to COPY (working `index.html`/`game.js`/`build.json`), and which template to PULL PATTERNS FROM (copy code out of it into `game.js` — never base the project structure on a template).

Copy base — pick the closest **example game folder** (under `<plugin>/examples/`):

| Game uses…                  | Copy this folder | Why                                                  |
|-----------------------------|------------------|--------------------------------------------------------|
| Box2D physics               | `box2dGame/`     | Already wires `box2dInit`, the wasm loader, `data`   |
| three.js 3D rendering       | `threejsGame/`   | Already wires `ThreeJSPlugin` + CDN module import    |
| Anything else (the default) | `emptyGame/`     | Canonical non-physics starter                        |
| A simple arcade reference   | `pong/`          | Complete tiny game to read for structure             |

> These four are the only vetted starters — name them directly, don't glob for others.

Pattern source — pick the closest **template** (`<plugin>/templates/*.html`):

| Game type / need                       | Template                  | Helper modules to wire (from `<plugin>/templates/`) |
|----------------------------------------|---------------------------|-----------------------------------------------------|
| Basic shapes / text / camera (default) | `game.html`               | —                                                   |
| Box2D physics                          | `box2dGame.html`          | (engine wasm loader, not a template module)         |
| three.js 3D                            | `threejsGame.html`        | —                                                   |
| Turn-based grid / board                | `boardGame.html`          | —                                                   |
| Playing cards                          | `cardsGame.html`          | `textureGenerator.js`, `cards.js`                   |
| Title / pause / options UI             | `menuGame.html`           | `menus.js`                                          |
| Procedural sprite atlas                | `textureGame.html`        | `textureGenerator.js`                               |
| Shape/abstract/neon visuals, or many round entities | `textureGame.html` | `textureGenerator.js` — then apply the **atlas-shape-art** skill |
| Runtime tuning controls                | `tweakableGame.html`      | `tweakables.js`                                     |
| Canvas UI widgets                      | `uiGame.html`             | `menus.js` (+ `uiGame.html` patterns)               |
| Sound effects + screen shake (any game)| (read `gameFx.js` API)    | `gameFx.js`                                         |

Combine rows freely — `gameFx.js` stacks onto any other choice. Load order matters: `textureGenerator.js` before `cards.js`. Wire every helper module BOTH ways: a `<script>` tag in `index.html` AND a `sources` entry in `build.json`.

## Step 3a — Scaffold (standalone mode)

Copy from `<plugin>` into the project directory:

1. The chosen starter's files (`index.html`, `game.js`, `build.json`; `emptyGame` also has `tiles.png`) into the project ROOT (not a subfolder).
2. `dist/littlejs.js` and `dist/littlejs.release.js` → `./dist/`. For Box2D also `dist/box2d.wasm.js` + `dist/box2d.wasm.wasm`.
3. Each helper module chosen in Step 2 → `./templates/` (only the ones needed, not all of them).
4. `build.mjs` → project root.

Then fix up the copies:

5. `index.html` — retitle, and rewrite script paths (starters use repo-relative paths): `../../dist/littlejs.js` → `dist/littlejs.js`, `../../dist/box2d.wasm.js` → `dist/box2d.wasm.js`, helper modules as `templates/<file>.js`, in dependency order between the engine and `game.js`:

   ```html
   <script src=dist/littlejs.js></script>
   <script src=dist/box2d.wasm.js></script>                <!-- only for Box2D -->
   <script src=templates/textureGenerator.js></script>     <!-- cards/texture need this FIRST -->
   <script src=templates/cards.js></script>
   <script src=templates/menus.js></script>
   <script src=game.js></script>
   ```

6. `build.json` — rewrite for the flat layout. `engine` must be set explicitly (the default assumes the repo layout). Engine is auto-prepended — do NOT list `littlejs.js` in `sources`:

   ```json
   {
       "title": "Memory Match",
       "name": "memoryMatch",
       "engine": "dist/littlejs.release.js",
       "sources": ["templates/textureGenerator.js", "templates/cards.js", "game.js"],
       "data": ["tiles.png"]
   }
   ```

   For Box2D add `"data": ["dist/box2d.wasm.js", "dist/box2d.wasm.wasm"]` (data files are copied into the build by basename).

7. Generate `package.json` (build tooling only — playing the game needs none of it):

   ```json
   {
       "name": "memorymatch",
       "private": true,
       "scripts": { "build": "node build.mjs" },
       "devDependencies": { "bestzip": "^2.2.1", "terser": "^5.31.0" }
   }
   ```

The project is now self-contained: `index.html` opens and plays from `file://` immediately. Building a shippable single-file zip is optional: `npm install` once, then `npm run build` (runs `node build.mjs`, which builds the current folder when it finds `./build.json`).

## Step 3b — Scaffold (repo mode)

1. Copy the chosen starter folder to `examples/<name>/` (keep `index.html`, `game.js`, `build.json`; `emptyGame` also has `tiles.png`).
2. Update `<title>` in `index.html` and `name`/`title` in `build.json`.
3. Add helper-module `<script>` tags to `index.html` in dependency order between the engine and `game.js` — paths from `examples/<name>/` are `../../dist/littlejs.js`, `../../templates/<file>.js`.
4. Mirror them into `build.json` `sources` by the same `../../templates/...` paths, `game.js` last; engine is auto-prepended, do NOT list it. For Box2D add `"data": ["../../dist/box2d.wasm.js", "../../dist/box2d.wasm.wasm"]`.
5. Build from the repo root with `node build.mjs <name>`.

## Step 4 — Build the smallest playable loop

Write the core loop into `game.js` (split into more files — `player.js`, `ui.js`, `constants.js` — only as it grows; add each to `index.html` AND `build.json`). Pull concrete patterns out of the chosen template(s) by reading them from `<plugin>/templates/`. Follow the **littlejs-conventions** skill for engine rules (global API, engine built-ins, pitfalls).

Then stop and give the standard output: 1-3 line step summary, quick test (open `index.html` directly from `file://` — no server — with expected result + controls), and 2-4 next-step options.

## Common mistakes

- **Scaffolding into or writing to `<plugin>`** — it is read-only and wiped on update. Copy OUT of it only.
- **Basing the project on a template** (`templates/*.html`) — single-file references; copy patterns OUT of them, copy the FOLDER from an example game.
- **Copying `emptyGame` for a physics game** — copy `box2dGame` so wasm/`data` are already wired.
- **Listing `littlejs.js` in `build.json` `sources`** — the engine is auto-prepended; only list loose JS.
- **Forgetting `"engine"` in a standalone `build.json`** — the default path (`../../dist/littlejs.release.js`) only exists in the repo layout.
- **`cards.js` before `textureGenerator.js`** — load order matters.
- **Leaving `../../` paths in a standalone `index.html`** — every `src` must resolve inside the project folder.
