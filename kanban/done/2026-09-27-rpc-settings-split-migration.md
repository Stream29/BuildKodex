# Task Tree

- `Receive sequential execution authorization`()
- `Select an unpublished migration target under the existing registry rules`()
- `Implement frozen settings conversion and reentry checks`()
- `Verify interrupted and multi-version upgrades in isolated Homes`()
- `Record evidence and hand off activation to the CLI cutover`()

# Details

## 位置与前置

- 实施批次 P10；父任务：[实施总计划](2026-09-08-explore-krpc-frontend-backend-separation.md#实施计划)。已获顺序实施授权，P01–P09 已验收。
- 状态：Done；依赖已完成的两侧文件格式；未启动并行会话。
- 交付：冻结 migration、codec/fixtures 和重入测试；**不能在旧 CLI 仍读 settings.yml 时激活旧文件删除**。
- 规范：[分批实施边界](../../checklist/rpc-architecture.md#分批实施边界)、[拆分迁移](../../checklist/rpc-settings.md#拆分迁移重入)、[文件切换门禁](../../checklist/rpc-settings.md#文件切换门禁)、[Home migration registry](../../checklist/kodex-home.md#migration-registry-管理)。

## 实施范围

- 实施时核对未发布目标版本和既有 registry 占用，不在计划里猜版本号，不重用或修改已发布 migration。
- 冻结旧/目标最小 codec 与确定性 defaults，按已定字段归属生成两侧配置；不依赖将来可变的当前业务模型。
- 侧栏只迁内容不迁宽度，新 hooks 为空，不转换旧脚本；新源确认完成后删除旧 settings.yml，不额外备份或 journal。
- 旧源存在：先计算两侧并校验全部已有目标，一致跳过、只补缺失；冲突/损坏拒绝，两侧就绪后最后删源。
- 旧源不存在：两目标有效可继续，全缺失不写默认文件，仅一侧存在则拒绝歧义；删除失败可重入。
- 版本推进仍由既有 coordinator 完成；P11 前不激活应用版本。若登记 future entry，必须证明当前版本不会选择它。
- 入口：[全局 registry](../../Kodex/app/migration/impl/src/commonMain/kotlin/io/github/stream29/kodex/app/migration/KodexHomeMigrations.kt)、[迁移模块](../../Kodex/app/migration/impl)、[旧及新设置文件实现](../../Kodex/app/shared/settings/filesystem)。

## 验收

- 全旧、目标已一致、只完成一侧、删源失败、全新 Home、源缺失双目标/单目标、损坏及冲突状态逐项覆盖。
- 中断在各写入/删除边界后重跑；不覆盖已有冲突文件，不在失败时推进版本。
- 旧普通字段迁移精确，宽度/旧 Hook 不进入目标，非法输入 fail closed；未知 Home 路径和已发布 migration 保持不变。
- 跨多个旧版本直接升级及当前版本忽略 future entry；使用隔离 coordinator fixture 检查目标结构。
- JVM/Native migration 测试；不读取、迁移、备份或删除真实 Home，不执行旧 Hook。

## 交接

- 将目标版本、冻结文件清单、状态表测试与激活前置交给 [P11](2026-09-27-rpc-cli-cutover.md)。
- 发布、版本 bump 提交和真实 Home 升级仍遵循各自授权及 SOP；本任务不等于发布授权。

## 目标核对

- 选择未发布的 `0.4.7` future entry；当前源码生成版本为0.4.6，registry 最新为0.4.5，没有重复目标。
- GitHub releases API 与远端 tags 只读核验：最近已发布为 [v0.4.6](https://github.com/Stream29/Kodex/releases/tag/v0.4.6)，没有0.4.7；本批不 bump 应用版本，不提交或发布。
- 旧 shell 缺省依赖宿主发现，不把当前机器的探测结果写进迁移；冻结转换保留该字段缺省，其余固定默认值使用0.4.6语义。显式 shell 原值校验后原样保存。

## 实施与验证记录

- 新增独立 [0.4.7 codec 与迁移](../../Kodex/app/migration/impl/src/commonMain/kotlin/io/github/stream29/kodex/app/migration/v0_4_7)及[冻结 fixture/测试](../../Kodex/app/migration/impl/src/commonTest/kotlin/io/github/stream29/kodex/app/migration/v0_4_7)；生产 codec 不导入当前业务模型，只使用冻结字段、YAML 节点和稳定路径原语。
- 两侧先全部解析/比较；只补缺失目标，私有临时文件写入后原子移动，源最后删除。无备份/journal，失败或取消保留可重入现场，coordinator 独自推进版本。
- 26项新增测试覆盖完整字段/MCP凭据、空/稀疏配置、目标一致/冲突/损坏、四处中断及版本不推进、写入取消清理、跨版本升级和当前0.4.6忽略future entry；均使用临时 Home。
- 新增歧义标量测试发现 Kaml 通用 YamlNode 序列化会将字符串 `0001` 改写为数字；本批冻结 writer 改为保留原标量字符串，覆盖未初始化 OAuth、数字/布尔/null 外观的凭据和 scopes。没有修改普通设置加载器或第三方库。
- 首轮编译修正了新增 codec 中同名扩展解析；歧义字符串回归曾失败，修正 writer 后 JVM/Native 全部通过，未修改旧测试。
- `:app-migration-impl:jvmTest --rerun` / `:app-migration-impl:linuxX64Test --rerun`：最终分别59/58项，均含26项新测试，修正后两次实际执行均通过。租约模块 JVM7项/Native6项通过，旧 CLI JVM编译通过。
- migration 库五目标编译通过；测试源码 JVM/Linux x64/Linux ARM64/Windows x64 编译通过，macOS测试因测试依赖的既有 cinterop 条件跳过。不是五平台实际运行。
- 与 v0.4.6 比对：已发布0.3.3/0.3.5/0.4.3/0.4.5迁移和 fixtures 未变；registry 只追加0.4.7，生成应用版本仍为0.4.6。RPC contract/models 未变，未提交、发布或升级真实 Home。
- 报告：`Kodex/app/migration/impl/build/test-results/{jvmTest,linuxX64Test}`、`Kodex/utils/filesystem-lease/impl/build/test-results/{jvmTest,linuxX64Test}`。临时 Home 随 fixture 清理，临时构建日志记录后删除。
