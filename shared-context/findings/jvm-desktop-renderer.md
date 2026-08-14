# JVM Desktop Renderer

- `Kodex/app/desktop/` is the standalone Compose Desktop host.
- JVM view code lives in `desktopMain`; native TUI code remains in `mosaicMain`.
- The TUI renderer is the canonical source for layout, copy, ordering, and behavior.
- Desktop renderers reuse existing contracts, ViewModels, and shared presentation models.
- MD3E controls only translate TUI interactions to desktop-native controls.
- Custom popup content uses Material 3 `BasicAlertDialog` through `DesktopModal`; do not use `DialogWindow` unless a real platform window is intended.
- The Desktop host resolves light or dark appearance before calling `KodexDesktopTheme`.
- Skiko is the primary system-theme source; when it reports `UNKNOWN`, the host falls back to GNOME, KDE, macOS, or Windows settings and observes later changes.
- Desktop colors use `MaterialTheme.colorScheme`; the application root paints the theme background.
- Desktop history renders committed, pending, and streaming domain events with the same semantic summaries, statuses, inline plan, and nested detail groups as the TUI; raw `toString()` is not a transcript presentation.
- Desktop history keeps viewport, follow-tail intent, and expansion state per `(Session index, Agent id)` outside shared ViewModels.
- The Desktop list declares history in visual chronological order so Compose focus traversal naturally continues through nested controls and then the composer; stable keys preserve reading anchors.
- Desktop committed-history items capture entry identity and generation from one immutable window snapshot; each item owns and lifecycle-registers its `FocusRequester`.
- Every committed multiline history item remains one focusable secondary-action surface without a focus background; right click, `Shift+F10`, and Menu open its actions.
- Desktop `PageUp` and `PageDown` move half a viewport and then focus the fully visible committed item at the destination edge.
- Validate with `:app-desktop:test`, shared ViewModel `jvmTest` tasks, `:app-desktop:run`, and `:app-desktop:packageDeb`.
