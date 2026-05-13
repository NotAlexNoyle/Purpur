---
created: "2026-05-12"
last_edited: "2026-05-12"
---

# Cavekit: Runtime DoS Protection

## Scope
Bound CPU and memory cost of per-packet operations and component-rendering recursion so a single malicious client cannot lag or crash the server. Detect non-recoverable NBT-parse failures during tab-complete and disconnect the sender. Covers the chat / sign / book / item-name render path, the command-suggestion wire path, and brigadier's alt-branch retry behavior.

## Requirements

### R1: Translatable component rendering is bounded
**Description:** When the server visits parts of a translatable component during text resolution, the total number of visited string-parts in a single visit must not exceed 32; exceeding degrades the result to a "..." placeholder rather than recursing further or throwing to callers.
**Acceptance Criteria:**
- [ ] A self-referential translation arg (a translation key whose arg is itself the same key) terminates and resolves to "...".
- [ ] A cross-referential translation chain (X → Y → X) terminates.
- [ ] A legitimate translatable component with 32 or fewer visited parts renders unchanged.
- [ ] No StackOverflowError or unbounded heap growth is reachable from a player-supplied chat, sign, book, or item-name component.
**Dependencies:** none

### R2: Separator components cannot themselves trigger expansion
**Description:** When a component is used as the separator in list-style component rendering, it must not be permitted to contain nested NBT contents, selector contents, or translatable args whose contents are NBT or selector. Invalid separators degrade to an empty separator.
**Acceptance Criteria:**
- [ ] A separator containing nested NBT contents → parent renders with an empty separator.
- [ ] A separator containing nested selector contents → empty separator.
- [ ] A separator containing translatable contents whose arg is a Component with NBT or selector contents → empty separator.
- [ ] A plain literal separator continues to render normally.
- [ ] The validity check recurses through translatable args.
**Dependencies:** none

### R3: Suggestion packet size bounded at wire decode
**Description:** The wire decoder of the command-suggestion packet rejects command strings whose UTF length exceeds 2048 bytes before any further processing of the packet contents.
**Acceptance Criteria:**
- [ ] Decoding a packet whose command UTF length exceeds 2048 raises the protocol's existing length-exceeded error.
- [ ] Decoding a packet whose command UTF length is 2048 or less succeeds.
- [ ] The receive-side limit applies regardless of upstream client behavior.
**Dependencies:** none

### R4: Long single-token tab-complete is treated as spam
**Description:** A suggestion request whose command string is longer than 64 characters and whose first space (if any) occurs at index 64 or later disconnects the sender on the server's existing spam-disconnect channel.
**Acceptance Criteria:**
- [ ] A 65-char command with no space disconnects the sender.
- [ ] A 200-char command whose first space is at index 70 disconnects.
- [ ] A 200-char command whose first space is at index 5 is permitted (legitimate long argument).
- [ ] A 64-char command with no space is permitted (boundary).
- [ ] The disconnect reuses the server's existing spam-disconnect channel (the translation key already used for chat spam, and the player-kick cause already used for spam), not a newly-introduced channel.
- [ ] The disconnect occurs before any tab-complete computation is dispatched.
**Dependencies:** none

### R5: Non-recoverable NBT-parse failures during suggestion parsing disconnect the sender
**Description:** When the dispatcher parse for a tab-complete suggestion contains the typed NBT exception (the world-data-integrity R2 exception) in its exception map, the sender is disconnected for spam and no further suggestion computation occurs.
**Acceptance Criteria:**
- [ ] A suggestion parse that produces the typed NBT exception → sender is disconnected.
- [ ] A suggestion parse that produces any other command-syntax exception → normal error path, no disconnect.
- [ ] No suggestion response is sent to the client after the disconnect.
**Dependencies:** R6 (the dispatcher must surface the exception into the exception map), cavekit-world-data-integrity.md R2 (the typed exception type).

### R6: Brigadier short-circuits on non-recoverable exceptions
**Description:** When a child parser throws the typed NBT exception during dispatcher parse, the dispatcher must not attempt the remaining alternate children with the same input; the typed exception propagates and the returned parse result reflects it.
**Acceptance Criteria:**
- [ ] Parsing a node whose first child throws the typed NBT exception and whose second child would succeed → the returned result reflects the original failure, not the alt-child success.
- [ ] Other RuntimeException types thrown during a child parse continue to wrap normally and allow alt-child attempts.
- [ ] The returned parse result's exception map contains the typed NBT exception.
**Dependencies:** cavekit-world-data-integrity.md R2 (the typed exception type).

## Out of Scope
- Selector-pattern error-message masking from the upstream patch (the class introducing the targeted code does not exist in this Minecraft version).
- Global packet rate-limiting or per-connection throttling.
- Pre-login or handshake-stage interactions.
- Changes to hot-path code outside the listed entry points (chat render, list-style render, command-suggestion wire decode, command-suggestion handler, brigadier dispatcher).

## Cross-References
- See also: cavekit-world-data-integrity.md — provides R2 (typed exception type) consumed here by R5 and R6.

## Source Traceability
- Upstream reference: `context/refs/upstream/0931-Improve-tag-parser-handling.patch` (PaperMC patch 0931, hunks for `TranslatableContents.java`, `ComponentUtils.java`, `NbtContents.java`, `SelectorContents.java`, `ServerboundCommandSuggestionPacket.java`, `ServerGamePacketListenerImpl.java`, `CommandDispatcher.java`).

## Changelog
- 2026-05-12: Initial draft.
