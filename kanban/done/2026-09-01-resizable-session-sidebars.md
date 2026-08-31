# Task Tree

- [done] 实现可拖动并可配置的会话侧栏宽度
  - [done] 审查现有工作树和实现边界
  - [done] 将左右侧栏宽度加入全局设置
    - [done] 持久化左右宽度
    - [done] 在 Global Settings 中公开字段
  - [done] 实现左右 splitter
    - [done] 保留空闲态侧栏背景
    - [done] 使用 MD3 hover 和 active 状态色
    - [done] 拖动时约束侧栏与主内容宽度
    - [done] 松开时持久化最终宽度
  - [done] 补充设置、布局和指针交互测试
  - [done] 运行相关检查
  - [done] 构建当前平台 CLI 二进制

# Details

- 用户已授权审查后在无阻塞点时直接实现并构建二进制。
- 左右侧栏宽度是独立的全局设置，默认值保持 28 列。
- Splitter 占侧栏内侧一列，不显示字符。
- 空闲态不绘制 splitter 背景；hover 使用 `onSurface` 8% 状态层；按下和拖动使用 16%。
- 拖动期间只更新内存宽度，释放时持久化最终值。
- 保留 `Shift+Drag` 的终端原生文本选择行为。
- 工作树包含正在进行的 History Index 侧栏修改；实现必须保留这些改动。
- 相关 JVM 测试模块全部通过；侧栏模块最终回归通过 64 个测试。
- Linux x64 release CLI 构建成功：`Kodex/app/cli/build/bin/linuxX64/releaseExecutable/kodex-cli.kexe`。
