# Kodex native segfault, 2026-09-28

## Evidence

- At 06:04:59 Asia/Singapore, kernel journal recorded `kodex-cli[1388903]` faulting on read of `0x2c000000ae` at instruction `0x2495bdf`, with `error 4`.
- The [release binary](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/cli/build/bin/linuxX64/releaseExecutable/kodex-cli.kexe) was last modified on 2026-09-23 03:48 +08; the project declared Kotlin 2.4.0 at that time. `addr2line` maps `0x2495bdf` to Kotlin/Native's `beforeHeapRefUpdateSlowPath`; disassembly shows the faulting instruction reads an object-header pointer from `%rbx`. This identifies where the invalid reference was consumed, not where it originated.
- The last Kodex log entries before the crash show session 395's response and turn cancelled at 06:04:53 ([Kodex.log:41882-41883](file:///home/stream/.kodex/log/Kodex.log#L41882-L41883)). The lid closed at 06:04:56, the segfault occurred at 06:04:59, and the system suspended at 06:05:00. The user's recollection confirms closing the lid, but not whether they manually cancelled the response.
- No core dump or Apport report was found for this process. The current shell's core-size limit is `0`, but the crashed process's limit is unknown. There is no caller stack to attribute the invalid reference to Kodex, Mosaic, Kotlin/Native, or another native dependency.

## Isolated checks

- The same release binary started under three fresh temporary `HOME` directories and pseudo-terminals without segfaulting.
- Five more fresh-home runs each received five terminal focus-out/focus-in pairs followed by Ctrl+C. All five opened and closed normally; no new kernel segfault appeared. This tests only an idle UI focus transition, **not** an active agent cancellation or actual system suspend.
- Temporary homes and processes were removed after the checks. No real Kodex Home, source file, or live process was modified.

## KVM suspend simulation on Xiaoxin Ubuntu

- A temporary QEMU/KVM VM used an official Ubuntu 24.04 minimal cloud image verified against Canonical's SHA-256 list. The guest ran the **same release binary**, verified by SHA-256 `a37dbbbb393b16a22b664e2eceb4e471c38f9dca5dda0bfd81fda347a59f61f0`, with a fresh Home and no real credentials or session data.
- One guest `s2idle` attempt logged `PM: suspend entry (s2idle)` but did not complete a usable wakeup: QMP did not report `suspended`, `system_wakeup` refused, and an RTC alarm and virtual key did not restore SSH. The guest was reset. This attempt is **inconclusive** for post-resume behavior.
- The guest's default ACPI S3 (`deep`) completed seven suspend/wakeup cycles. Kernel journal showed seven `PM: suspend entry (deep)` / `PM: suspend exit` pairs, and [QEMU QMP](https://www.qemu.org/docs/master/interop/qemu-qmp-ref.html) reported `suspended` before `system_wakeup`. Kodex survived all seven cycles with no recorded segfault: five cycles in a system service and two in a persistent `user.slice` service.
- The idle CLI did not crash on that guest's S3 path. This does not test the original host's s2idle wakeup or its active agent turn, device/network teardown, terminal focus behavior, and lid-close timing. The original segfault was logged at 06:04:59, **before** the original host's `PM: suspend entry (s2idle)` at 06:05:00.
- The VM was shut down; its disk, seed, and copied binary were removed. All 22 packages installed solely for the experiment were purged. The Xiaoxin host itself was never suspended.

## Extended memory and streaming matrix

- A second temporary QEMU/KVM Ubuntu 24.04 guest ran the same SHA-256-verified release binary with 2 vCPUs, 2 GiB RAM, and a 3 GiB host-side VM memory limit. Kodex ran in `user.slice` throughout.
- A separate process committed 1,200 MiB of resident memory, leaving about 500 MiB guest `MemAvailable`; Kodex survived eight `deep` suspend/wakeup cycles. At 1,400 MiB committed, leaving about 300 MiB, it survived another eight. The guest had no swap, and no kernel OOM or segfault was recorded.
- For actual agent streaming without real credentials or paid calls, the guest alone mapped `chatgpt.com` to a local HTTPS mock with a temporary trusted certificate and fake Kodex auth file. The unchanged CLI sent a `/backend-api/codex/responses` POST; the mock emitted continuing SSE `response.output_text.delta` frames, and the terminal repeatedly redrew while Kodex logged an active response request.
- Eight more `deep` cycles ran during that live SSE stream without pressure. Eight additional cycles combined the live stream with 1,400 MiB committed pressure, leaving about 260–300 MiB guest `MemAvailable`. After **each** wake, the Kodex process remained alive and the mock's emitted-frame counter increased. Kernel journal showed 32 `deep` suspend entries and 32 exits across this second VM, with no segfault or OOM.
- The 32 negative trials cover real CLI request handling and streaming UI activity under constrained guest memory, but still use virtual ACPI S3 rather than the original laptop's s2idle, and a mock response rather than the original account and session state. They do not identify the source of the invalid reference.
- The mock, fake credentials, guest image, copied binary, and VM were removed; the 22 host packages installed solely for this experiment were purged. The Xiaoxin host was never suspended.

## Conclusion and next evidence

- There is one confirmed crash but no stable reproducer or proven upstream component. Do not file a Kotlin/Mosaic issue or apply a speculative fix on this evidence alone.
- If it recurs, capture a native core/backtrace and the immediately preceding input/lifecycle events before attributing ownership. The host's `/proc/sys/kernel/core_pattern` routes crashes to Apport, while the observed shell core-size limit was `0`; a future diagnostic launch must first arrange crash capture without changing the active user's process.
- Kotlin's [Native memory-management documentation](https://kotlinlang.org/docs/native-memory-manager.html) describes its concurrent GC, but does not establish that the GC caused this crash.
