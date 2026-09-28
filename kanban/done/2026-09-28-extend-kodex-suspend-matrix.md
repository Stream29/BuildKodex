# Task Tree

- `Rebuild isolated KVM guest with original binary`()
- `Test suspend under resident memory pressure`()
- `Mock real CLI SSE request without credentials`()
- `Combine streaming, pressure, and suspend`()
- `Verify kernel results and remove resources`()

# Details

- User requested more trials covering low-memory and live streaming possibilities.
- The old CLI has a fixed ChatGPT base URL, so a guest-only hosts override, temporary CA, local HTTPS mock, and fake auth file supplied a real `/responses` SSE stream without using the user's account.
- In a 2 GiB guest, 8 cycles at 1,200 MiB resident pressure, 8 at 1,400 MiB, 8 during SSE alone, and 8 with SSE plus 1,400 MiB pressure all completed. The mock's emitted-frame count increased after every wake; kernel journal recorded 32 `deep` entries and exits and no OOM or segfault.
- Exact scope and remaining s2idle limitation: [native segfault finding](../../shared-context/findings/kodex-native-segfault-2026-09-28.md).
- The guest and temporary packages/files were removed. No real Home, credentials, external model, or host suspend was involved.
