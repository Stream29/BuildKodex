# Task Tree

- [done] 修复 Desktop 历史焦点请求器竞态
  - [done] 补充窗口替换回归测试
  - [done] 统一 Lazy item 窗口快照
  - [done] 按条目生命周期管理焦点请求器
  - [done] 运行 Desktop 历史定向验证

# Details

- `LazyColumn.items` 延迟组合时会混用已捕获的条目列表与实时 history generation。
- 父级组合提前裁剪 `FocusRequester` 映射，旧条目随后被测量时会在 `checkNotNull` 处崩溃。
- 修复范围限定为 Desktop history renderer 及其回归测试。
- 使用一致的窗口快照构造条目 identity，并由条目内的 `remember` 与 `DisposableEffect` 注册焦点请求器。
- 回归测试覆盖 generation 替换时的 requester 注销、重新注册与焦点恢复。
- `:app-view-history:jvmTest` 通过 7 项测试；`:app-desktop:test` 通过 11 项测试。
- Desktop 启动后持续进入事件循环至 30 秒 smoke timeout，未记录对应组合异常。
- 编译后的 committed-item lambda 不再捕获实时 window delegate，也不再包含空值强制检查。
