# Task Tree

- [done] 让 CLI session list 显示最后活动时间
  - [done] 确认现有轻量 catalog 未投影时间戳
  - [done] 将 root session 的最新活动时间加入 catalog
  - [done] 在 session list 标签中渲染最后活动时间
  - [done] 补充定向测试并运行相关 Gradle 验证

# Details

- 用户明确要求：在 CLI session list view 中展示最后活动时间。
- 保持 catalog 的轻量读取边界，不打开 Agent runtime。
- 已验证：`./gradlew -Dorg.gradle.java.home=/home/stream/.jdks/graalvm-jdk-21.0.7 :cli-app:linuxX64Test :cli-session:linuxX64Test --console=plain --no-configuration-cache -x :cli-history:compileKotlinLinuxX64 -x :cli-history:linuxX64MainKlibrary`。
