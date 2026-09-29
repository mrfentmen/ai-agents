# AI Agents — Build List 2: The Crew's Tool Arsenal

16 MCP servers for the FENTMEN: NYC NIGHTS crew. Each spec is self-contained:
one agent builds one section. 8 agents × 2 tools each. An orchestrator agent
assigns sections and collects finished repos.

Target game: Godot 4.3, gritty PS1/PS2-era low-poly, no cute assets (Kenney
ban stands). All tools serve the lanes: del = player/weapons,
milo = enemies/ragdoll, Hana = arena/assets, pax = repo/QA.

## EXISTING TOOLS — wire these into the gateway NOW (2026-09-29 survey)

Do NOT rebuild these. Clone, wire into the MCP gateway, hand to the lane
that needs them. Verified to exist via web search 2026-09-29.

### Engine drivers (pick ONE per engine, don't wire five)

| Server | Engine | Notes | Lane |
|---|---|---|---|
| `erodenn/godot-mcp-runtime` | Godot 4 | Headless edit, screenshots, input sim, runtime control. Best for us. | pax |
| `chibuikeod/gesso-mcp-server` | Godot 4 | Runtime debug, screenshots, input emulation, asset search (itch/Kenney/OpenGameArt). | pax |
| `ivanmurzak/unity-mcp` | Unity | Shared GameDev-MCP-Server backend, active. | — |
| UE 5.8 built-in MCP | Unreal | **Epic ships it natively** — enable the plugin, no third party needed. | — |
| `alarukai/unreal-mcp` | Unreal | 127 tools, no C++ plugin required (uses built-in Python). | — |
| `amyjeanes/gmod-mcp-server` | GMod | Run Lua in a live GMod session, file-based IPC. | — |
| `roblox-studio-mcp` | Roblox | Drives Studio (no Linux headless — testing gap unfillable). | — |
| `youichi-uda/renpy-mcp-pro-public` | Ren'Py | Visual novels only. Not us, listed for completeness. | — |

### 3D / Blender (pick ONE)

| Server | Notes | Lane |
|---|---|---|
| `carlosh7/blender-mcp` | 239 tools, headless-ready, asset integrations (PolyHaven, Sketchfab, AmbientCG). Strongest. | Hana |
| `rfingadam/mcp-blender` | 218 tools, Blender 4.2/5.0, render-analyze-refine loop. | Hana |
| `ahujasid/blender-mcp` | Original, smaller, Sketchfab + Poly Haven search. | Hana |

### Asset search & download

| Server | Sources | Lane |
|---|---|---|
| `evonar543/assetmcp` | itch.io, OpenGameArt, ambientCG, Kenney, Openverse. License metadata. Blender inspect/render. | Hana |
| `jonit-dev/threenative-asset-mcp` | Fab, Poly Haven, ambientCG, Sketchfab + game-audio catalog (Sonniss, Mixkit, Freesound...). | Hana |
| Meshy MCPs (`pasie15/meshy-ai-mcp-server` etc.) | Text/image-to-3D, auto-rig, 500+ animations. **Needs Meshy API key (paid).** | Hana |
| three.ws `rig_mesh` | Free auto-rig, no key, humanoids. Use until a key exists. | Hana |

### Still MISSING — confirmed gaps, build these (not covered above)

- **game-director** (list 1) — the conductor. Nothing like it exists.
- **lua-port** (list 1) — GLua→Luau/GDScript. Nothing exists.
- **gore-kit** (list 1) — dismemberment/ragdoll done right. Nothing exists.
- **psx-ify / texture-baker** (list 2) — PBR→PS1 conversion. Nothing exists.
- **ragdoll-tuner** (list 2) — auto-tune + drop-test ragdolls. Nothing exists.
- **game-qa-harness** (list 2, #11+#12) — scripted playtests with assertions. Driver primitives exist, no test runner.
- **mixamo-link** — browse/download Mixamo characters+animations as MCP. Doesn't exist (needs Adobe auth).
- **music-director** (list 2) — royalty-free stand-in finder. Doesn't exist.
- **ldtk-builder** (list 2) — LDtk→Godot scenes. Doesn't exist as MCP.

## Shared conventions (all agents)

- MCP over stdio. Python preferred, Node acceptable.
- Each tool = its own repo under `Mrfentmen/`, public, with README + LICENSE.
- When done: add to `Mrfentmen/ai-agents` list and the awesome-mcps catalog.
- No API keys in code. Keys go in env vars.
- Every tool must have a `--self-test` that proves it works without a human.

---

## AGENT 1 — Asset pipeline

### 1. asset-fetcher (MCP)
Search + download game assets from many sources in one call.
- `search_assets(query, sources, max_poly, license)` — sources: sketchfab,
  itch.io, poly_pizza, cgtrader_free, opengameart, quaternius. Returns:
  name, url, license, poly count, formats, rigged y/n.
- `download_asset(url, dest)` — downloads, converts to GLB via headless
  Blender if needed. Returns local path + a license receipt (name, author,
  license, url) the game ships in its credits file.
- Never return an asset without a verified license string.

### 2. texture-baker (MCP)
PSX-ify visuals.
- `bake_texture(input, palette_size, max_res)` — downscale, palette crush,
  dither. Returns baked PNG + preview.
- `snap_vertices(glb, grid)` — vertex snapping for that wobbly PS1 look.
- `affine_kit(glb)` — strip perspective-correct UVs where the engine allows.

## AGENT 2 — Character pipeline

### 3. rig-check (MCP)
Inspect a character model and give a go/no-go.
- `inspect(glb)` — returns: bone count, bone names, Mixamo-compatible y/n,
  skin weights present y/n, animations included, poly count, T-pose y/n.
- `verdict(glb)` — READY (import as-is), NEEDS_RIG, NEEDS_SKIN, or REJECT
  with reasons. No guessing — every claim cites the inspected data.

### 4. retarget-kit (MCP)
Put Mixamo animations on any humanoid rig.
- `bone_map(rig_glb)` — show the rig's bones mapped to Mixamo standard.
- `retarget(animation_fbx, rig_glb)` — returns animated GLB on the target
  rig. Reports unmapped bones instead of silently dropping them.

## AGENT 3 — Ragdoll & gore (milo's tools)

### 5. ragdoll-tuner (MCP)
The anti-month-of-pain machine.
- `setup_ragdoll(skeleton)` — generate PhysicalBone3D nodes with CONE
  joints (never PIN), hand-sized collision shapes, tuned damping/mass.
- `drop_test(scene)` — headless: spawn ragdoll, drop from 2m, simulate 5s.
  Returns stability score + notes any explosion or joint failure.
- `tune(scene, target)` — iterate joint params until drop_test passes.

### 6. gib-planner (MCP)
- `cut_points(rig_glb)` — propose dismemberment joints (shoulder, elbow,
  hip, knee, neck) with the bone names from rig-check.
- `gib_config(rig_glb, cut)` — generate the GibArchetype resource: which
  mesh chunk detaches, blood particle hook, impulse transfer values.

## AGENT 4 — Weapons & player (del's tools)

### 7. weapon-tuner (MCP)
- `read_stats(weapon_tres)` / `write_stats(weapon_tres, stats)` — edit
  damage, spread, recoil, fire rate on the game's WeaponData resources.
- `simulate(weapon_tres)` — DPS, time-to-kill vs each enemy .tres, ammo
  economy. Numbers, not vibes.

### 8. anim-tester (MCP)
- `play_frames(rig_glb, anim, frames)` — headless render of N frames.
- Returns screenshots + a mesh-integrity check (no exploded vertices,
  no detached limbs). Fail loudly on skinning errors.

## AGENT 5 — Levels (Hana's tools)

### 9. ldtk-builder (MCP)
- `build(ldtk_file)` — generate a Godot scene: TileMap layers, entity
  markers as Marker3D, trigger zones as Area3D, all named per the LDtk data.
- Must round-trip: rebuild the same file twice → identical scene.

### 10. scene-wirer (MCP)
- `wire(tscn_shell, model_glb, mapping)` — inject mesh, materials, and
  collision into one of the 136 empty scene shells. Mapping says which
  node gets which mesh.
- `verify(tscn)` — scene opens in Godot 4.3 headless with 0 errors.

## AGENT 6 — QA (pax's tools)

### 11. gut-runner (MCP)
- `run(project_path)` — execute the GUT suite headless, parse output.
- Returns per-test pass/fail with file:line, plus a summary count.
- Must not hang: hard timeout per test, report timeouts as failures.

### 12. screenshot-diff (MCP)
- `capture(project, scene, camera)` — headless screenshot.
- `diff(before_png, after_png)` — changed regions + a heatmap. For visual
  regression across builds.

## AGENT 7 — Build & perf

### 13. build-exporter (MCP)
- `export(project, preset)` — presets: windows, linux, web. Headless
  Godot export.
- `smoke(binary)` — launch, wait 30s, confirm no crash, report binary size
  and startup time.

### 14. perf-profiler (MCP)
- `profile(project, scene, seconds)` — headless run, sample FPS, physics
  tick time, draw calls.
- Returns hotspots sorted by cost with the node paths responsible.

## AGENT 8 — Audio & dialogue

### 15. music-director (MCP)
The licensed soundtrack can't ship. This finds stand-ins.
- `match(reference)` — reference = "artist - title". Searches royalty-free
  libraries (freepd, pixabay music, filmmusic.io) for same-energy matches:
  tempo, mood tags.
- Returns: track, url, license, download. License receipt like asset-fetcher.

### 16. tts-placeholder (MCP)
- `speak(text, voice)` — free TTS to WAV, 44.1kHz mono, into the game's
  `res://assets/audio/sfx/` path convention.
- Placeholders only — the real voice acting is humans later. But dialogue
  timing gets tested now, not later.
