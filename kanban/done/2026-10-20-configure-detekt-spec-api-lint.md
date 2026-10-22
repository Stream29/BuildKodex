# Task Tree

- `Freeze the product baseline and inspect local IDE and daemon resources`()
- `Implement the smallest detekt rule and source-set integration`()
- `Test positive and negative declaration fixtures`()
- `Measure cold warm changed-input and no-op runs`()
- `Compare parser parallelism and remove unnecessary integration work`()
- `Record coverage limits and recommend the measured configuration`()
- `Release owned resources and present the result`()

# Details

## 授权与范围

- 2026-10-20 / Session998：用户明确交付仓库与本机环境，授权配置、测试并提出
  性能和工程复杂度尽可能低的 lint 方案。本任务实现和验证 detekt 边界规则；
  测量阶段不顺带迁移业务 API，也不提交、推送或发布。
- 后续用户授权先提交、推送这部分配置，再立即迁移 API。产品配置已提交为
  `d212c43cf`（`build: add lightweight spec API boundary lint`）；
  该提交与外层记录 `178910e` 均已推送并核对远端，随后开始上游迁移。
- 上游：[导出 API 边界迁移](2026-10-20-audit-exported-api-spec-boundaries.md)。
  当前规则见[源集规范](../../checklist/kmp-spec-impl-source-sets.md#source-set-responsibilities)。
- 产品 HEAD `fc36bce2b1d20f1ea1bd5b18db6053714c4740b6`，开始时产品工作树干净。
  外层现有改动保持，不接管其他任务。

## 本机与复现

- 本机 32 GiB 内存，初检 available 7311 MiB；已有大量历史 swap，不把其绝对值
  作为本轮产生的内存。单 worker 执行，不停止用户 IDE 或现有 daemon。
- 复用 IDLE 的 Gradle 9.6.1 / JDK 25 daemon，PID 3429473；
  每次 Gradle 命令显式指定该 JVM。IDE 进程已识别，当前未发现可调用的 IDEA MCP
  工具；不通过窗口操作触发 Sync 或构建。
- 证据目录：`~/ACodeSpace/demo/kodex-detekt-998/`。
  初始 `help --offline --max-workers=1 --profile`：墙钟 9.16 秒，
  Gradle 报告 8 秒；该次存储 configuration cache。launcher RSS 不代表 daemon 峰值。

## 验收检查

- source inputs 来自真实 KMP 生产源集及过滤规则，排除 specMain；测试项目的
  可消费 Main 不豁免。记录生成文件和 compiler-synthesized 声明覆盖范围。
- 覆盖显式/隐式 public、internal/private 容器、构造属性、companion、扩展、
  actual 函数/类/成员；规则不能将 public actual 类的所有成员整体放行。
- 保留结构 lint 与完整符号/expect-actual 语义验证的边界，不虚报完整契约验收。
- 无默认规则集、无自动 baseline/suppress；严格入口发现现有违规应非零退出，
  若提供报告入口必须显式区分，不能将报告任务成功视为违规清零。
- 测量规则工程首次编译、配置、实际分析和端到端时间；分别验证冷/暖/改动/no-op、
  configuration cache、缓存失效、内存和 AST 并行开关。
- 重型操作前后监测可用内存；如资源压力明显或用户接管，停止受影响的自有调用，
  不停止或重配共享 IDE/daemon。

## 验证检查点

- detekt 2.0.0-alpha.6 / Kotlin 2.4.10 独立规则工程已接入；产品 KGP 仍为 2.4.20。
  最终 53 个结构规则样例通过。真实 KMP 输入为 419 个非 specMain 生产文件，含生成版本；
  analysis classpath 为空，已求值任务图只有生成版本和分析，无业务编译/link。
- 首轮报告为 575 个候选边界问题；违规公开容器只报一次，不将子成员重复展开。
  严格入口实际非零退出；独立干净/违规 fixture 分别验证 0/1 问题。
- 已验证 JVM + JS 目标模型下的 specMain/commonMain metadata 和 JVM jar：
  spec 定义接口、expect 工厂/值类型/显式 override，commonMain 提供 actual 与私有实现。
  最初 fixture 漏掉 spec 层 stdlib，并漏写 expect class 的接口 override；
  修正 fixture 后通过，不改产品编译器选项。
- 发现默认 DetektInvoker 的自定义规则重载缺陷：规则 JAR 和 task input 已更新，
  分析实际重新执行，但同一路径的 JAR 仍使用旧类。源码核对其全局 classloader cache
  只按 classpath 文件路径集合取 key，不按内容取 key；不能将该路径的暖态速度
  直接作为可交付方案。已对照验证 Worker API 与内容寻址 artifact transform。
- 原共享 JDK25 daemon 3429473 后续被 IDE 占用；一次未复用的启动按失败记录。
  自有 3580068 已停止；Gradle 随后自动停止旧兼容实例。后续显式复用本轮已有的
  JDK25 daemon 3585737；原 JDK26 daemon 不动。没有调用全局 `--stop`。

## 完成结果 — 2026-10-20 / Session998

- 子任务范围完成，移入 done；配置完成时上游契约审查仍在 discussion，
  后续按用户授权进入 executable。lint 配置结项不是完整契约覆盖验收。
- 选用单聚合 detekt light、关闭默认规则/无关 processors、Gradle 缓存，
  加一个小型标准 artifact transform，使可变规则 JAR 获得内容相关路径。
  在同一暖 daemon 中修改并恢复诊断文本，已验证真正重载，而非只看 task 重新执行。
- 不选择每次启动分析 JVM 的 Worker API 作为默认：对照为 3.052 s，
  最终串行全扫中位数 1.347 s，并行 0.926 s；53 个样例全通过。
  原默认路径虽快但规则重载失败，已淘汰，未作为最终性能口径。
- 无变化报告入口中位数 0.4525 s；无违规 fixture 的严格 no-op 0.469 s。
  当前整仓严格入口 575 个结构问题，实际退出 1；报告成功不代表 API 清零。
- 单文件内容改动后全扫 1.379 s；源码恢复和配置恢复均 FROM-CACHE；
  specMain 公开探针不进入输入，生成文件缺失会先重建且内容一致。
- 独立 256 MiB 堆 CLI 冷 JVM串行 2.32 s、RSS 322 MiB；这是引擎探针，
  不是共享 Gradle daemon 的内存或完整冷启动耗时。
- 主要交付位于 `Kodex/lint/`、根 Gradle 配置和版本目录；业务源码未改。
  `app/migration` 仅把生成目录与真实 producer 关联，未改生成内容。
- 第二次 IDE 占用导致额外 daemon 启动的样本剔除；最终显式复用 3646243。
  临时源码、注释和缺失生成文件均恢复；自有重复实例 3580068 已停止，
  Worker/监控进程已退出。当前标准 Gradle daemon 保留供用户复用，不停止 IDE/JDK26。
- [完整测量、失败口径与复现资料](file:///home/stream/ACodeSpace/demo/kodex-detekt-998/README.md#L1)，
  [最终机器可读汇总](file:///home/stream/ACodeSpace/demo/kodex-detekt-998/summary.json#L1)。
  上述测量阶段没有提交、推送、发布或启动真实模型调用；
  后续配置提交与 API 迁移由上游任务单独跟踪。
