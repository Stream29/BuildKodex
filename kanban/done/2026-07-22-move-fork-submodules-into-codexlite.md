# Task Tree

- [done] 将公开构建依赖的fork迁入Kodex仓库
  - [done] 保存并校验Mosaic与TomlKt当前工作状态
  - [done] 从外层BuildKodex移除两个submodule登记
  - [done] 在Kodex根目录登记Mosaic与TomlKt submodule
  - [done] 恢复TomlKt未提交解析器补丁与两个仓库的upstream远端
  - [done] 更新Gradle composite build路径和受影响的文档链接
  - [done] 验证submodule关系、补丁完整性与Kodex构建

# Details

- 只迁移公开fork `Stream29/mosaic`和`Stream29/tomlkt`。
- 不提交或推送任何仓库。
- Linux与macOS从新路径运行相关测试通过。
