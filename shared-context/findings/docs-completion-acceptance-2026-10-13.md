# UI showcase 收尾补录验收

## 基线与范围

- 本轮从主仓库 `95fd143`、Kodex `2b16ae50`、KodexDocs `19a0168` 接手三张既有 executable 卡，只修改 Kodex 测试侧录制场景、KodexDocs 单页展示及这些卡的验收记录。未修改产品运行时代码、Daemon 设计、用户真实会话或个人配置；验收阶段未提交、推送、部署或发版。
- 原十二段录制及其 manifest 哈希保留；三语言仍各一页，新增三段后每页十五段。原片段的来源基线仍为 `89205188`，本轮三段单独标记当前 Kodex `2b16ae50`；不把旧录制当成当前源码的全面重验。
- 三段均在 remote MacBook 的专用检出、临时 filesystem Home 和进程内 RPC 上运行，外部模型客户端为测试 mock，不挂载认证、私人 Home 或原始 Session。录制输出来自实际生产 Compose 组件；时间轴每检查点两秒，不冒充实时耗时。

## 覆盖结果

| 片段 | 实际操作与断言 | 边界 |
| --- | --- | --- |
| `suggestions-accepted` | 完整应用壳层接收 mock 模型返回的建议工具调用；在生产目录选择器中进入 `demo-project` 并确认批次 cwd；点击 Accept 后，真实后端创建两个可丢弃 Session，应用自动打开两个标签；选择其中一个可看到提示和 mock 回答。 | 不是在线付费模型或真实父会话；不把建议接受当成子任务已完成。 |
| `session-management` | 完整壳层中重命名、关闭标签、从会话目录重新打开、Close and archive、显示已归档、Unarchive、删除取消和确认；测试检查标签与目录/持久状态。 | 所有对象位于临时 Home；关闭标签不是删除会话，归档不等于停止运行。 |
| `history-invalidation` | 完整壳层中先打开历史菜单，再启动 mock 模型的可控运行回合，菜单消失；运行完成后打开 Revert 确认框，切换到另一标签后确认框消失且历史动作被取消。 | 当前 UI 通过关闭失效菜单/确认框而非保留禁用按钮表达不可操作；未覆盖所有可能的目标失效原因。 |

- 关键源码与导出测试：[建议会话补录](../../Kodex/app/impl/view/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/DocsCompletionRecordingTest.kt)、[工作台补录](../../Kodex/app/impl/view/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/DocsSessionManagementRecordingTest.kt)、[历史失效补录](../../Kodex/app/impl/view/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/DocsHistoryInvalidationRecordingTest.kt)。
- [manifest](../../KodexDocs/docs/public/recordings/manifest.json)记录每段 cast、生成器及旧片段边界的哈希；新增脚本 `KodexDocs/scripts/update-completion-manifest.py`只更新本轮三段，不覆盖旧片段来源。

## 验证与遗留

- 2026-10-13 用户查看临时预览后接受三段素材，并确认以当前产品“失效时自动收起”而非灰色禁用项作为历史场景的本轮验收边界；三张原 executable 卡已移入 `kanban/done/`，文件名保持不变。本确认不扩大模型、旧片段、其他失效原因或发布的验收范围。
- 用户随后单独授权提交推送本轮工作；Kodex 测试侧提交为 `27897e20`，KodexDocs 单页展示提交为 `db46a86`，均已推送 `origin/main`。主仓库只记录两子模块指针、三卡归档和本验收记录；不包含同时在主仓库工作区修改的 Daemon/discussion 文件。没有部署或发版。
- 验收后停止 remote MacBook 预览、SSH 本地转发及本轮启动的 Gradle daemon，删除专用 `~/ACodeSpace/local/BuildKodex-docs-session799/` 检出；本地 `out/docs-completion/` 仅保留最终验证日志、浏览器结果和逐帧文本证据，清除重复日志及 cast 副本。正式三段 cast 保留在 KodexDocs 中。
- remote MacBook 使用 Java 25、独立 Kodex 检出，三项新 JVM 测试一次定向重跑全部通过，0 失败/跳过；日志 `out/docs-completion/all-completion-jvm-final.log`。临时 Home 由测试清理，未使用个人模型凭据。
- remote MacBook 的 `npm run check`：构建及 14 项站点测试通过；日志 `out/docs-completion/docs-check-final.log`。本地独立 headless Chrome 对三语言 × 五宽度检查 15 个自动循环、`inert` 且无播放控件的播放器；新增三段共 33 个原帧与浏览器逐行匹配，每段至少两次循环，页面异常和失败请求为零。证据 `out/docs-completion/browser-verification.json`。
- 修改前的当前 Kodex 基线全量 `:app-impl-view:jvmTest` 为 54 项、2 项旧 patch 录制测试失败，均在 `DocsRecordingTest.kt`，日志 `out/docs-completion/baseline-jvm.log`。这不是三项新场景失败；尚未判定是产品缺陷还是旧测试预期漂移，也未擅自修复。
- 尚未重新逐帧验收旧十二段于当前产品版本；未验证真正在线模型、跨系统字体像素一致性或未录入的其他历史失效原因。
