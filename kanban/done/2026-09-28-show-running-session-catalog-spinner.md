# Task Tree

- [done] `追踪目录运行标记与渲染链路`()
- [done] `在运行中目录行显示动画 spinner`()
- [done] `验证快照语义、截断和原菜单交互`()

# Details

- 用户发现前后端分离后的 Sessions 目录尚未用后端返回的 `SessionCatalogEntry.running` 显示运行 spinner。
- [Session 目录设计](../../checklist/rpc-session.md#目录与运行标记)规定 running 来自后端查询快照，不从前端 tab、`isActive` 或辅助状态推导，也不保证弹窗打开期间实时变化。
- 后端 `getSessionCatalog` 与前端目录状态已经保留 running；遗漏仅在 `SessionCatalogRow` 的 label 渲染。
- 复用现有标签页 spinner 帧与动画，不新增 RPC、目录 Flow 或自动刷新；确保运行项前缀在截断前计入宽度，空闲但 isActive 的条目不显示 spinner。
- 验收目录标签值、Mosaic 行动态帧与原上下文菜单测试；不改冻结 RPC contract，也不触碰并行 Native 调查文件。
- 实施结果：目录弹窗在任一查询快照条目 running=true 时启动本地 spinner 帧；每行只按自身 running 值决定是否显示，`isActive` 不参与判断。prefix 在标题截断前计入宽度，更新时间后缀样式不变。
- 验证：`:app-view-application:jvmTest` 101 项、`:app-view-application:linuxX64Test` 91 项均通过，包括新增的标签值与 Mosaic 动态帧测试；原上下文菜单测试仍在两端回归内。随后成功执行 `:app-cli:linkReleaseExecutableLinuxX64`，新产物为 `app/cli/build/bin/linuxX64/releaseExecutable/kodex-cli.kexe`，SHA-256 `0e6ce628fff7b2317bee113d3578efbcc2c12cc14a53e9a02fc0a354e9ea6285`。构建期间用户原 CLI 未停止，仍映射旧的已删除 inode；新启动的 CLI 将使用该路径上的新产物。未操作用户 Home。
