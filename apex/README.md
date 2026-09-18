# Apex Legends config — FOV 120 (cl_fovScale 1.7) with stable FPS

Rig: RTX 5070 Ti · Ryzen 5 7600X · 32 GB DDR5 · 49" 5120x1440

## Why FPS became unstable after the FOV change

The FOV bump itself only costs ~10-15 %. What made it *unstable* was the
existing `videoconfig.txt`:

| Setting | Was | Effect |
|---|---|---|
| `dvs_enable` = 1, GPU frametime target 9.5-9.8 ms | Adaptive Resolution ON, targeting ~100 fps | Resolution constantly scales up/down as the GPU load changes, so frametime and image sharpness both jitter. FOV 120 pushed the GPU over the target more often. |
| `mat_vsync_mode` = 2 | V-Sync (triple buffered) ON | Frame pacing is tied to the refresh rate; any dip below it becomes a stutter plus input lag. |
| `ssao_quality` = 4, `volumetric_lighting` = 1, spot shadows high, sun shadows high | Every screen-space effect at max | These scale with pixel count. 5120x1440 is 7.4 MP, nearly 4K, and FOV 120 renders more of the world per frame. |
| TSAA on | Temporal AA at 7.4 MP | Meaningful GPU cost at this resolution. |

## What changed

### `videoconfig.txt` (the real fix)
- Adaptive Resolution **off** (`dvs_enable 0`)
- V-Sync **off** (`mat_vsync_mode 0`) — use G-Sync/driver-level sync if you want tearing control
- Ambient occlusion **off**, volumetric lighting **off**
- Spot shadows **disabled**, sun shadow coverage/detail **low**
- Model detail **medium** (`r_lod_switch_scale 1`), effects detail **low**, ragdolls/gibs/impact marks **off**
- Anti-aliasing **off** (`mat_antialias_mode 0`). Put it back to `12` (TSAA) if jaggies bother you; it is the one setting here that is purely visual.
- Kept: 5120x1440 fullscreen, 16x anisotropic, texture streaming budget unchanged (16 GB VRAM has room to spare)

### `autoexec.cfg` (new)
- `fps_max 189` — a fixed cap below what the rig can hold in fights is what makes frametimes flat. Tune it (see below).
- Re-applies `cl_fovScale 1.7` so the menu cannot clamp it back to 1.55.
- Threading/preload/ragdoll cvars. Apex ignores any it no longer supports, so nothing here can break.

### `settings.cfg`
- One change: `gfx_nvnUseLowLatencyBoost "0"` → `"1"` (NVIDIA Reflex: Enabled + Boost). Keeps GPU clocks pinned so lows don't sag when load drops.

### `profile.cfg`
- Unchanged, included as a backup. FOV stays at `1.7`.

## Install

1. **videoconfig.txt** → `%USERPROFILE%\Saved Games\Respawn\Apex\local\videoconfig.txt`
   Then right-click → Properties → **Read-only**. The game rewrites this file on exit; read-only stops it undoing the fix.
2. **profile.cfg** and **settings.cfg** → `%USERPROFILE%\Saved Games\Respawn\Apex\profile\`
   Set **profile.cfg** read-only too, otherwise opening the video settings menu clamps FOV back to 1.55.
3. **autoexec.cfg** → `<Apex install folder>\cfg\autoexec.cfg`
   (Steam: `steamapps\common\Apex Legends\cfg\`, EA App: `EA Games\Apex Legends\cfg\`)
4. Launch options (Steam → Properties, or EA App → Advanced launch options):
   ```
   +exec autoexec -novid -high -forcenovsync -fullscreen
   ```
5. Do everything with the game **closed**.

## Tuning fps_max

1. Set `cl_showfps "4"` in autoexec (or type it in console) and play a couple of fights.
2. Look at the **lowest** fps you see, not the average. Set `fps_max` just under that.
3. Panel-specific caps if you use G-Sync (cap 3 below refresh so G-Sync stays engaged):
   - 240 Hz → `225`
   - 144 Hz → `141`
   - 120 Hz → `117`
4. If lows still dip below the cap: drop `r_lod_switch_scale` to `0.6` (model detail low) in videoconfig.txt. That is the last big CPU knob.

## NVIDIA control panel (per-game profile for Apex)
- Low Latency Mode: **Off** (Reflex in-game replaces it)
- Vertical sync: **Off** (or **On** only if you use G-Sync and want it to handle the ceiling)
- Power management: **Prefer maximum performance**
- Texture filtering quality: **High performance**
