# open-physics

A clean-room, deterministic physics engine built to be embedded — in game engines, in OpenUSD / NVIDIA Omniverse pipelines, and in [Hermes Agent](https://github.com/NousResearch/hermes-agent). The founding use case is agent-native: give an agent a body it can create, advance, query, and replay. The same guarantees are what game studios and simulation platforms are missing from a physics middleware.

**Status: pre-alpha scaffold.** This repository currently contains the loadable Hermes plugin shell and the public design documents. It does **not** contain a physics implementation yet, and the plugin deliberately registers **no simulation tools** until the native core exists.

## Why

Agent game-studio pipelines can generate assets and export builds, but a playtester agent cannot judge a world it cannot simulate. open-physics aims to provide that missing capability directly inside Hermes:

- headless-first worlds, bodies, constraints, and queries;
- deterministic stepping and replay as a public contract, not an option;
- native solver diagnostics (maximum violation, broken impulses, island costs);
- goal-driven characters: active ragdolls whose muscles are force-limited constraint motors, with a default self-preservation behavior pack (flinch, brace, protect, grab-at-support) — emergent staggering, grasping, and getting up, with no scripted animation;
- a future first-class cloth solver;
- one core behind a small, stable C ABI: Python/MCP, Godot GDExtension, WASM, Unity/Unreal native plugins, and an OpenUSD/Omniverse bridge.

The properties that make the engine useful to an agent — bit-exact replay, inspectable solver state, no hidden globals — are exactly what a game studio needs from reproducible physics regression tests and lockstep replay, and what an OpenUSD pipeline needs from a verifiable simulation backend. open-physics is therefore designed as middleware: the host keeps its renderer and its native physics; open-physics runs its own deterministic world and exchanges state through transforms and queries.

The architecture is informed by independent analysis of public and locally available materials. It is not a Havok wrapper, fork, or redistribution.

## Integration targets

| Host | Path | What open-physics adds |
|---|---|---|
| Hermes Agent | Python/MCP plugin (this repository) | agent-operable worlds, replay, solver diagnostics |
| Godot | GDExtension over the C ABI | deterministic simulation for playtesting inside the studio loop |
| Web | WASM build | the same canonical result in browser builds |
| Unity / Unreal | native plugin over the C ABI | a reproducible physics world inside the host; the host's native physics (Unity PhysX, Unreal Chaos) is not replaced |
| OpenUSD / NVIDIA Omniverse | bridge consuming scenes authored with the standard USD Physics schema (`UsdPhysics`) | bit-exact, replayable stepping of USD-authored scenes, alongside the platform's default PhysX stepping |

Every host consumes the same core through the same C ABI. No host-specific fork of the engine's behavior.

## Clean-room boundary

This public repository must never contain proprietary SDK binaries, extracted object files, decompiled pseudocode, or copied vendor source. Research artifacts remain outside this repository. Only architectural facts and original implementation work belong here. See [`docs/research-boundary.md`](docs/research-boundary.md).

## Planned Hermes tools

These names describe the intended M0 interface, **not currently available functionality**:

| Tool | Intended role |
|---|---|
| `physics_world_create` | Create an isolated deterministic world |
| `physics_body_add` | Add a rigid body with an initial shape and transform |
| `physics_step` | Advance a world by a fixed number of deterministic steps |
| `physics_raycast` | Query the world without mutating it |
| `physics_contacts_report` | Inspect contacts and solver violations |
| `physics_replay` | Replay a recorded input/step log and verify the result |

The bundled skill explicitly tells agents not to claim that these capabilities exist while they are unavailable.

## Repository layout

```text
plugin.yaml                    Hermes manifest (zero tools at this stage)
__init__.py                    Loadable register(ctx) entry point
skills/open-physics/SKILL.md    Agent workflow and availability guardrails
core/                          Future native engine core
docs/design-note.md            Public architecture
docs/roadmap.md                Milestones M0–M5
docs/research-boundary.md      Clean-room and publication rules
```

## Disclosed behaviour

At the current scaffold stage, the plugin:

- executes no shell commands and starts no background processes;
- makes no network calls;
- reads or writes no files outside this package;
- stores no credentials and emits no telemetry;
- registers no tools, hooks, middleware, or desktop surfaces.

This section must be updated before any capability is added.

## License

MIT for original work in this repository. No third-party physics engine code is included.
