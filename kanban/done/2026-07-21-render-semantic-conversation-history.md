# Task Tree

- [done] 语义化渲染对话历史
  - [done] 对齐Codex对原始历史的投影和工具调用配对逻辑
  - [done] 建立弱化storage index的展示模型
  - [done] 分类型渲染用户、助手、推理、工具调用和结果
  - [done] 为可展开内容加入键盘、焦点和点击折叠
  - [done] 覆盖流式更新、工具配对和折叠状态测试

# Details

展示层不能继续直接打印原始history index。已完成的工具调用和结果作为一个语义单元展示，待完成调用保留明确的未完成状态。

CLI展示层现按协议item id和call id维持稳定身份；并行工具结果按call id原位配对，未匹配结果单独保留。推理摘要和工具详情默认折叠，可通过焦点、Enter、Space和鼠标切换。Linux与macOS测试通过。
