# Roadmap

The milestones are capability gates, not date promises. A milestone is complete only when its acceptance check has actually run.

## M0 — Minimal useful core

Scope: an original native core with rigid bodies, gravity, sphere/plane contacts, sequential-impulse solving, and a small C ABI. Initial Hermes tools: world creation, body creation, stepping, and state inspection.

**Acceptance:** from a real Hermes session, create a documented stack of bodies, advance it by a fixed step count, inspect the resulting state, and confirm that no error or fabricated result is being used as a success signal.

## M1 — Determinism and diagnostics

Scope: canonical world/input serialization, replay logs, bit-exact test harness on supported platforms, maximum-violation and breached-impulse reporting.

**Acceptance:** replay the M0 scenario repeatedly and on every supported CI target; compare the complete canonical result and publish the exact comparison scope.

## M2 — Dedicated chain and cloth path

Scope: choose the formulation after independent prototypes (XPBD/PBD/FEM), add a dedicated chain/cloth solver and continuous collision tests.

**Acceptance:** small documented cloth scenarios with reproducible outcomes and solver diagnostics. No feature claim based only on a rendered demo.

## M3 — Bindings and studio integration

Scope: Python/MCP packaging, Godot GDExtension, and WASM build. Integrate with the `godot-studio` verify/export/playtest loop.

**Acceptance:** the same canonical scenario produces the documented result through each binding, including a real exported web build where supported.

## M4 — Active ragdoll and motor brain

Scope: ragdoll joint limits, motor atoms with explicit force limits, goal-driven behaviors (balance, grasp, get-up) running inside the step, a default-on self-preservation behavior pack (flinch, brace, protect, grab-at-support, grab-at-wound) arbitrated by a deterministic priority stack, solver diagnostics consumed as proprioception, and the character-chain stability decision (soft-constraint tuning versus a dedicated articulated chain path, shared with M2). The `patch` operation for per-frame motor targets and bracing gains is part of the compiler scope.

**Acceptance:** a documented character scenario — pushed off a curb, staggers and recovers; grabs a railing on a moving object and holds until its force limit is exceeded, then slips; gets up after a fall; flinches from an approaching object and braces before impact; guards a positioned injury — reproduced bit-exact from a replay log, with solver diagnostics showing no accumulated cheat forces. No scripted animation in the outcome.

## M5 — Game-engine and Omniverse integration

Scope: Unity and Unreal native adapters over the C ABI (an open-physics world owned by the plugin, transform synchronization with the host, the host's native physics left in place), and an OpenUSD bridge that consumes scenes authored with the standard USD Physics schema (`UsdPhysics`) and steps them with the deterministic core — as an Omniverse Kit extension or a standalone USD consumer, whichever proves the cleaner path first.

**Acceptance:** the same canonical scenario produces the documented result through each host adapter; one documented USD-authored scene round-trips into the canonical world description and replays bit-exact from its replay log. No integration claim based on a rendered demo.

## Current state

Only the pre-alpha repository scaffold is complete. No milestone above is claimed as implemented.
