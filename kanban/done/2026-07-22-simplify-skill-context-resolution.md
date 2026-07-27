# Task Tree

- [done] 简化skill context的按cwd解析
  - [done] 复核当前`SkillsService`、`SkillsGeneration`与Rust侧turn snapshot语义
  - [done] 确定只支持全局skills与当前cwd项目skills的解析范围
  - [done] 确定逐user turn热重载与文件指纹缓存方案
  - [done] [收窄用户消息与持久化上下文注入边界](../done/2026-07-22-separate-user-message-and-history-injection.md)
  - [done] 以`resolve(cwd: Path)`查询替换共享`current`、`refresh`与数字generation
  - [done] 让AgentRuntime在新user turn开始时解析并固定本轮context injection
  - [done] 每轮重新枚举全局与项目skill目录，正确处理新增、删除和重命名
  - [done] 按canonical `SKILL.md`路径及修改时间、大小缓存文件读取与解析结果
  - [done] 在可用时将file key用于识别相同时间与大小的原子替换
  - [done] 保持available skills为轻量metadata，显式选择后才读取并持久化完整正文
  - [done] 移除不再需要的SkillsService、SkillsGeneration与prefix provider共享状态
  - [done] 更新相关contract、runtime、文件系统实现、测试与设计记录
  - [done] 运行格式化及相关跨平台测试

# Details

- 已完成实现与验证。
- 本任务采用Rust行为的有意子集，不引入plugin roots、config stack、远程filesystem authority或额外generation token。
- 全局skill目录由resolver构造参数固定；项目skill目录只根据本轮cwd发现。
- 每个新user turn都重新枚举目录并读取文件attributes，以此提供skill热重载。
- 同一turn中的Responses续跑、tool continuation和自动压缩复用已经解析的context injection。
- 文件指纹未变化时复用内存中的解析结果，只省略正文读取与解析；目录遍历和attributes读取仍会执行。
- cache key使用canonical `SKILL.md`路径，不使用skill name。
- 一次cwd扫描未发现的缓存项不得立即全局删除，避免不同Agent和不同cwd互相驱逐；使用有界缓存限制内存增长。
- global或project scope由本次发现所在的root决定，不固化进跨cwd复用的文件缓存。
- 已选skill正文通过独立`injectHistory`操作持久化；文件后续变化只影响下一条user message。
- AgentState不执行文件系统扫描，也不在每次Responses请求读取共享可变provider。
