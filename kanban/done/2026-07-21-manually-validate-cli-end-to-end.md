# Task Tree

- [done] 手工端到端验收Kotlin/Native Codex CLI
  - [done] 在其他CLI实现与清理任务完成后构建最新Kotlin/Native可执行文件
  - [done] 通过真实PTY或tmux启动CLI并使用真实鉴权与API
  - [done] 验证新建、切换、fork、revert和多轮会话链路
  - [done] 验证模型、reasoning、service tier、mode和settings链路
  - [done] 验证多行输入、中文输入、流式输出、工具调用和错误恢复
  - [done] 验证键盘、鼠标、焦点、弹层、滚动、终端尺寸变化和退出链路
  - [done] 修复发现的问题并重新执行受影响链路

# Details

该任务使用真实tmux、Codex鉴权与Responses API完成，不以单元测试替代手工操作。

验收期间修复了`image_gen.imagegen`与`view_image`将base64误作文本结果的问题。两者现均返回Responses原生`input_image`内容，图片生成结果由CLI宿主持久化到Codex home。

取消流式响应后可以继续发送用户消息。长历史已验证PageUp、PageDown和鼠标滚轮；终端缩放、弹层焦点、鼠标状态、空闲退出与流式中退出均无异常堆栈。真实工具链覆盖tool search、update plan、apply patch、view image、image generation、web run、current time、shell session、write stdin和context remaining。
