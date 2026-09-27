# Task Tree

- **`Await scoped implementation authorization`()**
- `Select an unpublished migration target under the existing registry rules`()
- `Implement frozen settings conversion and reentry checks`()
- `Verify interrupted and multi-version upgrades in isolated Homes`()
- `Record evidence and hand off activation to the CLI cutover`()

# Details

## 位置与前置

- 实施批次 P10；父任务：[实施总计划](2026-09-08-explore-krpc-frontend-backend-separation.md#实施计划)。
- 状态：Planning；依赖已完成的两侧文件格式，可与其他批次逻辑独立准备；不自动启动并行会话或构建。
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
