# Task Tree

- `Collect crash and session evidence`()
- `Symbolize the faulting instruction`()
- `Run isolated launch and focus checks`()
- `Record attribution limits and follow-up`()

# Details

- User requested deeper investigation and a stable reproducer before filing an upstream issue or attempting a fix.
- Result: one confirmed 2026-09-28 native segfault, but neither three clean-home launches nor five focus-transition runs reproduced it. A single faulting GC-barrier instruction without a caller stack does not justify upstream attribution.
- Evidence, exact timings, negative checks, and next diagnostic prerequisite: [native segfault finding](../../shared-context/findings/kodex-native-segfault-2026-09-28.md).
- No code change, upstream report, or actual laptop suspend was performed.
