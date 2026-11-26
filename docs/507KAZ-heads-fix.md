# Heads that show only eyes and teeth: give the skin bake time (by 507KAZ)

**Made by 507KAZ.** Found and tested on an RTX 4070 (12 GB) with 16 GB of RAM, Series S profile
(`Play-VIctory.cmd pareto-residency`).

## What you see

Characters' heads are missing their skin: only the eyes, teeth and gums show, sometimes the mouth
moves under nothing. It happens on the first run with a fresh cache and on PCs whose texture
uploads are slow.

## Why

While a level loads, the title bakes its characters' skin and decorations into textures of its
own, with a pixel shader that picks its source textures by index. Right before each such bake
draw, the texture pass gets `VICTORY_BAKE_TEXTURES_MS` milliseconds (default 1500) to upload the
textures those shaders pick (`draw.rs` `early_budgets`). On a slower PC 1.5 s is not enough: in our
first run the bakes ran with 18,151 textures still queued. Whatever was not on the card was baked
as nothing, once, for the whole run.

## Fix

Give the bake up to 20 s:

```bat
set "VICTORY_BAKE_TEXTURES_MS=20000"
```

`Play-VIctory.cmd` now sets this by default (only if you have not set it yourself). The wait
happens only during loading and only while textures are still queued: loading can take a little
longer, and nothing changes in gameplay. With it, heads were complete in every run we made.

To go back to the old behaviour, set `VICTORY_BAKE_TEXTURES_MS=1500` before starting.

If a run then ends with `E_OUTOFMEMORY` on a 16 GB PC, see the shared-memory guard
(`VICTORY_SHARED_STAGE_PERCENT`), sent as its own pull request.
