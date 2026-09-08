# Hooks / MCP 详细实录

- 新增 `hooks`、`mcp` 两段原生 CLI 录制，三语言各增加两个演示标题，没有增加说明字幕。全站现为 12 段、全部自动播放，仍可手动暂停/恢复。
- CLI SHA256 `99882904ad4d20eb65e00dffd983b77aec4893ef3ca4bdfcdb8684a6f4800af3`；录制器、操作脚本、测试服务和资产哈希见 [manifest](../../KodexDocs/docs/public/recordings/manifest.json)。
- [录制脚本](../../KodexDocs/scripts/record-integrations.py)创建独立 Home/project、禁网 bubblewrap、独立 tmux；不挂载个人 Home、认证或真实 Codex 配置。退出时回收专用进程树再删除临时目录。
- 操作脚本：[Hooks](../../KodexDocs/scripts/fixtures/hooks-actions.json)、[MCP](../../KodexDocs/scripts/fixtures/mcp-actions.json)。用于再现操作的 `contains`/`absent` 断言直接检查原终端，没有手写或修改 ANSI 画面。

## 展示与实际验证

- Hooks：空列表、Add hover、名称/命令校验、七种事件、保存、条目 hover、隐藏命令的详情、显式编辑、取消编辑、改名/改命令、删除取消/确认。23 个原终端检查点在浏览器逐行匹配；持久化快照证明取消不改配置、更新与删除确实落盘。
- MCP：HTTP/OAuth 未保存配置表单、stdio 字段/环境条目校验、真实初始化失败、Reconnect hover 与成功、两工具 catalog、启停、环境值隐藏及 `<keep>`、改名、Codex 新增/替换选择、Clear/Select all/单项取消、实际导入、删除取消/确认。33 个原终端检查点逐行匹配。
- 本地 [MCP 演示服务](../../KodexDocs/scripts/fixtures/mcp-demo.py)第一次按测试参数退出，其后正常响应 SDK 协议请求；录到真实 Failed → Healthy，而非给 ViewModel 手填状态。进程日志记录 6 次启动、5 次 initialize、5 次 tools/list。
- MCP 配置快照证明禁用保留参数、改名保留环境值、导入替换实际参数并新增第二项、取消删除不改变配置、最终删除清空列表。
- 边界：Hooks 片段展示配置管理，不声称执行七种事件链；HTTP/OAuth 仅展示草稿字段，不声称完成外部服务登录/注销。MCP 连接的是可丢弃本地演示服务，没有调用模型或个人服务器。

## 导入发现

- 含 unsupported 项的预演中，即使只选两个支持项，提交后也没有导入；该失败片段没有作为正式成功演示。正式样例源只包含两个支持项。
- 源码解释：UI [默认决策](../../Kodex/app/view/settings/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/McpSettingsDialogs.kt#L515)包含 unsupported 项的 `Skip`；Manager [预览配置表](../../Kodex/mcp/impl/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpManagerImpl.kt#L358)只保留支持项，[提交校验](../../Kodex/mcp/impl/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpManagerImpl.kt#L382)却要求全部决策 key 都在配置表中，因此会拒绝该组合。
- 没有为录制修改产品逻辑。失败现场文字保存在 `out/docs-integrations/import-unsupported-failure.txt`；没有把可见选择列表等同于导入成功。

## 验收与本机证据

- `npm run check`：构建及 14 项测试通过，包含 12 段资产哈希、来源/动作脚本哈希、三语言原文及页面纯净性。
- 实际浏览器：生产预览和开发服务各检查三语言、每页 12 段自动播放、五种宽度、手动暂停/恢复、语言锚点、目录 hover/钉住/Escape/移动端操作、无 JS 正文；没有页面异常。没有重跑 JVM 测试，本轮没有修改产品运行时代码。
- 新片段 56 个检查点及持久化/协议核验：`out/docs-integrations/verification.json`；网站验证：`out/docs-showcase/verification.json`。
- 最终素材、帧、来源、配置快照与本地协议证据保留在 `out/docs-integrations/final/`，不把配置/日志拷入网站。预演、失败中间文件、下载工具和临时环境用后清理；无提交、推送或发布。
