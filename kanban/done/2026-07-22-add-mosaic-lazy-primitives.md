# Task Tree

- [done] 补齐Mosaic懒布局底层原语
  - [done] 核对现有composition、节点、布局、绘制、pointer和focus数据流
  - [done] 设计并实现keyed measure-time subcomposition
    - [done] 将slot接入父composition、节点树和布局失效
    - [done] 实现slot复用、移动、retention、pin和dispose
    - [done] 保证stable key与`contentType`复用不泄漏状态
    - [done] 防止measure-time apply造成布局重入
  - [done] 实现统一viewport clipping
    - [done] 实现祖先viewport clip交集
    - [done] 将clip用于draw、pointer、focus scope、focus target和physical cursor
    - [done] 支持负向placement和部分可见item
  - [done] 覆盖subcomposition生命周期、identity、约束和clip测试
  - [done] 运行Mosaic相关格式化与多平台测试

# Details

- 状态：已完成。
- 这些能力属于Mosaic通用runtime，不包含Kodex业务模型。
- subcomposition必须留在当前父composition和`MosaicNode`树内，不能使用独立canvas式临时composition。
- 普通focus投影只包含clip内目标；clip外临时目标由后续beyond-bounds协议发现。
- 验证通过：`mosaic-runtime` Spotless、JVM、Linux X64和macOS ARM64测试。
- Mosaic上游bitcode任务不兼容Gradle configuration cache；测试成功，但Native构建的cache条目会被Gradle丢弃。
