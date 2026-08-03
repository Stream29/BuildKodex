# Task Tree

- [done] 纯拆分现有共享应用层与 CLI renderer
  - [done] 盘点现有文件与依赖边界
    - [done] 按 application runtime、共享 frontend state 与 CLI renderer 分类现有代码
    - [done] 识别随实现迁移的 common tests 与 Mosaic tests
    - [done] 固定最小 visibility、package 与构建调整
  - [done] 建立 `app/shared` 共享模块族
    - [done] 移入 application factory、DI 装配、资源所有权与 shutdown 生命周期
    - [done] 移入 application、Session、Agent、NewSession 与 history ViewModel
    - [done] 移入 composer、request-user-input、settings、auth 与 title 等无 UI 状态和服务
    - [done] 保持现有公开 state、effect、command 与行为不变
  - [done] 将现有 TUI 收敛到 `app/cli`
    - [done] 保留 native CLI entrypoint 与 Mosaic application host
    - [done] 移入 Mosaic screen、terminal component 与终端专用 renderer
    - [done] 保持 frontend-local interaction state 由 CLI renderer 所有
  - [done] 清理 Gradle 依赖方向
    - [done] 让 `app/cli` 单向依赖 `app/shared`
    - [done] 移除共享 JVM 产物对 `mosaicMain` 与终端组件的传递依赖
    - [done] 保持共享层支持现有 JVM 与 Native targets
  - [done] 验证纯拆分不改变现有行为
    - [done] 运行共享 ViewModel 与 application tests
    - [done] 运行 Mosaic component、renderer 与 CLI interaction tests
    - [done] 验证 JVM 编译与适用 Native executable 链接
    - [done] 检查共享模块不导入 Mosaic、终端 UI 或 Compose Desktop UI 类型

# Details

- 状态：`done`。
- 本任务只移动现有实现和测试，不设计或创建 Desktop。
- 不改变 runtime、storage、tool、MCP、hook、auth、配置、Session 或 ViewModel 行为。
- `app/shared` 表示按既有职责拆分的共享模块族，不合并成单一大模块。
- `app/cli` 是 Mosaic renderer、终端交互和 native entrypoint 的所有者。
- 现有 common implementation 与 common tests 按职责移入 `app/shared/*`；Mosaic implementation 与 Mosaic tests 移入对应的 `app/cli/*`。
- `CoroutineFailureLogging.kt`、`RootSessionRendering.kt` 与 `AgentRuntimeRenderState.label()` 属于 CLI；其余无 UI 状态与转换留在 shared。
- 本轮保留现有 Kotlin package，只为跨模块调用提升最小 visibility；package 重命名不混入纯拆分。
- 实施时避开当前用户 staged-delete 的旧 package 路径，不恢复或改写这些文件。
- 不创建 Git commit。
- 已将无 UI 框架依赖的实现和 common tests 移入 `Kodex/app/shared/*`，将 Mosaic renderer、native entrypoint 和 Mosaic tests 移入 `Kodex/app/cli/*`。
- 已验证 shared 不依赖 CLI，也不导入 Mosaic、终端 UI 或 Compose Desktop UI 类型。
- 已通过 shared application、CLI application、CLI session 与 CLI new-session 的 JVM 编译。
- 已通过 12 个 shared JVM test task 与 6 个 CLI JVM test task。
- 已通过 `:app-cli-application:linkDebugExecutableLinuxX64`。
- JVM CLI 验证排除了 Mosaic 既有的 JDK 22 binding 编译任务；这些任务缺少生成 binding classpath，与本次目录和模块拆分无关。
