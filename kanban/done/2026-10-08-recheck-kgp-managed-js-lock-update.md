# REVIEW READY — independent lock review, Session 566

# Task Tree

- `Review the entire frozen generated JS tooling lock delta`() // complete
- `Check retained application closure and coordinator Node evidence`() // complete
- `Return scoped readiness with outstanding adoption gates`() // complete

# Details

- Parent: [toolchain compatibility](../executable/2026-10-08-validate-kodex-toolchain-compatibility.md).
- User authorized standard `kotlinUpgradeYarnLock` after the actual KGP2.4.20
  mismatch, full review and Node-before-Native; original mismatch checks/Werror remain.
- Read-only reviewer: only this report edited with apply_patch; no tests, builds,
  Gradle/IDE/Git/device/credential operations or network writes. Primary-source GETs
  and lightweight in-memory lock/metadata inspection only; nothing production-adopted.

## R — scoped result and fixed provenance

- **Lock-input adoption READY, conditional on B2; no confirmed B1.**
  This reviews the fourth input, not Session565's separate three-file input or whole DevExp.
- Sole lock evidence: [manifest](file:///tmp/kodex-js-lock-final-review-20261008/SHA256SUMS#L1-L2).
  Manifest SHA256 verified `f08e1ed2edd8d20f7f10f85580a2315fe9d06a0a7c614521d22f52893f0926b6`;
  both manifest entries verified. Before SHA256:
  `3b1cd136c62d05e4ebb8d256f2c50832bb44fb5e370b1b8b6bea7d54687d5be8`;
  candidate SHA256: `60c4d2a402b4da36cc1555cefee9bbfda15aa6b21f5b0fc8ac452f6cb73470ac`.
- Accepted5b41be52 origin is coordinator-provided; independently, before-yarn.lock
  equals [current production lock](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/kotlin-js-store/yarn.lock#L1-L2442)
  byte-for-byte. Frozen candidate line numbers are NOT current production line numbers.
- Read all **544 unified-diff lines**. Lock blocks354→347; four managed roots'
  dependency closure187→178, with no missing dependency/optional-dependency selector.
  Every altered old/new block belongs to that closure, except retained mime-types'
  selector pruning caused by Webpack dropping its old edge; its payload is unchanged.

## R — full delta map and managed recipe

| Responsibility | Current production / frozen before | Frozen candidate |
| --- | --- | --- |
| Webpack5.101.3→5.108.1; complete direct edges | [production2286–2323](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/kotlin-js-store/yarn.lock#L2286-L2323) | [2240–2266](file:///tmp/kodex-js-lock-final-review-20261008/yarn.lock#L2240-L2266) |
| CLI6.0.1→7.2.1; json-ext1.1.0/commander14; remove configtest/info/serve, colorette, fastest-levenshtein | [before2246–2269](file:///tmp/kodex-js-lock-final-review-20261008/before-yarn.lock#L2246-L2269) | [2205–2217](file:///tmp/kodex-js-lock-final-review-20261008/yarn.lock#L2205-L2217) |
| Webpack resolver/parser: acorn split8.19.0, enhanced-resolve5.26.0, lexer2.3.2; remove obsolete glob/json-parse/type edges | [before2286–2323](file:///tmp/kodex-js-lock-final-review-20261008/before-yarn.lock#L2286-L2323) | [new edges2250–2266](file:///tmp/kodex-js-lock-final-review-20261008/yarn.lock#L2250-L2266) |
| Browserslist4.29.3: baseline2.11.27/caniuse30001815/electron1.5.451/node-releases2.0.57/update-db1.3.4 | [before579–589](file:///tmp/kodex-js-lock-final-review-20261008/before-yarn.lock#L579-L589) | [553–563](file:///tmp/kodex-js-lock-final-review-20261008/yarn.lock#L553-L563) |
| terser-webpack-plugin→minimizer-webpack-plugin5.13.0; terser5.51.2/schema-utils4.5.0/ajv-formats3 | [before2146–2165](file:///tmp/kodex-js-lock-final-review-20261008/before-yarn.lock#L2146-L2165) | [1502–1510](file:///tmp/kodex-js-lock-final-review-20261008/yarn.lock#L1502-L1510), [1838–1846](file:///tmp/kodex-js-lock-final-review-20261008/yarn.lock#L1838-L1846), [2115–2124](file:///tmp/kodex-js-lock-final-review-20261008/yarn.lock#L2115-L2124) |
| mime-db1.54.0 added; old mime-types/mime-db retained; webpack-sources3.6.0 | [before1512–1517](file:///tmp/kodex-js-lock-final-review-20261008/before-yarn.lock#L1512-L1517), [2281–2284](file:///tmp/kodex-js-lock-final-review-20261008/before-yarn.lock#L2281-L2284) | [1461–1476](file:///tmp/kodex-js-lock-final-review-20261008/yarn.lock#L1461-L1476), [2235–2238](file:///tmp/kodex-js-lock-final-review-20261008/yarn.lock#L2235-L2238) |
| KGP helpers3.0.0→3.3.0, Mocha11.7.5→11.7.6 | [production1437](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/kotlin-js-store/yarn.lock#L1437-L1442), [1555](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/kotlin-js-store/yarn.lock#L1555-L1580) | [1391–1396](file:///tmp/kodex-js-lock-final-review-20261008/yarn.lock#L1391-L1396), [1524–1549](file:///tmp/kodex-js-lock-final-review-20261008/yarn.lock#L1524-L1549) |

- Remaining trace-mapping/types/ajv/cross-spawn/envinfo/import-local/loader-runner/
  tapable/watchpack changes are selector merges/pruning or range changes, not
  additional payload upgrades. Removed type packages follow Webpack's removed edge.
- Actual KGP2.4.20 [recipe](https://github.com/JetBrains/kotlin/blob/v2.4.20/libraries/tools/kotlin-gradle-plugin/kotlin-npm-tooling/package.json#L16-L28)
  and [bundled package lock](https://github.com/JetBrains/kotlin/blob/v2.4.20/libraries/tools/kotlin-gradle-plugin/kotlin-npm-tooling/package-lock.json)
  match all four root versions/integrities; [generation wiring](https://github.com/JetBrains/kotlin/blob/v2.4.20/libraries/tools/kotlin-gradle-plugin/build.gradle.kts#L791-L797)
  generates NpmVersions from tooling inputs. This is not a manual Webpack/Koin override.
- Standard [upgrade wiring](https://github.com/JetBrains/kotlin/blob/v2.4.20/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/targets/js/yarn/YarnPluginApplier.kt#L99-L156)
  forces consistent install then copies the generated lock; normal store still checks mismatch.
- All21 new/replaced payload URL paths, versions, SHA1 fragments and SHA512 values
  matched public npm version metadata via GET, e.g. [Webpack](https://registry.npmjs.org/webpack/5.108.1)
  and [CLI](https://registry.npmjs.org/webpack-cli/7.2.1). Other URL/SRI fields unchanged;
  unchanged Karma GitHub commit/no-SRI entry is [1357–1359](file:///tmp/kodex-js-lock-final-review-20261008/yarn.lock#L1357-L1359).
  No introduced private URLs, auth material, local paths or unrelated application input.

## R — application preservation and minimum ablation

- Entire34-block sharp/node-pty closure, including optional native/libvips packages,
  identical: [sharp1872–1905](file:///tmp/kodex-js-lock-final-review-20261008/yarn.lock#L1872-L1905),
  [node-pty1571–1581](file:///tmp/kodex-js-lock-final-review-20261008/yarn.lock#L1571-L1581), [platform entries22–182](file:///tmp/kodex-js-lock-final-review-20261008/yarn.lock#L22-L182).
- Application declarations unchanged: [sharp44](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/impl/build.gradle.kts#L42-L44),
  [node-pty49](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/build.gradle.kts#L47-L49).
- Compiler/Werror and browser/Node/Native targets stay in [shared convention14–45](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/buildSrc/src/main/kotlin/kodex.kmp-shared.gradle.kts#L14-L45)
  and [host convention9–23](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/buildSrc/src/main/kotlin/kodex.kmp-host.gradle.kts#L9-L23).
  No stdlib/tooling override found in product build scripts; [automatic stdlib](https://kotlinlang.org/docs/js-project-setup.html#kotlin-standard-libraries)
  follows KGP, not npm yarn.lock. [Separate catalog2/21/32–33](file:///tmp/kodex-toolchain-koin-review-20261008/gradle/libs.versions.toml#L1-L33)
  supplies KGP/Compose2.4.20 and Koin1.2.1; Koin is unrelated to this npm delta.
- Minimum static ablation: reverting only Mocha's1524 selector to11.7.5 removes
  the recipe-required11.7.6 input; dropping minimizer1502–1510 leaves Webpack2261
  dangling. Restoring only old CLI/Webpack roots requires their old closure, not
  hand-splicing versions. Whole-old-lock actual mismatch already occurred before tests.
  No ablation executed; no generic DependencyLockService or blanket audit proposed.

## B1 — confirmed blockers

- None established in this exact lock delta; metadata matching does not certify tarball bytes or vulnerability freedom.

## B2 — precise coordinator-owned adoption gates

- [Central evidence134–143](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-10-08-validate-kodex-toolchain-compatibility.md#L134-L143)
  records standard update28s and original stdio/HTTP Node52s passed, not reviewer execution.
- Keep the four-file candidate atomic and frozen; complete Linux full CLI release link
  and actual Process/Shell/Curl Native targets, explicit source-composite compatibility
  and required IDEA Sync before production migration. Native currently running, not passed.
- Preserve mismatch FAIL/Werror. [CLI7 requires Node>=20.9](https://registry.npmjs.org/webpack-cli/7.2.1); [KGP default24.16](https://github.com/JetBrains/kotlin/blob/v2.4.20/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/targets/web/nodejs/BaseNodeJsRootExtension.kt#L91)
  satisfies metadata, but a preinstalled/overridden older Node cannot inherit that conclusion.

## U — evidence limits and handoff

- Two-file freeze contains no raw run logs/runtime receipt; elapsed passes are attributed
  to coordinator records. No reviewer runtime, browser/Webpack bundling, sharp/PTY native-addon
  execution, all-platform Native, source-mode or GUI pass implied by stdio/HTTP Node.
- Whole-report change/ablation review complete; no temporary files or owned processes remain.
- **REVIEW READY — scoped lock adoption readiness only; four-file production adoption remains gated.**
