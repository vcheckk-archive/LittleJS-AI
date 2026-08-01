# Changelog

Notable changes to the **littlejs** Claude Code plugin. Follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/); versions are [semantic](https://semver.org/spec/v2.0.0.html).

> **Maintainers:** the `version` field in `.claude-plugin/plugin.json` is what gates updates — the plugin cache is keyed by it, so people who already installed the plugin receive **nothing** until that number changes. Any user-visible change means bumping it and adding an entry here in the same commit.

## [Unreleased]

## [1.0.2] - 2026-08-01

### Changed

- **Scaffolded projects now mark the vendored engine as vendored.** New projects get a `.gitattributes` with `dist/** -diff linguist-vendored`. Without it the engine lands in the first commit as ~640KB of apparently-new code, and every later reviewer — human or AI — treats it as project source to read and verify. One real session spent its effort on *"let me verify the copy is byte-identical myself so the reviewer doesn't have to line-read 650KB of third-party code."* Now `git diff` reports `Binary files differ` and GitHub collapses it in pull requests. The skill also states outright that the engine must never be reviewed, diffed, or summarized.
- **Only one engine build is vendored.** Projects previously received both `littlejs.js` and `littlejs.release.js` (~640KB each) although nothing ever loaded the second. Now just `littlejs.js`, with `build.json` pointing at it — half the footprint, no functional loss. Copy `littlejs.release.js` in by hand if you want the smallest possible jam build.
- Plugin and marketplace descriptions rewritten to say what the plugin is for rather than enumerate engine trivia, and to mention Box2D physics and three.js 3D support.

### Added

- A `.gitattributes` in this repo, for the same reason: `dist/` is a vendored engine build and should not appear in review.

## [1.0.1] - 2026-08-01

### Fixed

- **`littlejs-conventions` and `new-littlejs-game` silently failed to load when the plugin was installed.** Both descriptions contained a colon followed by a space (`...mistakes are silent: drawCircle...` and `...IS the answer: scaffold...`). In YAML a plain unquoted scalar treats `: ` as the key/value indicator, so the frontmatter failed to parse and the skill was dropped — with no error anywhere. Only the two shortest descriptions, which happened to contain no embedded colon, loaded.

  This is worth knowing about because it does not reproduce under `claude --plugin-dir`, so every pre-release test passed. It surfaced only on a real `plugin install`, and the symptom is indistinguishable from a badly-worded trigger: asking for a game just produced "which tech stack do you want?"

  **If you write a skill description, keep `: ` out of it.** Use a dash or a semicolon instead.

## [1.0.0] - 2026-08-01

First release. The repo itself is the plugin: `.claude-plugin/marketplace.json` points at the repo root, so `dist/`, `templates/`, `examples/`, `reference.md`, and `build.mjs` ship as payload and the engine travels with the plugin.

### Added

- **`littlejs-conventions` skill** — portable engine rules and pitfalls, applied to any LittleJS code in any project. Carries knowledge that previously lived only in this repo's `CLAUDE.md` and could not travel: sizes are diameters, `lerp` takes percent last, `ParticleEmitter` speed is per-frame, spin is `angleVelocity`, Y is up-positive, `update()` needs no `super.update()`, and top-level consts collide with engine globals.
- **`new-littlejs-game` skill** — scaffolds a complete playable project into any empty directory, copying the engine and chosen starter out of the installed plugin. Opens in a browser straight from disk; no clone, no `npm install`, no server.
- **`littlejs-api` skill** — grep-only lookup into the bundled 900-line `reference.md`, so exact signatures are available without loading 48KB into context.
- **`atlas-shape-art` skill** — renders shape-heavy games as tinted atlas tiles instead of per-shape draw calls.
- **Standalone build mode** — `node build.mjs` with no arguments now builds the current folder when it contains a `build.json`, which is what lets a scaffolded project produce a single-file zip.
- `CHANGELOG.md` and a marketplace entry with author, homepage, repository, license, and tags.

### Fixed

- `build.mjs` still pointed at the pre-rename `games/` folder, so **every build in the repo was broken**. Now targets `examples/`, and the stale `GAMES_DIR` constant is `EXAMPLES_DIR`.
- `build.mjs` did not quote interpolated paths passed to terser and bestzip, so builds failed in any directory containing a space — which, on Windows, is most home directories.
- `.gitignore` also still referenced `games/`, so roughly 1.3MB of generated build output under `examples/` had been committed. Untracked, files kept on disk.
- `CLAUDE.md` presented `saveDataInit`, `SoundGenerator`, and `initDefaultAtlas` as engine API. They are helper wrappers in `templates/menus.js`, `templates/gameFx.js`, and `templates/textureGenerator.js`; a scaffolded project that called them would throw on startup.
- `jsconfig.json` included `./games/**/*.js`, giving editor IntelliSense no coverage of any game.

### Removed

- The `iterate-sprite` skill and its local HTTP server — an experiment that did not work out.
- `.github/copilot-instructions.md`, which described a repo layout from roughly six months earlier.
- The `GPT/` ChatGPT package, moved to its own repo at [KilledByAPixel/LittleJS-GPT](https://github.com/KilledByAPixel/LittleJS-GPT) with its history. A marketplace install copies the whole repo, so it was shipping 680KB of unrelated files to everyone installing a game-dev plugin.

[Unreleased]: https://github.com/KilledByAPixel/LittleJS-AI/compare/v1.0.2...HEAD
[1.0.2]: https://github.com/KilledByAPixel/LittleJS-AI/compare/v1.0.1...v1.0.2
[1.0.1]: https://github.com/KilledByAPixel/LittleJS-AI/compare/v1.0.0...v1.0.1
[1.0.0]: https://github.com/KilledByAPixel/LittleJS-AI/releases/tag/v1.0.0
