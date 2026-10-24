# Research and publication boundary

open-physics is an original clean-room project. This file defines what may enter the public repository and what must remain outside it.

## Allowed in this repository

- Original architecture and design documents.
- Original Rust/C/C++/Python implementations and tests.
- Public API names and behavior invented for open-physics.
- High-level, independently observed technical facts such as the existence of constraint schemas, accumulated-impulse clamping, warm starting, and solver diagnostics.
- References to public documentation and public project names for attribution or comparison.

## Not allowed in this repository

- Proprietary SDK archives, libraries, DLLs, executables, or extracted `.obj` files.
- Decompiled or disassembled pseudocode and reconstructed vendor source.
- Vendor headers, inline files, source trees, or substantial copied comments.
- Code transcribed from decompilation, whether manually or automatically renamed.
- Build scripts that download or repackage proprietary SDK materials.
- Any claim that open-physics is Havok, a Havok fork, or an official Havok binding.

## Implementation rule

Implementation starts from the public design and behavioral requirements. Developers do not translate vendor machine code or vendor source into a new language. When a design detail is uncertain, write a small independent prototype and choose behavior from documented requirements and tests rather than copying an implementation.

## Research artifacts

Reverse-engineering notes and artifacts used for architectural study live in a separate local research workspace. They are not versioned here, linked as downloadable assets, or required to build or use open-physics.

## Review checklist before every public push

1. `git diff --cached --name-only` contains no binary research artifacts.
2. No proprietary path, archive, object file, pseudocode dump, or vendor source is included.
3. Documentation distinguishes observed architectural facts from implementation decisions.
4. README disclosures still match the plugin's actual behavior.
5. No verification claim exceeds what was actually executed.
