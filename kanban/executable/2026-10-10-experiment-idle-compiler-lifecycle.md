# Task Tree

- `Inspect actual pinned daemon option and graceful shutdown`() // complete
- `Measure private idle shutdown during complete GUI import`() // complete; normal exit observed, GUI still blocked
- **`Verify active compilation and subsequent compiler reconnect`()** // suspended pending complete GUI
- `Require complete import and navigation before adoption`()

# Details

- Parent: [CInterop experiment](2026-10-10-experiment-kgp-cinterop-graph-lifetime.md).
  Continues the isolated repair; production remains unchanged.
- R17/R18 complete the Gradle build action but hit the original host-memory
  guard before external import success. The compiler process still holds
  about1.1GiB RSS with little recent CPU activity. This is not proof that its
  lifetime is the sole remaining cause.
- Test only private `-Dkotlin.daemon.options=autoshutdownIdleSeconds=60`.
  Actual pinned2.4.0 `CompilerSystemProperties` and `configureDaemonOptions`
  read this property. The documented `kotlin.daemon.jvm.options` example is
  not used as evidence for the pinned implementation.
- `lastUsedSeconds` reports current time during active read/write compilation
  locks. Idle shutdown enters `LastSession`, waits for sessions and schedules
  delayed termination with activity counters checked again. Static inspection
  does not replace active-compilation and reconnect gates.
- No external compiler termination during measurement. Cleanup happens only
  after the result is captured. Keep Gradle4GiB, IDE3GiB, compiler2GiB,
  daemon strategy, fallback disabled, all targets and4GiB host reserve.
- New private harness records the policy explicitly. Seed only the actual
  Native2.4.0 compiler and unchanged dependencies; do not delete the original
  multi-version seed or change any target. Archive R18 privately with SHA
  read-back verification before removing its redundant raw heap.
- Four genuine imports, external success and smart-mode evidence remain the
  gate. Navigation is not implied by a smart screenshot or generated script.
- R19 fails during initial IDEA/Tooling connection setup: two external
  resolves start44ms apart, then `Cannot use connection ... as it has been
  stopped`. No compiler process is observed and no Gradle action begins.
  Keep the failure; it does not test the idle policy. After verified cleanup,
  permit one unchanged-input repeat, not a success claim or suppressed error.
- R20 starts one resolve. Actual compiler command option is60s; its process
  disappears before cleanup, and the owned daemon log records idle timeout,
  graceful shutdown and shutdown started. Compilation/Gradle action completes.
  This supports the lifetime effect, not the active-compilation/reconnect gate.
- R20 still stops at the unchanged host reserve before external success.
  Peak RSS is Gradle4,985,004KiB and IDEA4,683,724KiB; the compiler's
  approximately1.1GiB exit is insufficient. No captured Gradle heap OOM.
  Continue the [committed-heap diagnostic](../done/2026-10-10-experiment-committed-heap-reclamation.md).
