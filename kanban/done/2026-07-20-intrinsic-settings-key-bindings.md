# Task Tree

- [done] 让 Settings dialog 按内容测量并重建输入键绑定
  - [done] 将 dialog 主体高度改为 intrinsic measurement
  - [done] 将换行键收敛为 Shift+Enter 与 Enter
  - [done] 建模并显示配对的 Enter 与 Ctrl+Enter 提交键
  - [done] 让编辑器按配对键分别换行和提交
  - [done] 更新全局设置与 TUI 交互测试
  - [done] 更新全局设置约束文档
  - [done] 运行相关跨平台验证

# Details

只允许两组无冲突绑定：`Shift+Enter -> Enter` 与 `Enter -> Ctrl+Enter`。提交键由换行键唯一确定，不能形成四种任意组合。

验证通过：JVM、Linux Native、Node.js 测试，Linux Native 可执行文件链接，以及 80×20 真实终端快照。
