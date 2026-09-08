# Task Tree

- [done] 为说明书接入可复用的真实终端录制播放器。
  - [done] [探索并验证可复用录制流程](../done/2026-09-08-docs-recording-workflow.md)
  - [done] 确认可录制环境、演示数据与源码/版本基线。
  - [done] 确定 Markdown 原文与播放器的挂载方式，复用播放器 JS/CSS。
  - [done] 复用并接入一个真实的侧栏展开、钉住示例。
  - [done] 修复原型链标记误命中并补充页面渲染回归测试。
  - [done] 验证字符、颜色、尺寸、播放控制、静态资源与隐私边界。

# Details

- 本轮由主 Session 统一完成白名单修复、三语言录制注册、页面整合与浏览器验收；不再受首批代理独占文件范围限制，不发布。
- Object.hasOwn 白名单、21 页/48 项站点检查通过，13 段片源的 122 个检查点回放核对通过；原生 CLI 已重建并记录源码/生成器哈希。旧 probe 页与资产已从正式站点移除。见[当前验收](../../shared-context/findings/docs-interaction-acceptance-2026-09-08.md)。
- 新文件 HMR 已在[主题任务](../done/2026-09-08-minimize-docs-theme.md)修复；下列首批记录不再代表当前阻塞。

## 首批实现与验收（历史记录）

- 方案已由用户选择：[asciinema-player](https://docs.asciinema.org/manual/player/)，仅观看真实操作回放，不需要访问者操作真实 TUI。
- 复用一个播放器组件与静态 `.cast` 文件；不自写终端仿真、播放控制栏或远程进程服务。
- 当前主题刻意不执行 Markdown 中的 HTML/Vue；接入方式必须保留原文展示与字面内容安全性，不能靠重新启用任意正文代码执行实现嵌入。
- 录制环境通过关联探索任务验证；不得擅自使用私人会话、录入凭证、修改个人 Kodex Home 或切换设备。
- [可复用实录流程](../../shared-context/findings/kodex-terminal-recording.md)已完成独立浏览器验证，用户认可效果；先复用现成录制器、PTY 输入和播放器，不另造录制工具。正式内容仍须确认构建基线、字体/默认主题及窄屏可读性。
- 先比对一个实际运行的短场景，再扩大录制范围；检查 CJK 宽字符、符号、颜色、终端行列数、窄屏缩放及暂停/跳转行为。
- 录制文件含终端内容，应逐项检查敏感信息；“播放器可交互”不等于“用户可以操作 Kodex”。
- 原型完成后供内容子任务复用；未经额外授权不提交、推送或发布录制。
- 子代理首批交付：确定并实现最小的原文文档/播放器挂载接口，局部接入一个真实样例；向内容任务提供静态录制路径、marker/封面约定及浏览器验证结果，不顺带决定最终目录。
- 并行写入范围：本任务、`KodexDocs/docs/.vitepress/`、必要的包清单/锁文件与播放器测试、自有样例页/资产；不改首页和其他内容稿，不改产品 `Kodex/`。试验页明确标识用途，不冒充正式说明书章节。
- 首批继承的[主题精简任务](../done/2026-09-08-minimize-docs-theme.md)当时尚有新文件发现问题；后续已统一解决。
- 首批本地实现：`docs/.vitepress/theme/TerminalPlayer.vue` 只在 allowlist marker
  `<!-- kodex-player: sidebar-pinning-probe -->` 出现时挂载；marker 仍由 `v-text` 原样展示，
  不执行正文 HTML/Vue。录制注册表在同目录 `terminal-recordings.mjs`，封面约定使用
  asciinema-player 的 `poster: 'npt:4.5'`，marker 时间点为 2.5/4.5/6.5/9.5/12.5 秒。
- 样例页为 `docs/demos/terminal-sidebar-pinning.md` 及其 `zh-CN` / `zh-TW` 对应页；
  静态资产为 `docs/public/recordings/sidebar-pinning.cast`，SHA-256 为
  `f527e5dd7c578f5ab3816e4a3e64eba7ebbb147fa183641c091500f0ea3912a1`。它复制自独立 probe，
  页面已明确标注不是正式发布版样例，CLI 与源码提交对应关系仍未确认。
- 已验证 `npm run check`（构建 + 14 项测试）；独立 4174 预览页在 1024/390/320 px
  回放，播放/暂停、进度/marker 条、poster、真彩色 `rgb(158, 239, 253)`、JetBrains Mono
  / Noto Sans Mono CJK SC、zh-CN 页面、侧栏触摸钉住与本地静态请求均通过；无控制台错误或横向溢出。
  120 列终端在窄屏仍只是等比缩小，正式内容需拆短场景，不能宣称手机上易读。
- 剩余依赖：需补录并记录可追溯的 CLI 构建/源码基线，正式内容任务仍需决定是否复用该 probe；
  主题精简任务的新文件 HMR 检查仍未通过，本任务未覆盖或重试该问题；未提交、推送、发布或改变 Pages。
- 主 Session 验收：正常构建/14 项测试及三语言五宽度浏览器路径通过；但 `<!-- kodex-player: constructor -->` 被注册表误识别，导致 SSR `withBase(undefined)` 报错（构建进程仍退出 0）。只接受注册表自有键并补测后，才能验收这一边界；详见[首批验收](../../shared-context/findings/docs-subagent-acceptance-2026-09-08.md)。
