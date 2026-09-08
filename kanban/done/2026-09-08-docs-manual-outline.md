# Task Tree

- [done] 确认基于实际能力的产品说明书结构。
  - [done] 将已核对的功能细节映射到内容清单，检查遗漏与重复。
  - [done] 提出页面、导航和演示编排，取得用户确认。
  - [done] 明确已有安装、认证、上下文、MCP、Hooks 和排错内容的去向。
  - [done] 确定三语言对应关系及逐页验收清单。
  - [done] 校正二进制来源表述，不将检查时 HEAD 当作构建证明。

# Details

- 用户已确认采用五页主线（含首页）加连接、安装两个参考页；三语言同构。执行中的离线样例使用真实组件和示例数据，明确区别于在线模型结果；本轮整合并提供本地预览，不提交或发布。

- 七页目录已获确认；21 页正式内容与五种宽度的浏览器验收通过。实际接入 13 段录制；不将页面结构完成等同于全部操作分支已成片。见[当前验收](../../shared-context/findings/docs-interaction-acceptance-2026-09-08.md)。

## 首批提案与覆盖盘点（历史记录）
- 已确认必须覆盖的十组细节：双侧栏；历史索引与预览；运行中 steer；Stop/Clear pending/Resume/Compact；提问表单；建议新会话审批；按条目 Fork/Revert；Patch 与计划展示；输入编辑与目录选择；配置作用域与自动标题。
- 细节和源码锚点分别记录在各内容子任务中，目录阶段检查覆盖，不复制第二套长清单。
- 用户偏好少量真实演示替代冗长文字；每项保留必要的用途、操作差异和限制。录制数量、具体页面粒度仍待确认。
- 不以测试名或设计文档替代实际实现；对新增盘点项标明已核对、待核对或版本不符。
- 子代理首批交付：在本任务 Details 中给出精简页面/演示编排提案与十组细节覆盖映射，列出必要的待确认选择；不直接改网站、不宣称提案已获批准。
- 并行边界：只修改本任务；其他内容卡和现有网站作为只读输入。目录确认前不阻挡六组任务准备独立草稿，但不要求其他代理按提案改路由。

## 页面与演示编排（页面已确认，六段合并编排已由十三段短片替代）

- 建议主线只保留五页，按用户完成一次工作的顺序组织，而不是按实现模块或测试名组织：
  - `index.md`：产品说明书首页。用一句话说明 Kodex 的工作方式，列出五页能力地图和演示入口；不把安装步骤放在首屏。
  - `workspace.md`：工作台与输入。合并工作台/侧栏和输入/目录两组草稿，先看会话、侧栏、历史定位，再用一段短片完成输入与工作目录选择。
  - `run-session.md`：执行中的协作。合并运行控制和用户协作两组草稿，先讲运行中追加要求，再讲回答问题与审批建议。
  - `review-history.md`：阅读结果与历史分支。覆盖历史定位、工具/补丁/计划阅读，以及从具体条目 Fork/Revert。
  - `configuration.md`：配置会话行为。覆盖全局、当前会话、新会话的作用域和自动标题，明确哪些改变只影响后续新会话。
- 建议另设两个低优先级参考页，不与主线抢导航位置：
  - `connections.md`：认证、上下文来源、MCP、Hooks；以 Settings 中的真实入口和边界为主，不把它们泛称为“工具”。
  - `install-reference.md`：发行版下载/校验/启动、数据位置和精简排错；服务于第一次启动和出问题时查阅，不作为产品导览首页。
- 建议首批演示编排为六段而非每个细节一段：`workspace`（侧栏 + 历史定位）、`input`（粘贴/撤销 + 目录选择）、`run`（steer + 控制）、`collaboration`（提问 + 建议审批）、`review`（结果阅读 + Fork/Revert）、`configuration`（作用域 + 标题入口）。每页正文先写用途、操作差异、限制，再放一至两段短实录；连接/安装页首批可只保留安全的界面演示和静态说明，是否录制真实认证/MCP/Hooks 待确认。
- 本节保留首批提案依据；当前批准状态、正式路由和实际片段以本文件开头及当前验收为准，不再执行旧的“仅独立草稿”边界。

## 十组精细交互覆盖映射

| 细节组 | 拟归页面 | 拟归演示 | 已核对的实现/测试依据与边界 |
| --- | --- | --- | --- |
| 双侧栏 | `workspace.md` | `workspace` | 左右侧栏分别渲染，支持独立展开、内容选择和拖动宽度；见 [`SessionSidebar.kt:148-250`](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionSidebar.kt#L148) 与 [`SessionTreeCliScreen.kt:760-791`](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt#L760)。应演示“分别配置”，不写成固定左侧会话树；正式实录尚未验证双侧都覆盖。 |
| 历史索引与预览 | `workspace.md` | `workspace` | 悬停会异步加载完整历史条目，右键 `Check out` 只请求历史滚动定位；见 [`SessionSidebar.kt:707-769`](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionSidebar.kt#L707) 与 [`SessionTreeCliScreen.kt:730-755`](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt#L730)。`HistoryIndexViewModelTest` 已覆盖完整 hover detail（[`HistoryIndexViewModelTest.kt:139`](../../Kodex/app/viewmodel/agent/src/commonTest/kotlin/io/github/stream29/kodex/cli/agent/HistoryIndexViewModelTest.kt#L139)）；需明确这不是 Git checkout 或对话回退。 |
| 运行中 steer | `run-session.md` | `run` | 运行中提交加入 `pendingSteer`，并在 Composer 显示 `Submit to steer`/Pending 预览；见 [`AgentRuntimeViewModel.kt:213-218`](../../Kodex/app/viewmodel/agent/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/AgentRuntimeViewModel.kt#L213) 和 [`ComposerInputTest.kt:61-85`](../../Kodex/app/view/application/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/app/ComposerInputTest.kt#L61)。不能写成立即抢占当前工具；安全的真实运行状态仍待 fixture 或实录。 |
| Stop / Clear pending / Resume / Compact | `run-session.md` | `run` | 状态栏按执行状态选择 Stop、Clear pending、Resume，空闲时才显示 Compact；入口分别调用 cancel、clearPending、resume、forceCompact，见 [`RuntimeStatusBar.kt:70-105`](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/RuntimeStatusBar.kt#L70) 和 [`RuntimeStatusBar.kt:529-535`](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/RuntimeStatusBar.kt#L529)。测试只证明 Compact 的可见条件（[`RuntimeStatusBarTest.kt:70-73`](../../Kodex/app/view/application/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/app/RuntimeStatusBarTest.kt#L70)），四种操作的实际运行片段不能用静态快照替代。 |
| 提问表单 | `run-session.md` | `collaboration` | 选项选择后焦点推进，`Other` 进入自由文本，最后显式 Submit；见 [`AgentRuntimeView.kt:91-157`](../../Kodex/app/view/agent/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/agent/AgentRuntimeView.kt#L91) 和 [`AgentRuntimeView.kt:292-334`](../../Kodex/app/view/agent/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/agent/AgentRuntimeView.kt#L292)。`CleanEventViewTest` 有已回答只读表单测试，尚不能证明在线 Agent 会发起该状态；需准备脱敏 pending fixture。 |
| 建议新会话审批 | `run-session.md` | `collaboration` | `Suggested Sessions` 展示任务批次，Accept/Create 与 Reject/可选反馈是批次级决定，接受前配置也是批次级；见 [`AgentRuntimeView.kt:183-235`](../../Kodex/app/view/agent/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/agent/AgentRuntimeView.kt#L183) 和 [`SuggestSubagentTaskPanelTest.kt:126-140`](../../Kodex/app/view/agent/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/agent/SuggestSubagentTaskPanelTest.kt#L126)。实录不能把 Accept 说成任务已完成或自动合并。 |
| 按条目 Fork / Revert | `review-history.md` | `review` | 条目菜单提供 `Fork from here` 与 `Revert to here`，Revert 先确认且明确不可撤销；见 [`SessionTreeCliScreen.kt:1372-1433`](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt#L1372) 和 [`SessionTreeCliScreen.kt:1435-1498`](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt#L1435)。ViewModel 测试覆盖 generation 绑定、Fork 所属会话和取消后确认（[`AgentHistoryActionTest.kt:33-151`](../../Kodex/app/viewmodel/session/src/commonTest/kotlin/io/github/stream29/kodex/cli/session/AgentHistoryActionTest.kt#L33)）；不得解释为文件或 Git 回滚。 |
| Patch 与计划展示 | `review-history.md` | `review` | 工具摘要默认收起 payload，Patch 有 `Changes`/更多行，Plan 显示为清单；见 [`PatchToolEventView.kt:147-185`](../../Kodex/app/view/patch/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/patch/PatchToolEventView.kt#L147) 与 [`CleanEventViewTest.kt:215-241`](../../Kodex/app/view/history/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/history/CleanEventViewTest.kt#L215)。计划/补丁片段必须来自实际结果，不能编造成功状态；现有测试是组件行为证据，不是运行录制。 |
| 输入编辑与目录选择 | `workspace.md` | `input` | 粘贴会规范化换行，Ctrl+Z/Y 撤销/重做，软换行不插入真实换行；见 [`TextInput.kt:100-124`](../../Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/TextInput.kt#L100) 与 [`TextInput.kt:221-240`](../../Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/TextInput.kt#L221)。`TextInputTest` 覆盖 Unicode/粘贴（[`TextInputTest.kt:83-103`](../../Kodex/app/view/components/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/components/TextInputTest.kt#L83)）；目录过滤、Enter 两阶段确认、Escape 分层和 Button8 返回见 [`DirectoryPickerPopup.kt:75-150`](../../Kodex/app/view/path-picker/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/pathpicker/DirectoryPickerPopup.kt#L75) 与 [`DirectoryPickerPopupTest.kt:85-182`](../../Kodex/app/view/path-picker/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/pathpicker/DirectoryPickerPopupTest.kt#L85)。已有隔离实录已验证粘贴/撤销/目录过滤，但软换行、键位配置、Button8 仍须正式录制前复核。 |
| 配置作用域与自动标题 | `configuration.md` | `configuration` | Settings 明确区分 General、Current session、New session；输入键位在 General，当前/新会话配置使用不同入口，自动标题有独立开关、模型和 reasoning；见 [`SettingsPopup.kt:385-433`](../../Kodex/app/view/settings/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/SettingsPopup.kt#L385) 与 [`SettingsPopup.kt:443-466`](../../Kodex/app/view/settings/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/SettingsPopup.kt#L443)。`NewSessionViewModelTest` 覆盖创建时快照和自动标题生成后的更新（[`NewSessionViewModelTest.kt:38-114`](../../Kodex/app/viewmodel/new-session/src/commonTest/kotlin/io/github/stream29/kodex/cli/newsession/NewSessionViewModelTest.kt#L38)）；设置控件已有隔离实录，禁网不能证明标题请求真正生成。 |

## 旧内容去向（保留但降级为参考）

- **安装/发行版**：移至 `install-reference.md` 的第一节，保留版本、平台归档、SHA-256、启动命令和“不需要 JVM”等已核对内容；首页只给一个“获取并启动”链接，不让安装步骤覆盖产品功能。
- **认证**：移至 `connections.md` 的“OpenAI 认证”，并从 `configuration.md` 链接过去。需分别说明 Kodex Browser sign-in 与 Codex CLI credentials：实现和测试显示 Kodex 来源可 Sign in/Sign in again/Log out，Codex 来源只 Reload、由 Codex CLI 管理（[`SettingsPopup.kt:481-501`](../../Kodex/app/view/settings/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/SettingsPopup.kt#L481)；[`AuthenticationSettingsTest.kt:17-97`](../../Kodex/app/view/settings/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/settings/AuthenticationSettingsTest.kt#L17)）。不在无凭证、无网络的演示中伪造成功登录。
- **上下文来源**：移至 `connections.md` 的“Context sources”，保留内置 `~/.agents/`、`~/.kodex/`、`~/.codex/`、Git root、cwd 与手工添加全局路径的实际入口（[`ContextSourcesSettings.kt:31-97`](../../Kodex/app/view/settings/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/ContextSourcesSettings.kt#L31)）。不要把它写成自动读取所有 Home，也不要把 `~/.codex` 与 Kodex Home 混为一处。
- **MCP**：移至 `connections.md` 的独立“ MCP servers”节，保留 global scope、Streamable HTTP/stdio、脱敏状态、详情弹窗、OAuth 与 `Import from Codex` 的边界；当前 Settings 测试确认主层只显示紧凑服务器按钮，详情才显示 URL/操作，导入没有额外 Preview 步骤（[`McpSettingsContentTest.kt:24-153`](../../Kodex/app/view/settings/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/settings/McpSettingsContentTest.kt#L24)）。不把 MCP 概括为“安装插件”，也不承诺首版未实现的 resources/prompts/逐工具审批。
- **Hooks**：移至 `connections.md` 的独立“Hooks”节，明确它是 Kodex 原生按名称/type/command 管理的本地命令；当前 Settings 不显示 Import、enabled 或 matcher，详情不显示 command，编辑时才读取（[`HookSettingsContent.kt:15-63`](../../Kodex/app/view/settings/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/HookSettingsContent.kt#L15)；[`HookSettingsContentTest.kt:17-91`](../../Kodex/app/view/settings/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/settings/HookSettingsContentTest.kt#L17)）。必须保留“命令在本机执行”的安全提示，不写成 Codex Hooks 兼容导入。
- **第一次会话/常见问题/数据位置**：第一次只读任务作为 `index.md` 的短引导，具体下载、找不到命令、登录未完成、模型请求失败、`~/.kodex` 私密数据提示移至 `install-reference.md` 的后半部分；排错页只保留可执行的首诊步骤，不重新把网站做成帮助站。

## 三语言对应关系

- 采用同一组相对文件名和同一章节顺序：`docs/index.md`、`docs/{workspace,run-session,review-history,configuration,connections,install-reference}.md` 与 `docs/zh-CN/`、`docs/zh-TW/` 下完全对应的文件。文件名和页面粒度只是本提案，须先获确认。
- 三个版本每页都保留相同的“用途 → 操作步骤 → 看到的结果 → 限制/边界 → 演示标记 → 源码/版本基线”结构；不得只翻译首页而遗漏参考页或演示说明。
- UI 操作名以 Kodex 当前实际字符串为准（例如 `Stop`、`Clear pending`、`Fork from here`），中文正文可翻译，但第一次出现保留原文；同一 `.cast` 由三种语言共享，步骤文字和 marker 标题分别本地化，不制作三套假界面。
- 每页验收都要同时检查三语言的相对链接、标题层级、代码块原文、播放器 marker、窄屏可读性和“未验证/阻塞”标记；任何语言缺失都不算该页完成。

## 逐页验收清单（提案）

- 首页不以安装/排错为首屏，能从能力地图进入五页主线；无未经确认的最终导航文字。
- 每个主线页面有真实功能说明、最少一段实际 Kodex `.cast` 或明确记录“等待安全 fixture/录制”；不得用手绘 TUI、静态测试快照冒充真实操作。
- 十组细节逐项可追溯到上表，并且每项只指定一个主归属；正文同时写出操作差异和至少一个限制，尤其区分 Check out/Fork/Revert、Stop/Clear pending/Resume/Compact、全局/会话作用域。
- 每段演示记录 Kodex 构建基线、CLI SHA-256、隔离 Home/tmux socket、录制尺寸、marker、是否禁网和私密数据检查；退出帧不作为封面，窄屏不以缩放冒充易读重排。
- 三语言逐页同构，语言切换后相对路径、源码链接和播放器资源都可用；页面显示 Markdown 原文，不引入模拟 TUI 或在线执行后端。
- 浏览器验收覆盖桌面与窄屏、播放/暂停/跳转/marker/重播、无横向溢出、无私密文本和无失败资源请求；此项尚未在本任务运行。
- 实现/测试验收区分“源码静态核对”“单元/组件测试已存在”“本次实际运行”。本轮只完成前两类盘点，未运行 Kodex Gradle 测试、未重录全部状态片段、未验证 VitePress 合入或 Pages。
- 当前 `Kodex/` 子模块 HEAD 为 `89205188d5da39f36b983a0bfb57752977feb21a`；共享录制记录中的探测二进制来自旧基线 `ab54349003db16e6a59c48f63b8b6caf1745441d`，因此正式正文和实录前必须由用户确认发布/录制基线，不能把旧片段当作当前版本证明。

## 仍待用户决定

- 是否接受“五页主线 + 两页参考”的页面粒度，及参考页是否进入主导航。
- 六段演示是否作为首批上限；运行控制、提问/建议、历史 Fork/Revert 是否接受脱敏 fixture 驱动的真实组件/CLI 实录，还是等可安全重现的在线状态后再录。
- `connections.md` 首批是只做实现边界和 Settings 界面说明，还是同时授权认证、MCP OAuth/导入、Hooks 触发的真实演示；当前禁网实录不能证明这些网络/命令执行行为。
- 正式录制和正文应以哪个 Kodex 提交/发行版本为准；是否允许复用旧探测片段（目前只能作为流程和已验证界面路径证据）。
- 页面标题、英文 UI 标签是否按上表保留，以及三种语言的术语表由谁最终确认；本任务不将任一选择记录为已批准指导。
- 主 Session 验收：提案与覆盖映射可供确认；“探测二进制来自旧基线”缺少构建来源证据，需改为记录检查时 HEAD 与二进制哈希、明确二者对应未知。见[首批验收](../../shared-context/findings/docs-subagent-acceptance-2026-09-08.md)。
