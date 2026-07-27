# Task Tree

- [done] 补全session设置页面
  - [done] 在current session页面接入model、reasoning、service tier和mode
  - [done] 在new session页面接入对应初始设置
  - [done] 复用现有模型目录约束候选项
  - [done] 接入设置提交语义并补齐交互测试

# Details

本任务只接入已有且含义明确的设置，不扩展新的配置项。

当前会话页面提交到ViewModel暂存层，新会话页面提交到内存默认配置；两者均复用模型目录候选项和校验。Linux与macOS原生平台各通过33项CLI测试。
