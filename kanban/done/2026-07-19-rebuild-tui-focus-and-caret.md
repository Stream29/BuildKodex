# Task Tree

- [done] 重新实现TUI焦点与光标定位
  - [done] 为Mosaic补充布局坐标回调并覆盖布局回归测试
  - [done] 建立TUI焦点宿主、焦点请求者与插入点上报机制
  - [done] 将演示输入框迁移到焦点宿主，移除组件直接定位物理光标的实现
  - [done] 覆盖焦点切换、插入点定位和现有输入行为
  - [done] 在Linux TUI路径构建并验证

# Details

旧的演示输入框直接调用`TerminalCursor`，其生命周期无法表达多个可聚焦控件的所有权关系。新的设计由`TuiFocusHost`集中选择活动插入点并作为唯一的物理终端光标出口；输入控件只维护自身编辑状态并通过布局坐标上报插入点。

当前仓库没有包含旧TUI光标方案的可达提交。保留Mosaic中已提交的Unicode键盘事件修复，直接替换尚未提交的旧光标实现。

验证：`JAVA_HOME=/home/stream/.jdks/graalvm-jdk-21.0.7 ./gradlew :tui-mosaic-runtime:linuxX64Test :tui-focus:linuxX64Test :tui-demo:linuxX64Test --configuration-cache --console=plain`。

终端 smoke test：启动Linux调试可执行文件，输入`你🙂`后正确显示，物理光标定位到第6个终端单元格，再以`Ctrl+Q`退出。
