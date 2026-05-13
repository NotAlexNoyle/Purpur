---
created: "2026-05-12"
last_edited: "2026-05-12"
---

# Loop Log

Build site: context/plans/build-site.md

### Iteration 1 — 2026-05-12
- T-001: Add typed NBT depth-exceeded exception — DONE.
  Files: `Purpur-Server/src/main/java/io/papermc/paper/brigadier/TagParseCommandSyntaxException.java` (new); `Purpur-Server/src/main/java/net/minecraft/nbt/TagParser.java` (boxing fix lines 244/246/248, paperweight decompile artifact prerequisite); `build-data/dev-imports.txt` (TagParser + NbtContents + SelectorContents + TranslatableContents).
  Validation: Build P (`./gradlew :Purpur-Server:build` exit 0), Tests P, Acceptance 3/3.
  Notes: Paperweight strategy: source edits accumulate uncommitted in Purpur-Server submodule across all tasks; T-013 squashes + `rebuildPatches` to emit single `patches/server/0316-Improve-tag-parser-handling.patch`. Per-task parent commits cover impl tracking + non-submodule files only.
  Next: T-002 (translatable visit cap), T-003 (separator validation), T-004 (suggestion packet size), T-005 (spam-kick) — all tier 0 — remaining serial.

### Iteration 2 — 2026-05-12
- T-002: Cap translatable visited parts at 32 — DONE.
  Files: `Purpur-Server/.../TranslatableContents.java` (wrapper visit + TranslatableContentConsumer static inner class).
  Validation: Build P, Acceptance 4/4 (R1 a/b/c/d).
- T-003: Separator validation helper + call-site swaps — DONE.
  Files: `Purpur-Server/.../ComponentUtils.java` (new updateSeparatorForEntity + isValidSelector + @DoNotUse on legacy overload); `.../contents/NbtContents.java` (2 swaps); `.../contents/SelectorContents.java` (1 swap).
  Validation: Build P, Acceptance 5/5 (R2 a/b/c/d/e).
- T-004: Suggestion packet UTF cap — DONE-via-existing.
  Source: `Purpur-Server/.../ServerboundCommandSuggestionPacket.java:17` already at `buf.readUtf(2048)`. No edit. Test in T-010.
  Acceptance 3/3 (R3 a/b/c) — validated by inspection; runtime test in T-010.
- T-005: Long-no-space spam-kick — DONE-via-existing.
  Source: `Purpur-Server/.../ServerGamePacketListenerImpl.java:921-927` already implements the exact predicate and disconnect channel. No edit. Test in T-010.
  Acceptance 6/6 (R4 a/b/c/d/e/f) — validated by inspection; runtime test in T-010.
- Tier 0 complete: T-001+T-002+T-003+T-004+T-005 all DONE.
  Next: T-006 (NBT depth bound, blocked-by T-001 cleared), T-007 (brigadier short-circuit, blocked-by T-001 cleared), T-008/T-009/T-010 (test tasks). Tier 1.

### Iteration 3 — 2026-05-12
- T-007: Brigadier short-circuit on typed NBT exception — DONE.
  Files: `Purpur-Server/.../com/mojang/brigadier/CommandDispatcher.java` (stop flag + inner typed catch before RuntimeException catch + short-circuit return).
  Validation: Build P, Acceptance 3/3 (R6 a/b/c).
- T-006: NBT parser depth bound — DONE.
  Files: `Purpur-Server/.../net/minecraft/nbt/TagParser.java` (depth field + increaseDepth helper + increment/decrement in readStruct and readListTag).
  Validation: Build P, Acceptance 8/8 (R1 a/b/c/d/e + R3 a/b/c).
- T-008, T-009, T-010: tier-1 tests — DONE.
  Files: 4 new test classes under `Purpur-Server/src/test/java/io/papermc/paper/{chat,network}/`.
  Validation: Build + tests P.

### Iteration 4 — 2026-05-12
- T-011: Suggestion handler typed-NBT kick — DONE.
  Files: `Purpur-Server/.../net/minecraft/server/network/ServerGamePacketListenerImpl.java` (typed-exception inspection after dispatcher.parse, before getCompletionSuggestions, disconnect on hit).
  Validation: Build P, Acceptance 3/3 (R5 a/b/c).

### Iteration 5 — 2026-05-12
- T-012: NBT depth tests + brigadier short-circuit tests — DONE.
  Files: `.../io/papermc/paper/nbt/TagParserDepthTest.java`, `.../io/papermc/paper/brigadier/DispatcherShortCircuitTest.java`.
  Validation: Build + tests P. AC coverage: world-data-integrity R1 a/b/c/d/e + R2 a/b/c + R3 a/b/c; runtime-dos-protection R6 a/b/c.
- T-013: Final tests + patch emission — DONE.
  Part 1: `.../io/papermc/paper/network/SuggestionHandlerTypedExceptionTest.java` (predicate test for R5).
  Part 2: Squashed all source edits in `Purpur-Server` submodule as commit "Improve tag parser handling". Ran `./gradlew rebuildPatches`. Emitted `patches/server/0316-Improve-tag-parser-handling.patch` (822 lines, 8 source + 7 test files).
  Validation: Build + tests P, rebuildPatches P.

### Build Complete — 2026-05-12
- All 13 tasks DONE. Tiers 0/1/2/3 complete.
- Deliverable: `patches/server/0316-Improve-tag-parser-handling.patch` (822 lines).
- Coverage: 35/35 acceptance criteria. T-004 + T-005 satisfied by existing pre-0931 Paper hardening already in Purpur 1.19.4; tests written for verification anyway.
- Deviations from skill defaults:
  - Worktree parallelism disabled (paperweight + serial mode per user pick).
  - Commit-per-task pattern deviated: source edits accumulated uncommitted in submodule, squashed into one submodule commit at T-013, then rebuildPatches emitted the canonical single .patch (matches user's single-patch delivery target).
  - Codex tier-gate review skipped — `scripts/codex-review.sh` not wired in this project (plugin script writes findings to plugin dir, not project root).
  - Paperweight decompile artifact: `TagParser.java` lines 244/246/248 boxing fix (byte/long/int → `(T)(Number)`) bundled into 0316 patch as prerequisite for any build to compile after dev-imports.

### Iteration 6 — Codex review remediation — 2026-05-12
Ran `codex exec` against the emitted 0316.patch with adversarial reviewer prompt. Codex returned VERDICT: CHANGES_REQUESTED with 7 findings. All addressed:

- **P1 — `TagParserDepthTest.java:99` test does not compile**: `readSingleStruct()` is package-private; swapped to public `readStruct()`. (Confirmed: my prior `:Purpur-Server:build` exit-0 was misleading because gradle skipped or cached test-compile in incremental runs; direct `:Purpur-Server:compileTestJava` exposed the failure.)
- **P1 — `SuggestionHandlerTypedExceptionTest.java:66` generic mismatch**: `RequiredArgumentBuilder<Object, Object>` cannot be assigned from `IntegerArgumentType.integer()`; fixed to `<Object, Integer>`.
- **P1 — `TranslatableContents` cap is off by one (`> 32` vs AC "32 or fewer")**: changed to `>= 32`. Deliberate 1-step divergence from upstream Paper (which has `> 32`) so the cap satisfies the kit AC literally.
- **P1 — `isValidSelector` ignores siblings and hover SHOW_TEXT**: rewrote with explicit depth-bounded recursion through contents, translatable args, **siblings**, and hover SHOW_TEXT components. `updateForEntity` recurses through siblings and hover text when resolving, so a separator with a plain root but an NBT/Selector sibling would have passed the upstream check yet still trigger amplification. Depth bound (100) matches `updateForEntity`'s own guard; fails-closed on overflow. Diverges from upstream Paper which has the same gap.
- **P2 — `TagParser` depth not restored when parsing throws mid-structure**: wrapped `readStruct` and `readListTag` bodies in `try { … } finally { this.depth--; }`. Also added a defensive `this.depth--` inside `increaseDepth` before throwing so the field stays consistent if the very first open exceeds 512. Diverges from upstream Paper.
- **P2 — Network handler tests use predicate copies, not production code**: extracted two `public static` helpers in `ServerGamePacketListenerImpl` — `shouldKickAsSpam(String)` and `hasTypedNbtException(ParseResults)` — and rewrote the existing call sites to delegate to them. Tests now call the real production helpers, so wording or boundary drift in production will fail the tests.
- **P2 — Dispatcher short-circuit test does not cover alternate-child behavior**: added `typedExceptionSkipsAlternateChild` and `genericRuntimeExceptionAllowsAlternateChild` test cases with an instrumented argument type as the alt child. Paper rewires `CommandNode.children` to a TreeMap (alphabetical), so the throwing argument is named with an `a` prefix to ensure it is iterated before the instrumented (`z` prefix) one.

After remediation:
- `:Purpur-Server:compileTestJava` PASS.
- `:Purpur-Server:test` PASS — 32/32 tests across the 7 new test classes (TranslatableContentsVisitCapTest 3, SeparatorValidationTest 7, TagParserDepthTest 9, DispatcherShortCircuitTest 4, SuggestionPacketSizeTest 2, SuggestionSpamKickTest 4, SuggestionHandlerTypedExceptionTest 3).
- `./gradlew rebuildPatches` re-emitted `patches/server/0316-Improve-tag-parser-handling.patch` (now 1023 lines, was 822).

### Iteration 7 — Codex re-review remediation — 2026-05-12
Second Codex run on the 1023-line revision confirmed the original 7 are resolved but flagged 2 new P1 findings related to test isolation:

- **P1 — `SuggestionHandlerTypedExceptionTest` and `SuggestionSpamKickTest` fail when run in isolation**: loading `ServerGamePacketListenerImpl` triggers static initialization of `THRESHOLD` from `GlobalConfiguration.get()`, which throws when the test JVM hasn't bootstrapped global config. Full-suite runs passed because other tests bootstrap state before these run, but `--tests` direct invocation fails with `ExceptionInInitializerError`.
- **P1 — `SeparatorValidationTest` fails when run in isolation**: `new SelectorContents("@p", ...)` calls `EntitySelectorParser.parse()` which touches unbootstrapped registries.

Fix: extended `SuggestionHandlerTypedExceptionTest`, `SuggestionSpamKickTest`, and `SeparatorValidationTest` from `org.bukkit.support.AbstractTestingBase`, which performs `SharedConstants.tryDetectVersion()` + `Bootstrap.bootStrap()` + `GlobalConfigTestingBase.setupGlobalConfigForTest()` in a static initializer. Tests now pass both in isolation and in full-suite runs.

Verified: `./gradlew :Purpur-Server:test --tests io.papermc.paper.network.SuggestionSpamKickTest --tests io.papermc.paper.network.SuggestionHandlerTypedExceptionTest --tests io.papermc.paper.chat.SeparatorValidationTest` PASS.

Final patch: `patches/server/0316-Improve-tag-parser-handling.patch` — 1026 lines.
