# Task Tree

- [done] 修复 Mosaic JVM `PlatformKt` 重名
  - [done] 将公共时间换算移至独立文件
  - [done] 保持现有跨平台调用和测试
  - [done] 记录 KMP JVM file facade 约束
  - [done] 验证 Mosaic 与 Kodex JVM 构建

# Details

- 用户已明确要求修复。
- `commonMain` 与 `jvmMain` 的同包同名 `platform.kt` 当前都会生成 `PlatformKt`。
- 采用已验证过的最小方案：让 `commonMain/platform.kt` 只保留 `expect` 声明，将普通公共实现移至 `time.kt`。
- 修改范围限定为 Mosaic 公共源码、JVM 约束清单和任务记录；现有测试继续覆盖秒边界换算。
- 验证顺序为格式与静态检查、Mosaic runtime JVM/Linux X64 测试、Kodex components JVM 测试。
- 具体修改与验证路径已确定，可以执行。
- `timespecToNanos` 与其换算常量已从 `commonMain/platform.kt` 移至 `commonMain/time.kt`；调用签名未变。
- `checklist/jvm-toolchain.md` 已记录同包同 basename 公共/JVM 源文件的 file facade 约束。
- `:Mosaic:mosaic-runtime:jvmTest` 与 `:Mosaic:mosaic-runtime:linuxX64Test` 通过。
- `:app-view-components:jvmTest` 通过，Kodex 实际依赖链不再出现重复 `PlatformKt`。
- IntelliJ 格式化及目标文件 inspections 通过。
- IntelliJ 定向构建被现有 Gradle 类型安全访问器生成错误阻断：`RootProjectAccessor.getKotlinSdk()` 重复；命令行目标构建不受影响。
- `:Mosaic:mosaic-runtime:spotlessCheck` 被现有 `DrawTextStyleOverlayTest.kt` 导入顺序问题阻断，未修改无关文件。
- 根仓库、Kodex 与 Mosaic 的 `git diff --check` 均通过。
