# MacBook 0.4.2发布构建记录

- 2026-09-05验证，源码为`8d4cfba7512b5ba6bea470e211e7e9bb9a52826c`。
- 使用GraalVM Java 25和`--no-configuration-cache`；四个CLI Release目标均能直接在MacBook构建，包括Windows x64，无需回退其他主机。
- JVM验证依赖Mosaic Zig 0.15.1。默认Xcode 26.5 SDK下，Zig的build runner报`_abort`等系统符号未定义。
- 仅为JVM验证命令设置`DEVELOPER_DIR=/Library/Developer/CommandLineTools`可解决；该环境使用现有26.2 SDK。仅设置SDKROOT未解决，不需要修改产品源码或全局xcode-select。
- Zig归档手动供应到`Mosaic/mosaic-tty/build/zig/`后，验证命令用`-x :Mosaic:mosaic-tty:zigDownload`跳过重复下载，不跳过JNI编译或测试。
- Native发布构建使用默认Xcode环境；76项JVM测试及14项macOS Native Hook测试通过。
- Linux x64隔离Home升级启动、macOS ARM64隔离Home首次启动及正常退出通过；Windows与Linux ARM64仅验证构建和归档，没有实机启动结论。
- macOS归档使用`COPYFILE_DISABLE=1 tar --no-xattrs`，防止带入`._kodex`或扩展属性；Windows使用`zip -X`。
- 四个归档均只有根入口`kodex`或`kodex.exe`；macOS二进制的本地codesign验证通过，不等同于Apple公证。
- 最终产物保存在MacBook的`~/ACodeSpace/local/Kodex/out/0.4.2/`；发布检查以GitHub的五个asset名称、大小和SHA-256逐项对照。
