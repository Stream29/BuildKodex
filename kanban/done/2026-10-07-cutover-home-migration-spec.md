# Task Tree

- `Pin frozen source and fixture paths against the accepted baseline`()
- `Move nonfrozen contracts and cut over the real Home handle`()
- `Preserve current prepare and version-generation algorithms`()
- `Add actual lifecycle/static-factory evidence and hand off`()
- `Receive independent scoped acceptance with the parent batch`()

# Details

- [主任务](2026-10-07-cutover-ui-foundations-and-home-bootstrap.md)；用户已授权实施。
- Own 旧 migration/contract source/test、new migration/spec source/test/build，
  现有 migration/impl 非冻结通用启动源/tests；不修改 impl/build，主协调者 retarget。
- 原 MigrationVersion/Migration/FQCN 保留，CoroutineFileSystem 依赖真实 spec。
  public 版本/layout 异常移入 spec。原 KodexHomeHandle 的 name/API 是唯一资源契约；
  当前持 lease 实现直接实现它，原 prepareKodexHome 返回接口，不留旧 public class。
- impl 保留原 close/closeAndJoin、read→write→read lease/exception/cleanup 算法；
  仅给真实操作补规范 KDoc，不把基线不保证的取消/幂等性伪称已实现。
- CurrentKodexApplicationVersion/GeneratedKodexApplicationVersion 与原生成任务仍在 impl；
  不新增 provider、runner、manager、DTO 或第二 registry。
- 所有 `v*` frozen source/private codec/fixture（含 future entry，本批都保留）
  字节与路径不变；KodexHomeMigrations registry 不变，原版 tests 不借迁移重写。
- 原 generic MigrationVersion tests 随 spec 迁移；可追加独立 unversioned lifecycle suite
  验证真实工厂静态类型、owner cancellation、cleanup、repeat close，与真实 isolated FS。
- 不碰 CLI/Application/Gradle shared sources，主协调者集成；不运行 Gradle/IDE、
  提交/推送。早期 handoff 列精确资源/factory/error/dependency 与旧声明删除闭包。

## Integration handoff

### Coordinator closure

- Final spec 2 / impl 70 JVM cases passed, including 11 actual handle-lifecycle
  tests and frozen migration suites. JVM/Native consumers and current CLI passed.
- First new cleanup-characterization assertion required accounting for JVM
  coroutine stack-trace recovery: the background exception's cause chain retains
  the injected cleanup instance. Original primary identity and other assertions
  remain; no cleanup operation changed.
- Two isolated CLI runs exited 0; the second reopened the same prepared Home and
  persisted Session. Home/Session locks cleared normally after both runs.
- All 16 frozen files including registry compare byte-identical at the original
  paths. Generated version source/path and project version remain unchanged.
- Both baseline cancellation/cleanup defects below were actually reproduced,
  not fixed. Full evidence and unverified platforms:
  [parent](2026-10-07-cutover-ui-foundations-and-home-bootstrap.md).
  Independent scoped acceptance passed under the user's conditional commit
  authorization; Home impl's 70 cases were freshly rerun. Local commits only.

- Historical worker handoff (superseded by the coordinator closure above):
  - New `:app-migration-spec` uses `kodex.kmp-host` and
    `api(project(":utils-kotlinx-io-coroutines-spec"))`; coordinator retargets
    impl and other consumers and deletes the old contract build entry.
    Impl needs the actual `:utils-kotlinx-io-coroutines-impl` runtime for
    `SystemCoroutineFileSystem`; it remains available through the lease impl's
    `api` dependency. An explicit direct implementation dependency also makes
    the startup source's composition dependency clear.
  - `io.github.stream29.kodex.app.migration.KodexHomeHandle` becomes the unique
    `AutoCloseable` interface: `home: Path`, `version: MigrationVersion`,
    `close()`, `suspend closeAndJoin()`.
  - Internal `HeldLeaseKodexHomeHandle(home: Path, version: MigrationVersion,
    lease: FileSystemLease)` directly implements that interface.
  - `MigrationVersion`, `Migration`, `KodexHomeVersionException(message, cause =
    null)` and `KodexHomeLayoutException(message, cause = null)` retain their
    original package and public signatures in spec.
  - Both `CoroutineScope.prepareKodexHome` overloads, the public factory's
    parameters/defaults, `CurrentKodexApplicationVersion` and the single
    generated application-version source remain in impl.
  - Home worker does not touch impl/build, registry, generated source, frozen
    paths, adapters, consumers or central validation resources.
- Source completion:
  - Old contract's two main declarations and its complete generic version test
    moved to spec; old source paths are absent. The original version test is
    byte-identical at its new path. Coordinator owns old build retirement.
  - Actual public exceptions moved without signature changes. The old public
    concrete handle is removed; both factory paths construct
    `HeldLeaseKodexHomeHandle` and statically return `KodexHomeHandle`.
  - Read→write→read preparation, registry selection, baseline/layout/version
    validation, generated-version access and close/join algorithms are unchanged.
  - Spec KDoc owns version/entry, real factory error conditions, retained-lease
    ownership, repeated closure and cancellable waiting semantics. No new factory,
    provider, adapter or runner was added.
  - New `KodexHomeHandleLifecycleTest.kt` contains 11 isolated real-filesystem
    tests: public factory static interface/AutoCloseable proof, release/restart,
    shared reference retention, owner cancellation, cancellable close/join,
    migration cancellation, real version/layout errors, and the two baseline
    characterization cases below. Existing generic startup tests are unchanged.
- Worker checks performed (not runtime validation):
  - `git diff --check -- app/migration` passed.
  - All 15 baseline `v*` source/test/codec/fixture/skill files compared at their
    original paths against `8297d388`: byte-identical, including future entry.
  - Registry and original unversioned startup tests compared byte-identical.
  - Declaration search found one public handle interface and one declaration
    each for the original versions/entry/errors; old concrete handle is absent.
  - No Gradle, IDE, CLI, commit/push, branch operation, device or temporary
    runtime resource used. Added tests have **not** been executed by this worker.
- Central validation: execute spec's moved `migrationVersionTest`, original
  `kodexHomeMigrationTest`, new `kodexHomeHandleLifecycleTest` and frozen suites;
  confirm the resolved graph and native CLI/static consumers after retarget.
- Baseline cancellation defect, separate from this cutover:
  - [Lease publication](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemLeaseImpl.kt#L90)
    writes the owner before constructing its owner-bound lease at line 95.
    Cancellation after the write but before return can strand a `.read.lock`,
    even after the factory operation and intended owner have been cancelled/joined.
  - [Deterministic reproducer](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonTest/kotlin/io/github/stream29/kodex/app/migration/KodexHomeHandleLifecycleTest.kt#L264)
    delegates to actual system FS, suspends only after owner bytes are published,
    cancels preparation, and asserts the orphan, released guard and unchanged
    version. Fixture cleanup removes the isolated orphan. Source inspection
    establishes the gap; central execution reproduced it, including the fresh reacceptance.
  - No acquisition, cancellation or owner protocol was rewritten. Fixing this
    preexisting defect requires separately scoped review/authorization.
- Baseline primary/cleanup characterization:
  - [Actual factory test](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonTest/kotlin/io/github/stream29/kodex/app/migration/KodexHomeHandleLifecycleTest.kt#L232)
    injects an action primary and write-owner deletion failure. Existing
    `Job.join()` preserves the action primary but does not rethrow the background
    cleanup failure or add it as suppressed; the test captures that failure via
    the owner's handler and checks the stranded write owner.
  - This is not a new primary/suppressed guarantee. Cleanup protocol improvement
    is outside this hard cutover; central execution confirmed this characterization.
