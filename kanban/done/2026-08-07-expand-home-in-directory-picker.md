# Task Tree

- [done] 修复以 `~` 打开目录选择器失败
  - [done] 定位初始路径读取失败原因
  - [done] 明确 home shorthand 展开边界
  - [done] 为共享目录浏览器补充回归测试
  - [done] 在读取目录前展开当前用户 home
  - [done] 运行 shared 与 Mosaic 定向测试
  - [done] 更新 path picker 决策记录

# Details

- 状态：`done`。
- 用户报告以 `~` 为基础打开路径选择器时显示 `Unable to read directory`。
- 修复前 `DirectoryPickerBrowser.load()` 直接调用 `CoroutineFileSystem.resolve(Path("~"))`；filesystem resolve 只解析相对路径，不提供 shell 的 home shorthand 展开，因此会读取当前工作目录下名为 `~` 的目录。
- 修复范围限定为当前用户 shorthand：支持 `~` 和 `~/...`；不实现 `~user` 查询。
- 在 `app-shared-path-picker` 中复用 `utils-os-environment` 的当前用户 home，并允许测试注入确定的 home 路径。
- 共享浏览器回归测试覆盖 `~` 与 `~/workspace`，并运行 `:app-shared-path-picker:linuxX64Test` 和 `:app-cli-path-picker:linuxX64Test`。
- 定向验证通过：`JAVA_HOME=/home/stream/.jdks/openjdk-26.0.2 ./gradlew --no-configuration-cache --console=plain :app-shared-path-picker:linuxX64Test :app-cli-path-picker:linuxX64Test`。
