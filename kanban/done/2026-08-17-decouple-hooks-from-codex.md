# Task Tree

- 将 Hooks 重构为 Kodex 原生模型
  - [done] 将全局配置改为有序名称映射
    - [done] 定义六种 Hook type
    - [done] 定义只含 type、command 的 body
    - [done] 使用名称作为稳定运行身份
  - [done] 简化 Hook 管理与设置界面
    - [done] 提供按名称添加、编辑、删除
    - [done] 删除 source、enable、environment 和 import UI
  - [done] 建立 Kodex 原生命令协议
    - [done] 输入使用名称、type、上下文和 payload
    - [done] 控制输出使用 type 专属 action
    - [done] 删除特殊退出码和 Codex wire 字段
  - [done] 迁移运行时解析与执行逻辑
    - [done] 控制型 Hook 按声明顺序串行
    - [done] 观察型 Hook 保持并发
    - [done] 删除 matcher 和位置身份
  - [done] 删除 Codex Hook 兼容层
    - [done] 删除 Codex Hook importer
    - [done] 删除 approval 预留端口
    - [done] 移除 codex-cli-storage 的 Hook 依赖
  - [done] 更新测试与持久化覆盖
    - [done] 覆盖名称映射管理语义
    - [done] 覆盖原生 wire 与控制结果
    - [done] 覆盖 settings YAML 结构
    - [done] 更新集成测试断言
  - [done] 更新 Hooks 相关决策文档
  - [done] 运行格式化和回归验证

# Details

- 状态：`done`。
- 用户要求直接执行重构，并保留工作区内其他既有修改。
- 配置结构固定为 `hook name -> { type, command }`。
- Hook 名称是全局唯一稳定身份；type 是 `pre_tool_use` 等触发边界。
- 同一 type 可以由多个不同名称的 Hook 使用。
- 不保留 matcher；命令需要筛选事件时自行读取 stdin。
- Hook body 不保留 timeout、environment、enable、status message 或 additional context limit。
- 不保留 Codex Hook 导入和 Codex wire 兼容。
- 支持 `pre_tool_use`、`post_tool_use`、`user_prompt_submit`、`stop`、`pre_compact`和`post_compact`。
- 同 type 的控制型 Hook 按配置声明顺序执行；pre tool use 和 user prompt submit 在第一个 block 处结束，stop 在第一个 continue 或 stop 处结束。
- `post_tool_use`、`pre_compact`和`post_compact`作为观察型 Hook 并发执行并忽略输出。
- 命令 stdin 使用统一 Kodex envelope；事件数据置于`payload`。
- 非零退出、超时、启动失败和非法输出继续 fail-open，不再赋予退出码2特殊含义。
- 旧 Hooks 设置不迁移；新的无版本 settings YAML 直接使用名称映射。
