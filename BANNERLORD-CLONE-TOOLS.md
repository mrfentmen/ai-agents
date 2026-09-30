# Mount & Blade-Style Web Game — Tooling Catalog

Existing MCP servers that can help build the Bannerlord clone on web, plus
confirmed gaps. Surveyed 2026-09-30. The skeleton repo (built by the boss's
local agents) will decide the final stack — wire these once it lands.

## EXISTING TOOLS — wire these in

### Web 3D (Three.js)

| Server | What it does | Use for |
|---|---|---|
| `threejs-devtools-mcp` (npx) | 59 tools: inspect + modify live Three.js scenes via Chrome DevTools, perf stats, material editor | Scene debugging, live iteration |
| `locchung three-js-mcp` | WebSocket scene control: add/move/remove objects, get state | Agent-driven scene building |
| `putervision/world-model-mcp` | Persistent 3D spatial memory, entity tracking, AABB collision sim, waypoint nav, Playwright game automation, multi-agent blackboard | Battle-scene entity tracking + automated playtests |

### Campaign / world simulation

| Server | What it does | Use for |
|---|---|---|
| `DangerBlack/fantasy-world-mcp` | Procedural fantasy world gen + EVOLUTION sim: population, tech, settlements (cave→village→city→ruins), raids, quest gen, religion, causal event tracking | Campaign-layer prototype: kingdoms rising/falling, dynamic history |
| `wwwbkgme-oss/voxelforge` | 22 MCP tools: terrain, buildings (medieval style!), characters, dungeons, sprites, dialogue trees, storyline gen | Rapid settlement/battlefield prototyping |

### Assets (carried over from FPS survey — still valid)

| Server | Sources | Use for |
|---|---|---|
| `evonar543/assetmcp` | itch.io, OpenGameArt, ambientCG — license metadata | Medieval weapons/armor/buildings |
| `jonit-dev/threenative-asset-mcp` | Fab, Sketchfab, Poly Haven + game-audio catalog | Units, horses, battle SFX |
| `carlosh7/blender-mcp` | 239 tools, headless | Model cleanup, GLB export for web |

### Testing

| Server | What it does | Use for |
|---|---|---|
| `mcp_playwright` (already in gateway) | Browser automation, screenshots, input | Web playtests, visual QA |

### Multiplayer — NOT an MCP, use the framework directly

No Colyseus MCP exists. **Colyseus** (open source, Node.js, WebSocket rooms,
matchmaking, authoritative state sync) is the standard answer for web
multiplayer — Vite plugin, Vercel-deployable. The skeleton should just use it.

## A2A / ACP — honest note

There are no "A2A game-dev servers." A2A (Google's Agent-to-Agent protocol, now
Linux Foundation) is how AI agents talk to *each other* — it's the crew
coordination layer, not a tool catalog. ACP (Agent Client Protocol) is
editor↔agent communication. The game-director ACP spec in BUILD-LIST.md is
already the right conductor design — it covers what A2A/ACP would do here.
Don't hunt for game-dev A2A servers; they don't exist.

## CONFIRMED GAPS — build these (nothing exists)

- **formation-ai** — troop formations, charges, morale breaks, unit AI for
  hundreds of battlefield agents
- **campaign-sim** — Bannerlord-style overworld: parties moving, sieges,
  economy, diplomacy, kingdom AI (fantasy-world-mcp prototypes this, but a
  game-ready sim needs building)
- **siege-kit** — walls, gates, ladders, siege engines as a reusable system
- **web-qa-harness** — scripted browser playtests with assertions
  (Playwright primitives exist, no game test runner)
