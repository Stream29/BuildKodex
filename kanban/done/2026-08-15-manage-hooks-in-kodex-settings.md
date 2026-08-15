# Task Tree

- [done] 在Kodex Settings中独立管理Hooks
  - [done] 将运行时Hook配置模型收归`hook:contract`
    - [done] 删除`hook:contract`对Codex CLI storage模型的依赖
    - [done] 保留事件、matcher、命令、顺序、enable和执行环境语义
  - [done] 移除Codex Home与项目`.codex` Hook配置的自动读取和继承
  - [done] 让`KodexGlobalSettings.hooks`成为唯一运行时配置来源
  - [done] 建立Hooks管理命令与状态边界
    - [done] 支持添加、编辑、删除、启停和原子持久化
    - [done] 只向Settings UI发布脱敏配置摘要
  - [done] 实现Kodex自有的Hooks设置管理界面
  - [done] 提供带preview与filter的显式`Import from Codex`
    - [done] 以每个`hooks.json`或含Hooks的`config.toml`配置源为最小导入单元
    - [done] 生成脱敏preview并区分新增、冲突、部分支持与不支持源
    - [done] 支持筛选配置源并逐项选择跳过或整体替换
    - [done] 只将确认的受支持内容一次原子持久化为Kodex配置
  - [done] 为Kodex Hook源定义稳定ID和可选Codex导入来源身份
    - [done] 以来源类型与规范化源路径识别同一Codex导入源
    - [done] 重新导入冲突时保留Kodex ID并整体替换该源内容
    - [done] 不让Codex来源身份参与运行时文件读取或同步
  - [done] 迁移现有settings schema和Hook runtime消费方
  - [done] 覆盖管理、导入、持久化、刷新和Hook执行回归验证

# Details

- 状态：`done`。
- Kodex不再自动读取、继承或同步Codex Hook配置。
- Hooks由Kodex自己的Settings管理，并持久化到`KodexGlobalSettings.hooks`。
- `Import from Codex`由用户显式触发，只将当时的Codex Hook配置复制为Kodex自有配置。
- 导入必须先显示不修改设置的脱敏preview，并允许用户筛选和逐项选择；交互原则与MCP import计划一致。
- Hooks导入按配置源操作；一个Codex源可以包含多个事件、matcher组和有序handler，不能在导入时拆散这些运行语义。
- 同一来源类型与规范化路径构成Codex导入来源身份；已有来源冲突只允许跳过或整体替换，替换保留既有Kodex稳定ID。
- 部分支持的来源在preview中明确列出被排除内容，只持久化Kodex支持的命令Hook；完全不支持的来源不可选择。
