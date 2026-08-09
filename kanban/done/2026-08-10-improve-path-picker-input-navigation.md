# Task Tree

- [done] 改善 path picker 的鼠标侧键与输入过滤交互
  - [done] 调研鼠标后退侧键映射到上级目录
  - [done] 调研字母输入触发过滤并预选首项
  - [done] 分离 `TuiDialog` 的 Escape 与普通 dismiss 回调
  - [done] 让 button 8 复用可用状态下的 `Up` 操作
  - [done] 增加目录名子串过滤、过滤提示和编辑按键
  - [done] 在过滤变化后聚焦首个结果并在目录切换时清空
  - [done] 补充 dialog 与 directory picker 交互测试
  - [done] 更新持久交互约束并运行定向验证

# Details

- 状态：实现、文档和定向验证已完成。
- 改动前 picker 没有过滤状态或弹窗级字符处理；目录行获得焦点后，Enter 会进入该目录。
- Kodex 已启用 `MouseTracking.AnyEvents`。Mosaic 已公开 `Button8` 至 `Button11`，并能解析 xterm 扩展按钮编码 `128` 至 `131`，不需要修改终端输入层。
- 当前实际环境为 foot 1.25.0。鼠标设备提供 `BTN_SIDE` 与 `BTN_EXTRA`；foot 将 `BTN_SIDE` 编码为 xterm button 8，因此当前环境的后退侧键应对应 `MouseEvent.Button.Button8`。
- 扩展按钮编号本身不携带跨终端的后退语义；其他终端或经过 tmux 时仍需实际验证。
- 后退侧键仅在指针位于 picker 弹窗内时生效，并复用 `Up` 的同一语义操作；仅在父目录存在且当前不处于加载状态时执行。
- 过滤采用忽略大小写的子串匹配。
- 没有 Ctrl/Alt 的字母输入进入过滤模式；Shift 只影响字母大小写。界面需要显示当前过滤词。
- 过滤词非空时，每次变化后列表回到开头并程序化聚焦首个匹配目录；可复用 popup menu 已有的下一帧聚焦模式。Enter 保持现有语义，进入该目录。
- Backspace 编辑过滤词；过滤模式下第一次 Escape 清空过滤，过滤为空时 Escape 才关闭 picker。
- 进入子目录或返回上级时清空过滤。
- 精确区分 Escape 与弹窗外点击需要让 `TuiDialog` 分离 Escape 回调和普通 dismiss 回调；改动前两者共用 `onDismissRequest`。
- 过滤与焦点行为属于 CLI picker；共享 filesystem browser 无需变更。
- `TuiDialog` 已提供默认委托到普通 dismiss 的独立 Escape 回调；directory picker 用它实现过滤优先清空。
- directory picker 已让 button 8 与 `Up` 复用导航操作，并实现过滤提示、忽略大小写的子串过滤、首项聚焦、Backspace 编辑和导航清空。
- 已补充 dialog 回调分离、button 8、过滤聚焦与 Enter、Backspace 和 Escape 顺序的交互测试。
- IntelliJ reformat、warning 级 lint 和指定文件 build 均完成；IDE build 成功但仅提供有限诊断。
- `:app-cli-components:linuxX64Test :app-cli-path-picker:linuxX64Test` 通过。
- JVM 定向测试在测试编译前被 Mosaic 既有的 `PlatformKt` 重复类名错误阻塞；报错文件位于 Mosaic runtime 的 `commonMain` 与 `jvmMain`，不属于本次改动。
