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
