# 首批文档子代理验收

- 本文保留首批验收时的事实；后续修复、整合和仍未展示的交互以[当前验收](docs-interaction-acceptance-2026-09-08.md)为准。
- 范围：Session 297–304 的首批交付；本次是验收，不是正文合入、录制补做或发布授权。
- 结论：**素材与正常回放路径可保留，不能将八个任务全部标为完成。** 播放器有已复现错误，配置/结构记录需校正来源表述，三组状态演示仍阻塞。
- 检查时 Kodex HEAD：`89205188d5da39f36b983a0bfb57752977feb21a`，产品源码工作区干净。本次没有修改产品/正式网站代码，没有提交、推送或发布。

## 分项结果

| Session / 任务 | 首批验收 | 尚缺内容 |
| --- | --- | --- |
| 297 / 结构 | 提案和十组覆盖映射可供审阅，不当作批准目录 | 用户确认；校正旧二进制“来自旧基线”的表述 |
| 298 / 播放器 | 正常路径通过；**边界用例需返工** | 自有属性白名单校验与回归测试；可追溯样例版本 |
| 299 / 工作台 | 三语言素材及双侧栏实录通过本次核对 | 有数据的历史索引、活动进程与多标签实录 |
| 300 / 执行控制 | 三语言素材和 fixture/场景研究可保留 | 无 `.cast`；缺安全运行态到真实屏幕的入口 |
| 301 / 用户协作 | 三语言素材和表单/审批研究可保留 | 无 `.cast`；缺安全 pending interaction 入口 |
| 302 / 历史阅读 | 三语言素材与可丢弃历史构造方案可保留 | 无 `.cast`；缺可运行的安全历史/结果数据 |
| 303 / 输入导航 | 三语言素材及输入/目录实录通过本次核对 | 当前源码的键位设置控件实录、原生文本选择；正式合入 |
| 304 / 配置 | 草稿和旧 Settings 画面可保留；**证据记录需校正** | 未录 New session 默认值/Title generation 操作，不是完整配置演示 |

## 需返工的具体问题

- **P2：播放器白名单接受原型链属性。** [terminal-recordings.mjs:19–21](../../KodexDocs/docs/.vitepress/theme/terminal-recordings.mjs#L19) 直接返回 `recordings[marker]`。输入 `<!-- kodex-player: constructor -->` 会返回 `Object` 函数而非 `undefined`，随后 [TerminalPlayer.vue:34](../../KodexDocs/docs/.vitepress/theme/TerminalPlayer.vue#L34) 对空 `src` 调用 `withBase`。
  - 复现：在临时 Markdown 页只写上述 marker，执行 `npm run build`，SSR 输出 `TypeError: Cannot read properties of undefined (reading 'startsWith')`。
  - 该次构建仍退出 0，不能只看退出码判定成功。反例页已删除，恢复后的 `npm run check` 通过。
  - 返工：只接受注册表自有键；增加 `constructor` 等非注册标记的测试，并检查页面原文能正常生成。没有任意代码执行证据，不将此问题夸大为 RCE。
- **P2：配置/结构记录混淆源码检查与二进制来源。** [配置 scenes.md:65](../../KodexDocs/drafts/configuration-guide/scenes.md#L65) 当时将 `ab54349…` 写成当前 HEAD、写成脏工作树；这与本轮 `89205188…` 的源码基线不一致。[结构提案](../../kanban/done/2026-09-08-docs-manual-outline.md) 当时的“探测二进制来自旧基线”也超出证据。
  - 已知事实只有 CLI SHA256 `92539baa…` 及不同时间的源码检查记录；没有证明该二进制由哪个提交构建。应分别记录，不反推来源。
- **配置片段范围需要明确收窄。** [配置 scenes.md:18–34](../../KodexDocs/drafts/configuration-guide/scenes.md#L18) 列出了模型菜单、New session 和 Title generation 操作，但交付的 `settings-defaults.cast` 与旧 `short.cast` 哈希完全相同。
  - 重放确认只打开 Current session 的 Settings 画面并进入目录选择器，没有录到上述默认值/自动标题控件的操作；不应让计划步骤与已录步骤混在一起。
  - `scenes.md:70` 仍写测试 10/10，而任务收尾记 14/14；应标明各次检查或统一最终结果。本站测试通过不证明配置作用域/标题生成行为。

## 本次实际验证

- 六组各有 `zh-CN.md`、`zh-TW.md`、`en-US.md`、`scenes.md`，共 24 文件；18 份语言稿标题层级对应；141 个 Markdown 相对链接及其行号范围有效。
- 抽核当前实现：steer 提交、控制能力、问答焦点/显式提交、Fork/Revert 边界、目录键鼠处理、设置作用域和标题生成入口。没有将已有测试名称当成本次执行结果。
- 正常站点 `npm run check`：VitePress 构建及 14 项测试全部通过；IDE 对 TerminalPlayer.vue 未报告静态错误。
- 本机 Chromium：三语言首页/样例页原文与 `.md` 字节内容一致；原文资源可读；样例页三语言 × 1440/1024/768/390/320 px 无横向溢出，侧栏贴左。
- 实际验证 hover 展开/移开折叠、点击钉住、Escape、触屏切换；播放器封面、marker 跳转、播放/暂停，以及 SPA 语言切换后只有一个播放器；无 JS 时原文和目录仍可见。
- 三份交付录制逐帧重放：工作台 11 帧、输入导航 15 帧、配置复用片段 1 帧，全部 36 行文本按顺序与原 tmux 抓屏匹配。相同文本帧不能独立证明点击时刻；输入步骤与状态变化仍以原录制/场景记录为依据。
- 三份 `.cast` 均只有输出与正常退出事件，没有输入捕获；哈希与记录一致，输出未命中所检查的私人路径/凭证模式。关键词检查不是完整泄密证明；本轮没有为录制访问个人 Home。
- 未重建 CLI、未重跑 Kodex Gradle 测试、未请求在线模型；未验证跨浏览器、跨机器字体或手机易读性；未重测既有开发态新增文件 HMR 问题。

## 证据与后续边界

- 本机证据：忽略目录 `out/docs-acceptance-2026-09-08/` 的 `verification.json`、`verify.py`、`browser.log`、`check.log`、`constructor-build.log`、截图和独立回放页。不是网站发布资产。
- 正常检查和反例复现使用自有测试服务；已删除临时文档、恢复构建，清理测试服务；原有 4173 开发服务未动。
- 三个复杂状态任务已交付被允许的“入口研究与阻塞说明”，不是伪造成片；缺同一个安全 fixture 到实际屏幕的可运行入口，应统一解决，不让三组分别复制模拟 UI。
- 所有正式内容仍需目录确认、构建来源对齐、必要片段补录与最终合入验收；本次通过的是首批素材和已测路径。
