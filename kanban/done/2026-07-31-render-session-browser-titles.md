# Task Tree

- [done] 让 CLI session popup 显示 root Agent 名称
  - [done] 确认 popup 硬编码编号且 catalog 未投影标题
  - [done] 为 Session catalog 提供不打开 runtime 的 root `threadName` 摘要
  - [done] 将摘要接入 Session ViewModel 与 popup 标签
  - [done] 覆盖 repository 摘要和 popup 标签回退
  - [done] 运行定向验证

# Details

- 用户要求：session list popup 显示对应 master Agent 的名称，而非一律显示 session 编号。
- 已验证：`./gradlew -Dorg.gradle.java.home=/home/stream/.jdks/graalvm-jdk-21.0.7 :agent-session-in-memory:jvmTest :agent-session-filesystem:jvmTest`；以及排除现有 Mosaic JDK22 编译问题后的 `:cli-session:jvmTest :cli-app:linuxX64Test`。
