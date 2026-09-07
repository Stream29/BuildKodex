# Task Tree

- [done] 统一 Kodex 测试声明并集中真实凭据测试
  - [done] 盘点旧注解、生命周期及真实凭据依赖
  - [done] 将旧注解式测试迁移至 TestBalloon
  - [done] 将真实凭据测试及专用辅助代码迁入 integration-test
  - [done] 验证测试发现、离线回归及必要平台编译

# Details

- 用户授权两项修改；保留 kotlin.test 断言、现有测试行为及联网测试启用策略。
- 44 个文件、181 个注解测试改为 TestBalloon testSuite/test；使用框架 TestScope，保留原超时。
- 所有原方法名均保留；静态核对测试声明与断言调用计数不变，主仓库无旧测试注解。
- 19 项真实凭据测试迁入 integration-test；storage、catalog、image-generation 的 mock/临时目录用例留在原模块。
- 移动专用辅助代码、清理失效测试依赖；补齐集成模块 Ktor client、JVM auth-filesystem、Native Mosaic runtime 依赖。
- 用户允许本次例外使用本机 Java 25 启动 Gradle 9.5.1；未切换设备、提交或修改依赖子模块。
- 181 个迁移用例均在 JVM/Native 报告中找到，未发现迁移用例失败；两个手动性能探针未启用测量。
- JVM 20 个普通模块最终本轮报告共 538 项，536 通过，2 项未修改用例失败；不是全量通过。
- Linux Ktor 测试 6 项通过，包含迁移的 2 项；选定公共/视图/视图模型迁移模块 Linux 测试编译通过。
- 集成模块 JVM、Linux 测试编译通过；JVM dry-run 发现 33 项，全部跳过实际执行，包含迁入的 19 项。
- 原集成 mock conversation 筛选运行失败：latestIndex 期望 4、实得 3。
- 其余失败：history bounded-window 的初始 viewport 超过 2 秒；history model 的 revert 用例等待 ready 超时。
- 这些失败用例未修改，未断言迁移前也失败，未放宽断言或扩展修复范围。
- Mosaic Native 构建触发配置缓存序列化失败；最终用 --no-configuration-cache 验证，不改缓存策略。
- 未使用真实账户执行联网测试，未做 macOS/Windows 实机验证；git diff --check 通过。
