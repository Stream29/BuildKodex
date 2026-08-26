# Task Tree

- [done] 完善 Sessions 归档交互
  - [done] 复核当前标题与筛选布局
  - [done] 将 Show archived 移入标题栏
  - [done] 为 Session tab 增加 Close and archive
  - [done] 打开归档 Session 时自动取消归档
  - [done] 覆盖 frontend 与 Application 回归
  - [done] 运行完整验证并重建二进制

# Details

- 状态：`done`。
- 用户要求 `Show archived` 与 `Sessions` 位于同一标题栏，并靠标题栏右侧显示，不再独占第二行。
- 保持现有筛选状态所有权、复选框交互和归档右键菜单行为不变。
- 仅调整 Sessions catalog frontend 布局及对应回归覆盖。
- 标题与复选框现由同一满宽 Row 渲染，使用 `Arrangement.SpaceBetween` 分别贴靠左右两端。
- 新增标题栏生产组件测试，验证单行布局、固定宽度右边界和复选框切换；定向 JVM 测试通过。
- 用户追加要求：持久化 Session tab 的右键菜单增加 `Close and archive`；通过 Application 打开已归档 Session 时自动 Unarchive。
- `Close and archive` 只适用于持久化 Session，Archive 成功后再关闭 exact tab；失败时不提前关闭。
- Application 继续负责组合导航与归档命令；registry 负责在打开或复用持久化 Session handle 前幂等 Unarchive。
- root repository 增加按 index 获取 actionable entry 的精确入口，避免 Archive/Unarchive 为定位目标而扫描完整 catalog metadata。
- 受影响模块的 JVM 与 Linux Native 测试各通过 116 项，均为 0 failures、0 errors。
- IntelliJ 增量构建和 `git diff --check` 通过。
- Linux x86-64 release 二进制已重建：`Kodex/app/cli/build/bin/linuxX64/releaseExecutable/kodex-cli.kexe`。
- SHA-256：`65282b46d6e4511efd4e89d2710d725a1c8c0225f06d2063f2876039c2412feb`。
