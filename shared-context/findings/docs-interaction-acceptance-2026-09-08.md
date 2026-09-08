# 新版说明书：逐交互验收

- 本文件保留前轮 13 段/122 检查点的历史验收；历史索引、执行和结果录制已被后续真实场景替换，当前结论见[补录验收](docs-showcase-refinement-2026-09-08.md)。

## 交付与结论

- 用户批准七页结构；三语言共 21 页已合入 `KodexDocs/docs/`，13 段演示已接入。保留原文 Markdown、语言代码、贴左全宽和 hover/钉住导航。
- 本轮由主 Session 自行检查，没有启动新子代理。只完成本地修改，不提交、推送或发布；预览入口 `http://127.0.0.1:4173/KodexDocs/zh-CN/`。
- **主要场景可观看，不等于每条交互分支都已展示。** 下表区分原生操作、真实组件/离线状态、尚未展示；看见菜单不算操作完成。
- 版本与全部资产哈希以 [manifest](../../KodexDocs/docs/public/recordings/manifest.json) 为准：源码 `89205188d5da` 加确认框测试可见性变更，不是已发布版本证明。

## 实际展示清单

原生帧号从 1 开始，对应 `out/docs-native-current/*.frames.json`；组件帧号从 0 开始，对应 `out/docs-component-final/*-N.txt`。播放器时间对应保存在 `out/docs-final-verification/verification.json`。

| 交互 | 片段 / 帧 | 验收边界与实现依据 |
| --- | --- | --- |
| 左右栏 hover/移开收起、独立钉住/取消 | workspace 2–7、12–13 | 原生 CLI 键鼠操作；[SessionTreeCliScreen.kt:96](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt#L96) |
| 左右宽度拖动、内容选择 None、菜单保持未钉住栏展开 | workspace 8–11、14–15 | 原生 CLI；[SessionSidebar.kt:148](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionSidebar.kt#L148) |
| 有数据的历史悬停预览、右键 Check out 定位 | history-index 0–3 | 真实 InMemory 历史与组件；断言定位不删除历史。测试单独保持预览请求，不能代替完整壳层的延迟关闭验证；[导出测试:89](../../Kodex/app/view/application/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/DocsHistoryRecordingTest.kt#L89) |
| 阅读旧历史时新条目不抢位置、[↓] 恢复跟随 | history-index 3–5 | 真正追加第 41 条并检查滚动状态；[导出测试:105](../../Kodex/app/view/application/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/DocsHistoryRecordingTest.kt#L105) |
| 终端条目完整命令/ID 预览、Close session 菜单及条目移除 | terminal-sessions 0–3 | 真实组件，关闭的是测试句柄，未执行或终止 OS 命令；[导出测试:41](../../Kodex/app/view/application/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/DocsWorkspaceRecordingTest.kt#L41) |
| 标签溢出滚动、选择末项、非当前标签右键不切换当前项 | session-tabs 0–3 | 真实标签组件与草稿 ViewModel；Rename/Close 仅显示；[导出测试:127](../../Kodex/app/view/application/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/DocsWorkspaceRecordingTest.kt#L127) |
| 多行中文粘贴、软换行、宽字符边界编辑、Ctrl+W/Z/Y | input 2–7 | 原生 CLI；实际插入 `第\|二行` 后撤销，不只是光标截图；[TextInput.kt:82](../../Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/TextInput.kt#L82) |
| 目录过滤/进入、Button8 返回、单独确认、分层 Escape | input 8–16 | 原生 CLI；[DirectoryPickerPopup.kt:97](../../Kodex/app/view/path-picker/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/pathpicker/DirectoryPickerPopup.kt#L97) |
| 运行中实际键入/提交至 Pending steer、Stop/Clear pending/Resume/Compact | execution 0–8 | 真实 Composer/运行界面，执行状态由 fixture 提供；Clear pending 后 steer 保留；不证明模型执行或压缩结果；[导出测试:242](../../Kodex/app/view/application/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/DocsRecordingTest.kt#L242) |
| Other 输入、下一题焦点、选项、回改草稿、显式提交 | questions 0–6 | 真实表单和键鼠；提交回调由断言验证，录制未展示在线 Agent 收到答案；[导出测试:112](../../Kodex/app/view/application/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/DocsRecordingTest.kt#L112) |
| 建议完整提示、批次提问模式/模型分层设置、反馈拒绝 | suggestions 0–8 | 真实面板/菜单与 fixture；只录拒绝路径，不算已接受/创建 Session；[导出测试:171](../../Kodex/app/view/application/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/DocsRecordingTest.kt#L171) |
| 工具摘要、Arguments/Result、计划 [ ]/[>]/[x] | history 0–3 | 真实渲染器，`example_tool` 明示测试数据，不是命令成功证明；[导出测试:321](../../Kodex/app/view/application/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/DocsRecordingTest.kt#L321) |
| 待执行 Patch、Changes 增删行、更多行分页 | patch 0–2；patch-pages 0–4 | 真实补丁组件。实际点击最多 200 行和最后 6 行，显示第 405 行；未改文件；[导出测试:65](../../Kodex/app/view/application/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/DocsRecordingTest.kt#L65) |
| 条目右键 Fork，显示独立 [fork] 标签并保留源历史 | history-actions 0–3 | 真实 InMemory Session.fork，不是仅回调标志；按历史索引检查边界，避免把标题设置条目误算历史；[导出测试:184](../../Kodex/app/view/application/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/DocsHistoryRecordingTest.kt#L184) |
| 条目右键 Revert，确认/取消/再次确认及删除后续历史 | history-actions 4–9 | 实际菜单入口、生产确认框和 InMemory 回退；取消仍有第 5 条，确认后保留第 4 条；未操作文件或私人历史；[导出测试:195](../../Kodex/app/view/application/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/DocsHistoryRecordingTest.kt#L195) |
| 新默认值不回写已有草稿，后开草稿复制默认值 | configuration 6–9、17、24–25 | 原生 CLI；状态栏改当前目标不改默认值；[NewSessionViewModel.kt:27](../../Kodex/app/viewmodel/new-session/src/commonMain/kotlin/io/github/stream29/kodex/cli/newsession/NewSessionViewModel.kt#L27) |
| 自动标题开关、独立模型/推理强度、禁用控件与重新启用 | configuration 10–16 | 原生设置操作，不展示模型生成标题；[SettingsPopup.kt:536](../../Kodex/app/view/settings/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/SettingsPopup.kt#L536) |
| General 换行键、状态栏 Model→Reasoning→Service tier、ask/no question | configuration 3–5、18–25 | 原生菜单及设置页核对；[RuntimeStatusBar.kt:201](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/RuntimeStatusBar.kt#L201) |

## 尚未展示，不记作通过

- 建议批次 Accept 后实际创建/打开新 Session；建议批次工作目录选择也未在该片段操作。
- 会话目录重新打开、重命名、关闭、归档/恢复、删除确认及结果；目前正文说明与标签菜单不能代替这些操作。
- 空闲首次提交与运行中 steer 的完整对照片段；当前运行控制片段只用 fixture 演示后者和控件状态。
- 完整应用壳层中历史预览跨区域移动、延迟关闭的生命周期；组件测试保留请求只验证内容与定位。
- 执行中/历史目标失效等所有禁用分支，没有逐个录成片；本轮相关 ViewModel 测试通过不等于画面已展示。
- 在线模型消费 steer、实际 Stop/Resume/Compact 结果、真实标题生成；真实终端进程关闭；认证/MCP/Hooks 外部结果未做运行验证，没有使用私人账号补齐。
- 宿主终端 Shift+拖拽选择不在 ANSI 输出中；网页播放器选字不是该操作证据。手机等比缩小不等于可读性验收，页面提示横屏/全屏。

## 构建、测试与实际运行

- 原生 CLI `linkReleaseExecutableLinuxX64` 成功：`out/docs-cli-final-build.log`。只将 Revert 确认框改为 internal 供复用，无产品行为修改。
- JVM 实际重跑：application 87、session 9、agent 11、patch 16，共 **123 项、0 失败/跳过**；日志 `out/docs-final-kotlin-checks.log`。
- 修正三项旧测试：Revert 目标先物化；Fork 标题期待 `[fork]` 前缀。没有放宽产品目标校验。未宣称全仓库/四平台测试通过。
- `npm run check`：生产构建和 **48 项测试通过**；检查 21 页原文/SSR、资源/链接、13 段哈希、事件种类和常见私密标记。日志 `out/docs-site-final.log`。
- 浏览器：21 页 × 1440/1024/768/390/320 px；原文和下载内容一致、三语言、导航/hover/钉住/Escape/触摸、无 JS 退化、播放暂停、SPA 切换无重复播放器，无横向溢出/控制台错误/请求失败。
- 同版本播放器逐行核对 **122 个检查点**：原生 16/17/26，组件 9/7/9/6/3/5/10/6/4/4；证据 `out/docs-final-verification/verification.json`。首次验证用 12 ms 等待绘制漏配 Undo 帧；改成等待两个浏览器动画帧后全量重跑通过，没有改动录制来匹配。
- 开发态新 Markdown 增加/编辑/删除无需重启；采用 Vite 正向 glob，不保留试验用轮询或自写 watcher。临时 `constructor` marker 与 literal script 页面也验证为原文且无播放器/脚本执行，已删除并重建。
- 最终逐帧检查是终端行文字一致性，不宣称跨设备字体、默认调色板或鼠标指针像素一致。ANSI 录制不包含系统鼠标指针。
- 验收后停止专用 4175 静态服务，清理临时预览映射、下载录制器和本轮旧中间录制；保留最终 sidecar、帧、日志和回放证据。4173 开发服务继续运行，最终请求返回 HTTP 200；看板/发现文件的 151 个本地引用检查通过。
