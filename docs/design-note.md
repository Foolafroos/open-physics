# open-physics — public design note

This document describes the intended architecture of an original, clean-room physics engine. It contains no vendor source code, decompiled pseudocode, or proprietary binary artifacts.

## 1. Product contracts

### Universal by ABI, not by wrappers

The native core will expose a small, stable C ABI. Python/MCP, Godot GDExtension, and WASM bindings should all consume that same ABI rather than fork the engine's behavior.

### Agent-native

An agent must be able to create a world, add bodies and constraints, advance it by an exact number of steps, query contacts, perform raycasts, read solver diagnostics, and replay a recorded input log. The engine is headless-first and requires no editor or GUI.

### Deterministic by contract

The same canonical world description and input/step log must produce identical results on every supported platform. Determinism is a tested public guarantee, not a best-effort flag. Any platform-specific exception must be explicit and tested.

## 2. Solver architecture

### Pure solver boundary

The solver should be conceptually pure:

```text
solve(configuration, constraint_program, velocity_accumulators, scratch) -> status
```

No hidden world state should be required to reproduce a solve. State that affects results must be part of the serialized input or the accumulator/scratch model.

### Constraints as a data program

A constraint group is encoded as a stream of schemas:

- control records for group boundaries and iteration structure;
- typed constraint records with expensive values precomputed, including effective mass and lever arms;
- separate metadata for record size and solver-temporary requirements.

A schema ID and a kernel are not required to have a one-to-one relationship. Specialized variants may share a kernel when their runtime behavior is identical.

### Two IRs: authoring and solver

The public API should not expose solver schemas directly. The engine needs two distinct intermediate representations:

1. an **authoring IR**: ordered, canonically serializable constraint atoms, including local frames and modifiers;
2. a **solver IR**: compiled schemas with explicit sizes, temporary counts, and optimized variants.

The compiler between them must be deterministic and must produce a memory plan — schema bytes, solver results, and solver temporaries — before emitting the program. Timing factors and accumulator references are baked during this build rather than rediscovered by the hot loop.

This design makes constraint ordering explicit data, which is essential for deterministic replay. It also lets the implementation choose between separate kernels and an inlined switch without changing the public model.

Study of representative builders (a ball-socket joint, a soft linear axis, an ignore modifier) sharpened three rules for the authoring IR:

- **Atom records are fixed-size and 16-byte aligned in a flat arena.** The list is traversed by record-size arithmetic, not by pointers between atoms. Record sizes must therefore be explicit, stable, and part of the public format.
- **Modifiers act on emission, not on the solver.** Disabling a constraint is implemented by rewinding the schema cursor over the group header that was just written; the constraint never enters the solver program. The compiler should expose a small set of program-level operations (cancel, wrap, patch) instead of a modifier class per behavior.
- **Variants are explicit fields, never sentinel values.** Choosing a limited-impulse variant by comparing a maximum against float-max is fragile and invisible in the data. The authoring IR should carry optional fields (e.g. an impulse limit) and let the compiler select the variant from them.

### Sequential impulses with accumulated clamping

Contact resolution uses sequential impulses. The impulse delta is clamped against the **accumulated impulse**, not only against the current delta. Limited contacts add an upper bound derived from their configured maximum impulse. The accumulated value persists for warm starting.

### Warm starting

Velocity accumulators and previous accumulated impulses are first-class inputs to the solve. Their storage and canonical ordering must be defined so that replay does not depend on allocation addresses or hash-table iteration order.

### Multi-pass solving

The stepper should support multiple solver passes with integration between passes. Each pass is a discrete, testable replay point. Gravity and integration factors are explicit configuration, never ambient globals.

### Native diagnostics

Solver diagnostics are designed from the start:

- maximum constraint violation by type;
- breached impulse limits and broken constraints;
- per-island cost and iteration counts;
- deterministic run identifiers for replay comparison.

A playtester agent must be able to inspect solver health rather than infer it from screenshots.

### Breakage and specialized solvers

Constraint breakage is a native state, not a gameplay workaround. Specialized solvers are planned for chains and cloth instead of forcing every problem through generic contact handling.

### Character layer: motor brain (active ragdoll)

A goal-driven character controller — the capability that made Euphoria-powered characters in GTA IV and Red Dead Redemption stagger, grab at supports, and get up — is a third layer above the two IRs, not a new physics engine:

- **Body**: a skeleton of rigid bodies and colliders joined by limit atoms (the public ragdoll decomposition: ball-socket plus angular/twist/cone limits).
- **Muscles**: motor atoms with explicit force limits. The force limit *is* muscle strength; when the impulse a grasp demands exceeds it, the hand slips and the visible stagger emerges from the data, not from a special case.
- **Brain**: behaviors are pure functions from perception (contacts, pose, velocities, goals) to motor targets and gains, plus emission or removal of grasp constraints. They run inside the step, closing the loop.

Consequences for the compiler: motor targets are per-instance values baked into schemas, so an active character rebuilds or patches its constraint program every frame. The program-level `patch` operation (rewrite baked values without re-emitting structure) is therefore a first-class requirement, not an optimization.

"Euphoria's results are different every time" is sensitivity to initial conditions, not randomness. A deterministic brain keeps bit-exact replay intact; explicit seeded variation is opt-in.

Stability is the open architectural question. Field testimony around this class of controller is consistent: on a plain sequential-impulse solver, active ragdolls fight instability and accumulate cheat forces; a whole-chain articulated formulation (Featherstone-style) respects configured joint strengths and avoids foot jitter. The engine keeps sequential impulses for M0/M1 and defers the character-chain formulation decision to M4, where the "pushed → staggers → grabs a railing" scenario and solver diagnostics decide between soft-constraint tuning and a dedicated chain path (shared infrastructure with M2).

## 3. Beyond the reference architecture

1. **First-class cloth** through a dedicated chain/cloth path with continuous collision handling.
2. **Determinism as a contract**, including canonical constraint ordering and reproducible replay.
3. **Modern SIMD design** from the first kernel, without changing numerical results between paths.
4. **Agent-facing diagnostics and replay** as public features rather than debug afterthoughts.

## 4. Proposed implementation choices

These choices remain open until M0 implementation begins:

- **Rust core with a C ABI** — preferred for memory safety, WASM support, and shared bindings. A C++ core remains a defensible alternative if middleware interoperability requires it.
- **Caller-owned arenas** — no garbage collector, no exceptions across the ABI, and no hidden global allocator.
- **Canonical serialization** — world descriptions and replay logs use a stable, documented format rather than native struct dumps.

## 5. Hermes integration

The Hermes plugin will expose small atomic tools; the bundled skill will teach the workflow. Tools must never report success without returning inspectable state. Replay and violation reporting belong to M0/M1, not to a distant debug release.

The project is complementary to `godot-studio`: Godot tooling handles verification and export; open-physics is intended to provide deterministic simulation for playtesting.

## 6. Explicit non-goals for the scaffold

- No claim of Havok compatibility or feature parity.
- No redistribution of proprietary binaries, objects, headers, or decompiled output.
- No fabricated M0 implementation, benchmark, or determinism result.

See [`roadmap.md`](roadmap.md) for delivery milestones and [`research-boundary.md`](research-boundary.md) for publication rules.
