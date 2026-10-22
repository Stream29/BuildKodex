# Task Tree

- `Freeze the current client target boundary and preserve existing edits`()
- `Align external URL and View contracts with client targets`()
- `Verify target models metadata JVM tests and Native consumers`()
- `Compare strict lint and preserve unrelated migration work`()
- `Record acceptance and release temporary validation resources`()

# Details

- 上游：[导出 API 迁移](2026-10-20-audit-exported-api-spec-boundaries.md)。
  用户确认 external-url 属于客户端能力并授权实施；不是后端宿主能力。
- 本轮以已落地客户端为准：JVM、linuxX64、linuxArm64、macosArm64、mingwX64。
  View 与 external-url 共用客户端 convention，CLI/后端目标不随之扩张。
  结果类型随客户端目标；取消原 Shared 模块中仅结果类型的 JS/Wasm/iOS 导出。
  浏览器目标与真实实现随 Web 前端建设推进，不添加空 actual 或提前移植整图。
- 不拆 system 模块、不引入 RPC 打开链接、不改包名/签名/默认值、失败结果或取消行为。
  本轮同时分离 OpenAiLoginPopup 和 SessionTreeCliScreen；
  它们的公开默认参数原先被 external-url 缺少 spec 声明阻塞。
- 保留已有桌面实现、源集层级、Windows 链接参数和 Native 网络 pin；
  不改变其他 Shared/Host 项目的目标集合。客户端与 Host 只复用准确的桌面目标配置，
  不让客户端 convention 继承后端 convention。
- 产品基线 `d212c43cf`，此前迁移工作树未提交；原文件 hash 清单保存在
  `~/ACodeSpace/demo/kodex-detekt-998/url-client-before/manifest.json`。
  所有本轮范围外文件须保持，不能批量清理工作树或提交推送。
- 单 worker 串行验证、复用空闲 JDK25 daemon；IDE MCP 重试连接已丢失，
  不操作用户窗口或停止共享进程。
- 验收：真实 KMP 目标/源集模型、三个模块的 spec/common metadata、
  JVM 测试与 CLI 消费者、代表性 Linux 编译、严格与报告 lint 对照。
  app/view 两个已在干净基线复现的旧录制失败单列，不改断言或排除用例。

## 实施与验收

- 增加小型 `kodex.kmp-client` convention，View 与 external-url 共用；
  原桌面目标函数改名为 configureDesktopTargets，CLI/Host 仍直接复用同一桌面配置，
  Host 继续单独加自己的 JS，客户端没有继承 Host/CLI convention。
  不新增业务模块或依赖端口。
- openExternalUrl 的完整行为 KDoc 与 expect 进入 specMain，已有桌面机制只加 actual；
  OpenAiLoginPopup、SessionTreeCliScreen 同样分离，默认参数保留在 expect，
  签名/defaults 依赖进入相应 spec，UI 实现体保持。
- 外链空白输入测试移入客户端 commonTest，覆盖空串、空格和制表/换行，
  不打开真实浏览器或运行宿主 launcher。
- 实际模型验证：外链、根 View、登录 View 和 CLI 都是 JVM/四 Native；
  后端 composition 仍有 JS，Shared images 仍有 JS/Wasm/iOS。
  spec/common、cliHost/desktopNative、Mosaic 的原 dependsOn 边保留。
- 三个模块的 spec 及适用 cliHost/Mosaic metadata、JVM 主代码通过；
  CLI JVM、integration-test 测试源码及 Linux CLI klib 编译通过。
  外链 JVM 3 项、Linux Native 3 项、登录 View JVM 9 项测试全部通过。
  Mac/Windows 未在本机运行；浏览器 target/实现不在本轮交付中。
- 根 View 68 项 JVM 测试为 66 通过、2 个录制失败；
  当前失败名、类型、消息和首个测试源码栈帧与此前干净 d212c43cf 基线完全一致，
  测试源文件 hash 未变。组合 JVM 命令因此退出 1，不能写成全部测试通过。
- 严格/报告 lint 的 43 项完全一致，严格退出 1；
  对照原 46 项恰好移除本轮三个声明，没有新增发现，三个模块均清零。
  最新为 43 项/14 模块；剩余窄平台与关联依赖 24、inline/contracts 7、
  CLI/kRPC 桥 10、Sidebar const 2，继续留上游讨论。
- 本轮除 12 个原文件修改/迁移和 4 个新增文件外，快照中 1493 份产品文件保持原 hash；
  没有改 lint/版本/协议、添加 suppress/stub、暂存或提交推送。

## 验证回执与限制

- 原始日志和汇总在 `~/ACodeSpace/demo/kodex-detekt-998/url-client-*`，
  总回执为 `url-client-validation-summary.json`，包括目标模型、测试、保护检查、
  lint 差异和旧失败对照。目标探针代码已存入汇总，临时 init 文件已删除。
- 首次启动因共享 daemon BUSY 在预检退出，没有启动竞争构建；
  首版 init 探针误作用于 included build，限定主产品根目录后重跑通过。
  所有实际验证复用同一 JDK25 daemon、单 worker，无资源阈值中断或额外 daemon。
- 性能保留实测而非沿用上一轮数字：两次配置缓存命中强制全扫端到端
  3.094/3.153 s，分析 1.791/1.640 s；无变化命中缓存为 1.340 s，分析 UP-TO-DATE。
  首次独立全扫需配置为 15.143 s；此前混合业务测试/报告/严格检查为 45.384 s。
  本次紧接重型编译且共享本机资源，未做隔离 A/B，不把波动归因为本次代码或声称性能不变。
- 保留正常共享 IDE/Gradle daemon；无遗留测试 worker，未将本项完成扩展为整仓迁移完成。
- 文档已完成删减、变更含义及全文一致性检查；有界辅助复审仅基于提供的文本，
  返回未发现问题，回执为 `url-client-document-review.json`，不代替实际构建验收。
