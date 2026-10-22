# Task Tree

- `Migrate independent exported tool declarations with Session1005`()
- `Review subagent delivery within its directory ownership`()
- `Run centralized compilation tests and lint`()
- `Complete shared provider and remaining tool declarations with Session1014`()
- `Verify completed tool boundaries with centralized compilation tests and lint`()

# Details

- 上游：[导出 API 迁移](2026-10-20-audit-exported-api-spec-boundaries.md)。
- 已读取 [Session1005](file:///home/stream/.kodex/sessions/1005) 的部分迁移交付；
  子任务未构建，父 Session 首次编译发现 const 无初始化错误，修正后七个变更模块的
  第一轮 JVM 测试通过。最终七个变更模块的 metadata/JVM 编译与 75 项 JVM
  测试通过，包含三个编译期常量回归；CurrentTime JS、WebRun/UnifiedExec
  Linux X64 klib 编译通过，真实消费者及 integration-test 测试源码编译通过。
- 唯一写范围：`Kodex/tool/**`；其他目录及既有 26 模块改动保留。
- 按[源集规范](../../checklist/kmp-spec-impl-source-sets.md)分离工厂、客户端、
  函数、属性和成员，保持包名、默认参数、KDoc、序列化形状、资源及取消语义。
  inline/reified/contracts/const 等形状先核实约束，不能靠改变 API 语义过 lint。
- 子任务不提交推送、不改共享配置或任务记录、不执行 Gradle/编译器或操作用户进程；
  父 Session 统一审查并串行验证。跨范围/窄平台阻塞需精确记录，不以 stub 或隐藏 API 绕过。
- 首轮已迁移 core/current-time/get-context-remaining/web-run，部分迁移
  unified-exec/tool-search/image-generation。apply-patch、plan、view-image
  及部分 client 尚未处理；provider 依赖包括 patch、系统 IO、图片 codec、
  agent-state 扩展、search-index 的窄平台工厂和 system ShellClient。
  其中实现体调用 provider 是跨模块契约债务，不自动等于本地 expect/actual 不可编译；
  默认参数/签名所需 provider 声明则须先保证 spec 可见和目标闭合。
- 首轮 lint：core/current-time/get-context-remaining/web-run 均 0 项；
  image-generation 为 2 项，tool-search 为 1 项，unified-exec 为 2 项。
  该检查点只是部分交付，后续完成情况见下节。
- const 保持公开 spec 对象中的初始化常量；实际工具组装绑定到 internal
  expect/actual 对象，事件构造、描述生成和业务机制留在 commonMain。
  未添加 lint suppress 或 baseline，未提交推送。
- 用户随后选择暂缓平台拆分：可继续不改项目/目标边界的 provider 与工具声明迁移；
  需要新受限模块的窄平台问题仍待定，不用 stub 或豁免强行完成。

## 后续完成 — Session1014 / Session998

- [无争议补迁](2026-10-21-migrate-uncontested-spec-api.md)补齐系统 IO/Process/Shell、
  patch、图片、AgentState 等共享契约，并完成 apply-patch、plan、view-image、
  image-generation、unified-exec、tool-search 的剩余声明。
  工具范围当前结构发现为 0，未将未使用 public 隐藏来规避规则。
- 本轮 22 个 utils/tool 变更模块的 metadata/JVM 主代码编译通过；
  定向测试 304 项最终通过，包含新增三个工具常量回归。
  其中租约首次超时、重跑 54 项通过的限制见补迁回执，不能将首次失败略去。
- Linux CLI 依赖图和 ImageGeneration JS 编译通过，CLI/app-rpc 与集成测试源码
  消费者编译通过；本卡不声称全仓测试或全部平台运行通过。
- 工具实现仍调用 search-index 的窄平台工厂，这是上游保留的 provider 边界缺口，
  不是本工具模块公开声明缺失；平台拆分仍未授权。
  原始验证和剩余声明清单见补迁回执；本卡完成范围是工具自有导出 API。
