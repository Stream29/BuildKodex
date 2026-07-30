# Task Tree

- [done] 改进 CLI 工具调用 history 的展示
  - [done] 调查现有工具事件模型与渲染分支
  - [done] 折叠态显示基于调用内容的语义化描述
  - [done] 展开态保留原始工具名称与详细内容
  - [done] 添加针对不同工具类别的渲染测试
  - [done] 运行相关验证

# Details

- 用户要求：工具调用在默认折叠态使用语义化描述，原始工具名称只在展开后显示。
- 已验证：`env JAVA_HOME=/home/stream/.jdks/graalvm-jdk-21.0.7 ./gradlew :cli-history:jvmTest :cli-patch:jvmTest -x :Mosaic:mosaic-tty:compileJdk22KotlinJvm -x :Mosaic:mosaic-tty:compileJvmJdk22Java`。
