---
created: "2026-05-12"
last_edited: "2026-05-12"
---

# Cavekit Overview

## Project
Purpur 1.19.4 — fork of Paper. Paperweight patcher model: edits live in `patches/api/` and `patches/server/`. Current focus: backport applicable hunks of PaperMC patch 0931 ("Improve tag parser handling") to mitigate NBT-bomb world corruption and runtime DoS via tab-complete, translatable components, and brigadier non-recoverable exceptions. Delivery target: single new patch in `patches/server/`, JUnit tests covering parsers.

## Domain Index
| Domain | Cavekit File | Requirements | Status | Description |
|--------|--------------|--------------|--------|-------------|
| world-data-integrity | cavekit-world-data-integrity.md | R1–R3 | DRAFT | Bounded NBT parse depth so deep NBT cannot reach disk. |
| runtime-dos-protection | cavekit-runtime-dos-protection.md | R1–R6 | DRAFT | Bounded component recursion, tab-complete size + spam-kick, brigadier short-circuit on non-recoverable NBT exceptions. |

## Cross-Reference Map
| Domain A | Interacts With | Interaction Type |
|----------|----------------|------------------|
| world-data-integrity | runtime-dos-protection | Exports R2 (typed exception) — consumed by runtime-dos-protection R5 and R6. |

## Dependency Graph
1. **world-data-integrity** — no dependencies. Implement first (R2 typed exception type must exist before runtime-dos-protection R5/R6 can reference it).
2. **runtime-dos-protection** — depends on world-data-integrity R2.
