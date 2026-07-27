# Task Tree

- 移除AgentRuntime的turn admission抽象
  - 恢复只有`resume()`的AgentRuntime契约
  - 让各Runtime直接围绕`delegate.resume()`编排
  - 将用户消息与显式skill注入恢复为CLI侧AgentState操作
  - 重写Turn Hook与Session Hook的resume前后织入
  - 清理coordinator、测试和文档中的admission语义
  - 运行相关多平台测试

# Details

- 不引入OpenAI或Rust风格的turn runner/admission边界。
- 一次最外层`AgentRuntime.resume()`就是可组合的运行单元。
- Hook由对应Runtime在`resume()`前后织入，不通过回调控制内层Runtime。
- JVM、JS/Node与Linux/Native相关测试已通过。
