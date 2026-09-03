# Task Tree

- [done] 改善 `request_user_input` 表单的键盘焦点导航
  - [done] 为每个问题的首个选项和自由输入框建立焦点请求目标
  - [done] 选择普通选项后聚焦下一个问题
  - [done] 选择 `Other` 后聚焦自由输入框
  - [done] 自由输入内容非空时按 Enter 聚焦下一个问题
  - [done] 完成最后一个问题后聚焦 `Submit`
  - [done] 支持跨越 `verticalScroll` 视口定位并滚动焦点
  - [done] 验证焦点能够离开滚动容器

# Details

- 状态：`done`。
- 表单最多占 12 行，并使用普通 `verticalScroll` 裁剪内容。
- 已增加 `verticalScroll` 的 beyond-bounds focus 和 bring-into-view 支持。
- 普通焦点导航仍可从滚动容器进入和离开，不会锁定在滚动容器内。
- Other 输入框只在内容非空且没有修饰键时消费 Enter。
- 已验证 `Mosaic:mosaic-runtime:jvmTest`、`app-contract-lazy-list:linuxX64Test` 和 `app-view-agent:linuxX64Test`。
