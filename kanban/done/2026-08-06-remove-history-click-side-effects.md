# Task Tree

- [done] 移除历史消息左键聚焦的视觉与滚动副作用
  - [done] 确认焦点高亮与视口重定位来源
  - [done] 移除已提交 history 条目的 focus 背景
    - [done] 删除 history 专用 focus 颜色参数与绘制
    - [done] 保留完整条目的 focusable surface
  - [done] 让鼠标聚焦不发起 bring-into-view
    - [done] 仅在 pointer focus 路径跳过 relocation
    - [done] 保持其他焦点入口的 relocation 语义
  - [done] 保留键盘聚焦、分页与上下文菜单能力
    - [done] 覆盖左键聚焦后 `Shift+F10`
    - [done] 覆盖部分可见条目点击不改变滚动 anchor
  - [done] 运行定向测试并重新构建 release CLI

# Details

- 左键仍需把已提交 history 条目设为焦点，以保留 `Shift+F10` 与键盘导航。
- 历史条目不显示 hover 或 focus 背景。
- 鼠标命中的焦点目标已位于当前 viewport；鼠标聚焦不应请求视口重定位。
- 键盘遍历、程序化聚焦与初始聚焦继续使用既有 bring-into-view。
- 修改范围为 Mosaic `FocusOwner` 的 pointer 入口、history row 绘制及其定向测试。
- Mosaic runtime Linux X64 测试通过，共 412 项；CLI components、history 与 application Linux X64 测试通过。
- Mosaic runtime Kotlin 格式检查、IDE inspection 与 `git diff --check` 通过。
- release executable 已重新链接：`Kodex/app/cli/application/build/bin/linuxX64/releaseExecutable/app-cli-application.kexe`。
