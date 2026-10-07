# Task Tree

- `Record the failed Java21 JNI gate and confirm Win32 return semantics`()
- `Obtain authorization to repair the fork and update its immutable pin`()
- `Repair only the inverted console-resize result and add focused regression tests`()
- `Validate targeted Windows Native tests and all three package-consumer hosts`()
- `Publish the new commit version only after every original gate passes`()
- `Hand off the verified pin to the default binary consumer`()

# Details

- Parent: [Fork CI](2026-10-07-implement-fork-package-ci.md).
- User explicitly approved this source correction on `kodex-submodule`,
  a new gitlink and new commit version; do not overwrite an existing package.
- [Mosaic run 37671576113](https://github.com/Stream29/Kodex/actions/runs/37671576113):
  all three producer stages/merge passed, Linux/macOS complete smoke passed.
  Windows JVM25 FFM and Native passed, but Java21 JNI `Jni.testResize` threw
  IOException 3. Writer was skipped; the old Mosaic version remains unpublished.
- Existing C helper returned success when `SetConsoleWindowInfo` was zero,
  and read stale GetLastError on nonzero success. The fix reverses that one
  predicate, preserving actual failures.
  [Microsoft return-value contract](https://learn.microsoft.com/en-us/windows/console/setconsolewindowinfo#return-value).
- Two Windows Native tests cover successful resizing with preexisting error3,
  actual size80x24, and invalid rectangle failure87 rather than stale error3.
  They own/close their real TestTerminal; no new native API, wrapper or mock
  library is added. Keep the original Java21/JVM25 and Native smoke gates.
- Fork upstream/trunk remains untouched. Coordinator alone owns Git/push and
  producer test command; standard Windows CI runs real tests. No local runtime
  or VM is used while the user is gaming.
- Current known packages: MCP/Lucene exact remote audit at 453/453 and 133/133;
  do not rebuild or overwrite these immutable versions with the new recipe.
- Fork correction signed/pushed as `78f94c4c2e136d908b838f09cd3231dc37ee67fd`,
  tree `0c9b20c8d4da1d905e8c2121708e28877ca7e168`; upstream/trunk unchanged.
  The new version is `0.19.0-SNAPSHOT-kodex.78f94c4c2e13`.
- Xiaoxin offline receipt/admission suite passed 78 cases. Windows producer
  will require exactly two real successful Native test results before sealing.
  [Independent narrow review](../done/2026-10-08-recheck-mosaic-windows-resize-gate.md)
  examines the fixed five-file snapshot before controlled CI dispatch.
- Kodex signed/pushed pin-and-gate commit:
  `49d78d99069104baa777ac5fd8a7db569fab8e8b`.
  [Automatic push 37677564146](https://github.com/Stream29/Kodex/actions/runs/37677564146)
  passed offline tests; all publishing stages were correctly skipped because
  main is unprotected. This is not a package-build or publication success.
- Independent review is READY with no confirmed static B1. Controlled main
  dispatch [37678581820](https://github.com/Stream29/Kodex/actions/runs/37678581820)
  is running at exact `49d78d99069104baa777ac5fd8a7db569fab8e8b`; real Windows
  tests, all original consumer gates and remote verification remain pending.
- Current run's three real producer stages and merge passed. Downloaded Windows
  `toolchain.json` contains `windowsResizeRegression.passed=2`, both expected
  real Native methods and `:mosaic-tty:mingwX64Test`; the enforced XML gate
  therefore passed, including actual success and invalid-rectangle error87.
  The three binary consumer jobs and writer still need to finish.
- All three consumer jobs now passed, including Windows Java21 JNI, Java25 FFM
  and host Native. The writer is running at the same exact main/fork identity.
  Keep default consumer unchanged until its final remote verification succeeds.
- Final result: the run succeeded, including **803 exact remote files** and
  manifest digest `c3a84f667256ec29ce70f49d873fbf71511e9417f8209c7cf83bcbfdaa53f88c`.
  Original failed runs remain unchanged; no gate was weakened or version
  overwritten. Scoped repair/publication is complete; whole-Kodex integration
  continues in the [consumer task](2026-10-07-consume-verified-fork-packages-by-default.md).
