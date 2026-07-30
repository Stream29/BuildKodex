# Task Tree

- [done] 将 CLI 顶栏改为可管理的 Session / New session 标签栏
  - [done] 盘点现有 Session repository、NewSessionViewModel 和标签交互边界
  - [done] 建立应用级标签目标：多个未物化 New session 与已打开的历史 Session
  - [done] 支持创建、选择、物化和关闭标签，关闭不删除持久化历史 Session
  - [done] 将顶栏改为 `Sessions | New session… | +`，并为当前 Session 标签提供关闭菜单
  - [done] 按活动标签路由 Agent runtime screen、New session screen、侧栏和设置
  - [done] 补充多 New session、历史 Session 切换和关闭标签的适用测试并验证

# Details

- 用户明确期望：启动时显示 `Sessions | New session | +`；可多次 `+` 创建并保留多个未物化 New session；Sessions 打开历史 Session；单击标签切换；当前 Session 标签再次单击打开菜单并可关闭。
- 交互实现假设：单击非当前标签切换，单击当前真实 Session 标签打开其下拉菜单；关闭只关闭前端打开实例，不删除持久化 Session。
- 此需求取代 `checklist/cli-session-view-models.md` 中“只支持唯一 NewSessionViewModel”的旧产品约束。
- 已验证：`./gradlew :cli-app:linuxX64Test :cli-session:linuxX64Test --console=plain --no-configuration-cache -x :cli-history:compileKotlinLinuxX64 -x :cli-history:linuxX64MainKlibrary` 成功；它复用了此前成功构建的 cli-history 产物，并编译、链接、执行了 CLI app 测试。
- 未跳过 cli-history 的完整命令目前会在工作区既有 `cli/history/.../CleanEventView.kt:682` 失败，报错为泛型类型推断，未修改该文件。
