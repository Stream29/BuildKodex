# Task Tree

- [done] 为 CLI session list 增加删除与最近活动排序
  - [done] 确认既有删除操作会删除持久化 root session，并关闭其打开标签
  - [done] 按最后活动时间从近到远投影 session catalog
  - [done] 在每条 session list 记录上提供带确认的 Delete 操作
  - [done] 补充排序与删除路径的定向测试并运行相关 Gradle 验证

# Details

- 用户明确要求：session list view 提供 Delete 按钮，并按最后活动时间从近到远排序。
- Delete 操作复用既有 `SessionTreeCliViewModel.delete(sessionIndex)`，其目标是单个持久化 root session。
- 已验证：`./gradlew -Dorg.gradle.java.home=/home/stream/.jdks/graalvm-jdk-21.0.7 :cli-app:linuxX64Test :cli-session:linuxX64Test --console=plain --no-configuration-cache -x :cli-history:compileKotlinLinuxX64 -x :cli-history:linuxX64MainKlibrary`。
