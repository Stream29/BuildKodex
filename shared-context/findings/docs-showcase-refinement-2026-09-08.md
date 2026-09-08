# 真实场景补录与网页动画验收

## 最新：动图式循环

- 用户要求自动播放、自动重播且不可操控。十二段统一使用播放器原生 `autoPlay: true`、`loop: true`、`controls: false`，挂载节点加 `inert`；没有新增控制条、说明文案或手写循环逻辑。
- 原生循环从零开始，不沿用旧 `startAt`。结果录屏将前 190.1 秒准备输出的时间压到零；后续间隔保持，所有事件类型、顺序及终端输出不变。原始 cast 保留于 `out/docs-live-current/` 和 KodexDocs 提交 `0c071b2`，编辑脚本及前后哈希记入 manifest。
- 构建及 14 项站点测试通过。开发/生产预览的三语言、五宽度、十二个无控件播放器、鼠标/键盘/焦点禁用、持续播放和语言导航通过。
- 十二段各在独立验证页以 50 倍速检测两次循环回绕；这是加速循环验证，不冒充自然时长的完整观看。结果录屏十五个检查点与原终端逐行一致，原始输出串哈希一致。
- 证据：`out/docs-animation/verification.json`、`out/docs-animation/browser.log`、`out/docs-animation-check.log`。没有新模型调用或产品代码变更；本次提交前另重跑应用视图和会话 ViewModel 共 96 项 JVM 测试，全部通过，见 `out/docs-precommit-jvm.log`。

## 后续页面收敛（循环改造前）

- 三语言各只保留一页 UI showcase；十八个旧页面已删除，包括 Installation、连接和独立参考页。安装入口改为 `Kodex/README.md` 的一句 Agent prompt。
- 页面只留演示标题/播放器与用户要求的 Codex 复用说明；自动来源字幕、窄屏提示、额外说明段落已删除。资产种类仍在 manifest 中区分，不把组件片段改记为原生录制。
- Codex 默认认证来源见 [KodexGlobalSettings.kt:34](../../Kodex/app/shared/settings/contract/src/commonMain/kotlin/io/github/stream29/kodex/cli/settings/KodexGlobalSettings.kt#L34)，Codex Home 上下文默认启用见 [AgentContextSettings.kt:31](../../Kodex/agent-context/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentcontext/contract/AgentContextSettings.kt#L31)。MCP 是设置中的显式导入，不宣称整份 Codex 配置自动迁移。
- `results-history` 的原始 131–190 秒为尚未打开目标 Session 的工作台持续重绘；输出频率约 10 Hz，因此 `idleTimeLimit` 不能跳过。采样证据在 `out/docs-showcase/lead-frames.json`。
- 默认播放用播放器原生 `startAt: 190.1` 从目标历史画面开始；封面 `npt:63.067` 使用跳过空闲后的播放时钟。未改写原始输出文件、时间或哈希，也没有增加手写剪辑/动画逻辑。
- 收敛后重新构建及 14 项测试通过。生产预览和开发服务：三语言 × 五宽度、每页十段播放器、无字幕、语言切换保留章节锚点、hover/钉住/Escape、移动端目录和无 JS 正文通过；十八个旧生产路由为 404。
- 实际点击结果播放器后，第一秒已有目标 Session 历史、第五秒画面继续变化，不再等待开头连续重绘；证据 `out/docs-showcase/verification.json`。移动端播放器覆盖层曾拦截目录，给播放器设置 `isolation: isolate` 后复验通过。
- 本次仅修改网站/README/记录，未重跑 JVM 测试、未重新安装 CLI、未新调用模型；以下 JVM 与完整录制逐帧核对保留前轮结果。站点日志 `out/docs-showcase-check.log`。

## 本轮原生补录资产与来源（后续又新增 Hooks/MCP）

- 本轮由主 Session 完成；无新增子代理、提交、推送或发布。当前 10 段资产为 6 段原生 CLI、4 段组件 fixture；[manifest](../../KodexDocs/docs/public/recordings/manifest.json)记录种类、哈希与授权边界。
- 源码基线 `89205188d5da`，CLI SHA256 `99882904ad4d20eb65e00dffd983b77aec4893ef3ca4bdfcdb8684a6f4800af3`。不是发行版兼容承诺。
- 用户明确授权只读本地特性开发会话及认证。选取 Session 178「侧栏history index」中连续 13 条稳定历史（索引 697–826），翻译人类可读内容，保留四类事件、顺序、计划状态和已回答选项；不是完整会话翻译。
- [英文测试资源](../../Kodex/app/view/application/src/jvmTest/resources/docs/history-index.en.json)的 SHA256 为 `e84ca6c7c70db12cfbcc529eef8a07de07343b4884e77e1ecff8a415fb6d9c11`。来源及原文件哈希留在本机 `out/docs-live-current/history-source.json`；原始 Session 没有被演示 CLI 打开或修改。

## 三段原生实录

- `execution`：载入英文历史，在独立 Home/project 中实际提交新回合、追加并消费 steer、Stop/Resume、Clear pending 真实待回答问题、完成真实 Compact。模型修改两个可丢弃示例文件，实际检查结果为 4 项通过；不是修改 Kodex 生产代码。
- `results-history`：一段展示消息、计划、真实工具 Arguments/Result、两文件 Changes、已回答问题、失败与压缩结果，以及 Fork、Revert 取消/确认。存储证据证明取消不变、确认保留到目标索引、fork 不受影响；不回滚项目文件。旧四段结果录制已退出正式注册表。
- `history-index`：英文原历史及实际续篇，展示用户/计划/助手最终回复/已回答问题四类预览、移入浮窗保持、移出关闭、索引右键菜单、Check out 定位及回到底部。没有强制保持测试 hover 状态。
- 原始输出未改写。`execution` 默认 2 倍速；三段均使用播放器原生 `idleTimeLimit: 2` 跳过长空闲间隔。原始时长约 434/724/111 秒，不把压缩空闲后的播放长度当作模型耗时。

## 隔离与复用

- [record-live.py](../../KodexDocs/scripts/record-live.py)在当前机器使用 bubblewrap、tmux、asciinema；只读绑定用户明确允许的认证文件，原 Home/Session 不挂载。清空继承配置；仅演示 Home/project 可写。
- 录制需要共享网络以调用模型；旧 `record-native.py` 的禁网录制仍用于工作台、输入、配置。启动前检查沙箱内 `/bin` 链接、Python 和示例文件，不降级到个人环境。
- 录制为输出事件加退出事件，不捕获键入。三段资产检查无原 Home 路径和实际认证 token；演示证据只保存项目及历史，不保存认证副本。
- 本机证据：`out/docs-live-current/` 的 cast、frames、provenance、`execution-evidence/` 和 `results-actions.json`。结果与索引续录使用同一独立环境，控制器来源哈希记录在 sidecar。
- 第一次未满足预期的执行尝试未进入站点；修正沙箱入口并通过预检后重新执行。可复用命令与只读认证刷新限制见 [README](../../KodexDocs/README.md)。

## Hover 与网页动画

- 原生历史预览已验证四类内容、浮窗交接及离开关闭；见 [SessionSidebar.kt:393](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionSidebar.kt#L393)、[707](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionSidebar.kt#L707)。
- 终端条目完整命令/ID 预览由现有 `terminal-sessions` 组件录制覆盖；不等于真实 OS 进程关闭。侧栏边缘、分隔线和设置菜单使用既有原生片段。
- 公共控件 hover/pressed/selected/disabled 和 PreserveColors 的样式规则见 [TuiInteractionStyle.kt:17](../../Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/TuiInteractionStyle.kt#L17)。源码核对不等于每种控件每条样式分支都已成片；文字匹配也不能单独证明全部颜色和加粗效果。
- 网页目录使用 Vue Transition 和 CSS：桌面宽度过渡、移动端淡入/收起、点击钉住、Escape、减少动画偏好均经过实际浏览器检查。采样帧保持贴左，无横向溢出；证据 `out/docs-live-current/animation-verification.json`。

## 验证与边界

- JVM：`:app-view-application:jvmTest --rerun` 成功，87 项测试；本轮未重新运行前轮全部 123 项集合。日志 `out/docs-history-refresh-test.log`。
- 站点：`npm run check` 构建及 49 项测试通过。日志 `out/docs-live-site-check.log`。
- 浏览器：三语言 21 页、五种宽度；原文/原文资源、语言切换、播放器播放暂停、无 JS 正文和侧栏交互通过，无页面错误/失败请求。
- 10 段录制共 121 个原终端/组件检查点匹配；新三段分别 13/15/10 个。证据 `out/docs-live-verification/verification.json`。页面验证对应当时的七页结构。
- 尚不声称所有分支成片：建议会话接受结果、会话目录管理、历史运行中/目标失效禁用状态、长补丁分页仍有展示缺口；新合并片段不包含旧长补丁分页片段。
