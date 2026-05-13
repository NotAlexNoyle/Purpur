---
created: "2026-05-12"
last_edited: "2026-05-12"
---

# Implementation Tracking: world-data-integrity

Build site: context/plans/build-site.md

| Task | Status | Notes |
|------|--------|-------|
| T-001 | DONE | Wrote `Purpur-Server/src/main/java/io/papermc/paper/brigadier/TagParseCommandSyntaxException.java` (public final, extends CommandSyntaxException, namespace `io.papermc.paper.brigadier`, literal NBT-too-complex message). Full `./gradlew :Purpur-Server:build` PASS. Also fixed an unrelated paperweight decompile artifact in `TagParser.java` lines 244/246/248 (added explicit `(Number)` cast for autoboxing of byte/long/int into `T extends Number`) so the imported file compiles; this fix will be bundled into patch 0316. |
| T-006 | DONE | Edited `Purpur-Server/.../TagParser.java`: added `private int depth` field, `increaseDepth()` helper that throws `TagParseCommandSyntaxException("NBT tag is too complex, depth > 512")` when depth exceeds 512, increment after `expect('{')` in `readStruct` and after the value-readability guard in `readListTag`, decrement after `expect('}')` / `expect(']')`. Build PASS. Verified against AC R1 a/b/c/d/e + R3 a/b/c via T-012 tests. |
| T-012 | DONE | JUnit tests for NBT depth + brigadier short-circuit. Files: `Purpur-Server/src/test/java/io/papermc/paper/nbt/TagParserDepthTest.java` (10 tests: 512 succeeds, 513 compound throws typed, 513 list throws typed, 513 mixed throws typed, sibling 512-each succeeds, fresh parser resets depth, namespace check, instanceof check, alt-entry-path via readSingleStruct enforces same bound); `.../brigadier/DispatcherShortCircuitTest.java` (2 tests: typed exception propagates into ParseResults.getExceptions; generic RuntimeException does NOT produce a typed exception). Build PASS. |
