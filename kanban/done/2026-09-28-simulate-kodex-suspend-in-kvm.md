# Task Tree

- `Check Xiaoxin KVM capacity and access`()
- `Boot isolated low-resource Ubuntu guest`()
- `Run original binary across suspend cycles`()
- `Verify evidence and release resources`()

# Details

- User suspected suspend and requested a KVM-based simulation on Xiaoxin Ubuntu.
- Result: one guest s2idle attempt could not be woken reliably and was not counted as a completed cycle. Seven guest ACPI S3 (`deep`) suspend/wakeup cycles completed with the original release binary alive and no segfault; two cycles ran Kodex inside the guest's `user.slice`.
- The host crash preceded kernel suspend entry, so resumed execution could not cause this particular crash. The pre-suspend/lid-close sequence remains plausible, as does an unrelated concurrent bug.
- Full setup, observations, and limitations: [native segfault finding](../../shared-context/findings/kodex-native-segfault-2026-09-28.md).
- The temporary VM and all test files were removed; the 22 packages installed on Xiaoxin solely for this experiment were purged. No host suspend was attempted.
