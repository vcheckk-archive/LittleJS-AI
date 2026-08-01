---
name: littlejs-api
description: Use when you need the exact signature, parameter order, defaults, or existence of a LittleJS engine symbol — e.g. drawTile arguments, the ParticleEmitter constructor, Timer or Sound methods, EngineObject properties, tile/TileInfo helpers. Look the symbol up instead of guessing from memory; LittleJS argument orders are easy to get subtly wrong.
---

# littlejs-api

The curated LittleJS API reference ships with this plugin at:

`${CLAUDE_SKILL_DIR}/../../reference.md`

It is ~900 lines / 48KB — do **NOT** read the whole file into context. Grep it for the symbol you need and read only the surrounding lines:

- Look up one symbol: grep for the symbol name (e.g. `ParticleEmitter`, `drawTile`, `tileCollisionTest`) and read ~20 lines around the first hit — entries are one-line signatures with a trailing `//` comment, grouped in sections.
- Broader area (e.g. "what sound functions exist?"): grep for the section keyword (`Audio`, `Input`, `Drawing`, `Particles`, `Tile Collision`) and skim that section only.
- If a symbol is NOT in the reference, verify against the engine build itself before concluding it exists: grep `${CLAUDE_SKILL_DIR}/../../dist/littlejs.js` for it. Trust the engine source over memory.

For engine *conventions and pitfalls* (argument-order traps, per-frame vs per-second units, naming rules), use the **littlejs-conventions** skill instead — this skill is only for exact API lookup.
