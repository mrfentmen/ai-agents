# AI Agents — Build List

Servers an AI crew needs to **autonomously build a video game** — no human in
the loop between "here's the design doc" and "here's the playable build."
Written by Pax for Mrfentmen, 2026-09-29. Build these on your own machine,
push them here or straight into awesome-mcps / awesome-acps / awesome-a2as,
and the crew wires them into the gateway.

Priority order. Each entry: what it is, why it's missing, what it must do.

---

## 1. game-director (ACP / A2A) — THE CONDUCTOR ⭐ top priority

**What:** An agent (not a tool server) that takes a game design doc + asset
brief and runs the entire build loop by itself: fetch assets → rig them →
write code → screenshot-test → fix → export. The thing that makes "I give you
info, you build the game" real.

**Why missing:** Every piece exists (asset search, rigging, engine drivers,
screenshots). Nobody wired them into a closed loop. This is the autonomy
unlock — everything else on this list is a tool it calls.

**Must do:**
- Accept a design doc (markdown) + constraints (engine, style, budget).
- Plan the build as lanes (player / enemies / world / audio) and dispatch.
- Call asset MCPs for models, rigging MCP for skeletons, engine-driver MCP
  for code/scene work.
- Run the QA harness after every lane completes; on failure, read the
  screenshots + logs, patch, retry (max N retries, then escalate to human).
- Produce a playable export at the end. Report what was verified vs not.

## 2. lua-port (MCP)

**What:** Transpiler server: Garry's Mod Lua (GLua) → Roblox Luau, and GLua →
Godot GDScript. Preserves logic, maps engine APIs, flags what can't port.

**Why missing:** Nobody built it. Mrfentmen has a full GMod prototype whose
logic should port, not be rewritten.

**Tools:**
- `transpile(code, target)` — `luau` | `gdscript`. Returns converted code +
  a list of unmappable calls with suggestions.
- `analyze_project(path)` — walk a GMod addon/gamemode, inventory scripts,
  rank files by portability.
- `map_api(glua_call)` — "what's the Luau/GDScript equivalent of X?"

## 3. gore-kit (MCP + Godot addon)

**What:** Dismemberment + ragdoll, done right, as a reusable kit. Born from a
month of fighting Godot's built-in ragdoll.

**Why missing:** Godot ships `PhysicalBone3D` but the defaults are broken
(PIN joints crumple, auto collision shapes are too small). The hard-won setup
knowledge exists nowhere as a package.

**Must do:**
- `setup_ragdoll(skeleton)` — generate physical bones with CONE joints,
  hand-tuned collision shapes, damping + self-collision presets that don't
  explode.
- `add_dismemberment(character)` — limb-detach system: lethal limb hit →
  hide/detach mesh, spawn gib + blood particles, ragdoll the chunk.
- `partial_ragdoll(bone)` — hit-reaction: simulate hit bone + children
  briefly, impulse, blend back to animation.
- Ship with a test scene proving each feature at 60fps.

## 4. psx-ify (MCP)

**What:** Feed it a modern PBR asset, get back PS1/PS2-era: vertex snapping,
palette crush, texture downscale, affine-style mapping.

**Why missing:** The PSX aesthetic is huge on itch.io but every artist does
the conversion by hand. No tool server exists.

**Tools:**
- `psxify(asset, options)` — input GLB/FBX/OBJ + texture; options for
  vertex snap grid, palette size, texture resolution cap. Returns converted
  GLB + a before/after preview render.

## 5. game-qa-harness (MCP)

**What:** Scripted playtests with assertions for games. The engine-driver MCPs
give you screenshots and input injection; nobody built the test runner.

**Why missing:** Game QA is still humans pressing buttons. For autonomy, the
crew needs assertions.

**Tools:**
- `run_scenario(project, script)` — scripted input sequence (move, shoot,
  jump) against a running build; captures screenshots + frame metrics.
- `assert_no_clip`, `assert_fps(min)`, `screenshot_diff(vs_last_build)` —
  regression checks.
- Returns pass/fail + evidence bundle per scenario.

---

## Do NOT rebuild — wire these in

These exist. Cloning them wastes time; wrap or fork only if they fall short.

| Server | What it does |
|---|---|
| `erodenn/godot-mcp-runtime` | Godot headless editing + runtime control, screenshots, input sim |
| `amyjeanes/gmod-mcp-server` | Run Lua inside a live GMod session |
| `evonar543/assetmcp` | Search Kenney/Quaternius/itch.io/Godot Asset Lib, license checks, Godot 4 scaffolding |
| Meshy MCP servers | Text/image-to-3D, auto-rig, animations (needs Meshy API key) |
| `three.ws` rig tool | Free auto-rig, no key (humanoids) |
| Bake3D API | Image/text → rigged character, non-humanoids too |
| `roblox-studio-mcp` | Drive Roblox Studio (note: Studio has no Linux build — headless testing gap is unfillable) |

## The honest footnote

These tools close the loop on *building*. The last 5% — "is it fun, does it
feel right" — is still a human. Nobody's automating taste.
