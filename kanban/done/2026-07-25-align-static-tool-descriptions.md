# Task Tree

- [done] 对齐静态工具描述
  - [done] 对齐`apply_patch`
  - [done] 对齐`image_gen.imagegen`
  - [done] 对齐`view_image`
  - [done] 对齐`update_plan`末尾换行
  - [done] 增加精确描述测试
  - [done] 运行相关模块测试

# Details

- 本任务只允许替换或删除静态模型可见文本。
- 不改变工具签名、schema形状、handler行为或依赖关系。
- Rust没有提供字段描述时，删除Kotlin自行添加的字段描述。
- JVM、JS Node、Linux Native和macOS ARM64测试均通过。
