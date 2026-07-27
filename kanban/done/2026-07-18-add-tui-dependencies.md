# Task Tree

- [done] 引入Mosaic与Koin依赖来源
  - [done] 将官方Mosaic仓库作为`shared-context/`子模块加入
  - [done] 将官方Koin仓库作为`shared-context/`子模块加入
  - [done] 在版本目录中声明实际需要的Mosaic与Koin模块
  - [done] 验证项目Kotlin版本、Mosaic和目标source set能够共同编译

# Details

子模块仅作为源码研究上下文，不作为复合构建依赖。演示程序只使用Mosaic终端
运行时和测试模块，以及Koin core DSL；不引入Koin编译插件或Koog等无关能力。

兼容性验证必须覆盖JVM、Linux Native、macOS Native和mingw Native的共同代码，
并明确Mosaic不支持JS Node这一目标边界。
