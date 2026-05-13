---
created: "2026-05-12"
last_edited: "2026-05-12"
---

# Cavekit: World-Data Integrity

## Scope
Bound NBT compound and list parsing depth so deeply-nested NBT structures cannot be persisted into world files (item NBT, books, anvils, signs, banner patterns, command-output chunks). A 513-deep compound parsed today survives in level.dat and region files and crashes the server on next load — that is the world-corruption vector. Capping at parse time prevents the structure from ever reaching disk.

## Requirements

### R1: NBT parse depth is bounded
**Description:** Any string-source parse of an NBT compound or list rejects inputs whose nested structural depth exceeds 512, throwing a typed exception that callers can distinguish.
**Acceptance Criteria:**
- [ ] Parsing a compound nested exactly 512 levels deep succeeds.
- [ ] Parsing a compound nested 513 levels deep throws the typed depth-exceeded exception.
- [ ] Parsing a list nested 513 levels deep throws the typed depth-exceeded exception.
- [ ] Parsing a mixed compound+list structure 513 levels deep throws.
- [ ] Each structure-close token decrements the depth counter so sibling structures of depth 512 do not falsely accumulate across siblings.
**Dependencies:** none

### R2: Typed non-recoverable exception exists
**Description:** A distinguishable exception type exists for "NBT too complex to parse safely" and is detectable by callers via type check.
**Acceptance Criteria:**
- [ ] Exception is distinguishable from generic command-syntax exceptions via instanceof.
- [ ] Exception lives under the upstream Paper brigadier namespace so downstream-plugin source compatibility with later Paper versions is preserved.
- [ ] Carries a literal message identifying the depth-limit cause.
**Dependencies:** none

### R3: All string-NBT entry points share the bound
**Description:** Every parser entry path that accepts untrusted NBT strings flows through the bounded parser; no alternate constructor or static helper bypasses the depth counter.
**Acceptance Criteria:**
- [ ] The closed set of entry paths is: parsing a single struct from a string, parsing as a brigadier argument, parsing a list from a string, and any plugin-facing API that constructs a string-source NBT parse. Every member of this set enforces the depth bound from R1.
- [ ] No public NBT parser entry path exists that can be invoked while skipping depth tracking.
- [ ] All entry points share the depth counter via a single instance field, so depth is consistent across the lifetime of one parse and reset for each new parse.
**Dependencies:** R1

## Out of Scope
- Binary NBT chunk load from gzipped region files (data the server wrote itself, not player input).
- Migration of pre-existing deep NBT already persisted in world files (this is prevention, not remediation).
- NBT size limits other than nesting depth (string length caps, list element count caps, byte budget).

## Cross-References
- See also: cavekit-runtime-dos-protection.md — consumes R2 to trigger a player-kick on the tab-complete path.

## Source Traceability
- Upstream reference: `context/refs/upstream/0931-Improve-tag-parser-handling.patch` (PaperMC patch 0931, hunks for `TagParser.java` and the new `TagParseCommandSyntaxException.java`).

## Changelog
- 2026-05-12: Initial draft.
