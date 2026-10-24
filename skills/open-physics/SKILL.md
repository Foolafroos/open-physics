---
name: open-physics
description: "Use when an agent needs deterministic physics simulation in Hermes. Enforce honest availability checks and the build-step-replay workflow."
version: 0.1.0
author: Fool (Dingbat)
license: MIT
metadata:
  hermes:
    tags: [physics, simulation, deterministic, hermes-plugin]
    related_skills: [godot-studio]
---

# Open Physics

## When to use

- An agent needs to create, advance, query, or replay a deterministic physics world.
- A playtester needs contacts, raycasts, or solver diagnostics rather than visual guesses.

## Availability guardrail — current stage

This repository is a **pre-alpha scaffold**. No `physics_*` tools are registered yet.
Do not claim that a world was simulated, that a replay matched, or that a build is
physically correct. If the planned tools are unavailable, report that the native core
is not implemented and stop instead of substituting another engine or fabricated output.

## Intended M0 workflow

Use this loop only after the corresponding tools actually exist:

```text
physics_world_create
  → physics_body_add (for every body)
  → physics_step (fixed step count)
  → physics_contacts_report / physics_raycast
  → physics_replay (same input log; require identical results)
```

Rules for the future implementation:

1. Keep the initial world description explicit and serializable.
2. Never infer simulation success from the absence of an error; inspect the resulting state.
3. Treat determinism as a testable contract: same inputs and step log must produce identical outputs.
4. Report maximum constraint violation and broken impulses with simulation results.
5. Do not present the engine as Havok or as containing Havok code. It is an original clean-room implementation.
