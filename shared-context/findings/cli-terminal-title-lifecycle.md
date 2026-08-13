# CLI Terminal Title 生命周期

- Mosaic 的 Ctrl-C 路径会取消 composition job，不保证执行根 `DisposableEffect.onDispose`。
- title 计数更新保留在 [`TerminalTitle.kt:38-54`](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/TerminalTitle.kt#L38-L54)，只按计数变化写入 OSC 0。
- 退出清理必须放在包裹 `runMosaic` 的 [`Main.kt:43-54`](../../Kodex/app/cli/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/Main.kt#L43-L54) `finally`，不能依赖 composition dispose。
- Linux PTY smoke test 已验证初始 title、Ctrl-C 正常退出和一次空 OSC 0 清理。
