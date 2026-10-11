# Task Tree

- `Locate the extra Ultrafast UI copy and its design record`() // complete
- `Use name-only tier choices and remove settings supporting copy`() // complete
- `Review scoped changes and regression coverage`() // complete; static validation only

# Details

- 用户指出 Ultrafast 自带的说明文字不符合设计且完全没有必要。
- Runtime 和建议子 Session 的档位菜单统一只显示名称；Current session
  和 New session 设置移除 Ultrafast 专属 supportingText。
- 请求档位、模型能力、权限处理、实际返回档位 Warning 及错误出口保持不变。
- 同步 Runtime spec 和原 Ultrafast 任务的展示要求；回归断言纯名称、
  不显示用量/权限说明，并保留显式点选后提交 Ultrafast 的检查。
- 修改前 Kodex 工作区干净；外层已有其他任务修改，不纳入本次变更。
- IDEA 已打开 BuildKodex，已通过 MCP 确认；不切换用户窗口或启动 Sync。
- 本机未检测到可复用 Gradle Daemon；不为此次纯展示修正启动重型构建。
  静态检查不能视为自动化测试执行。
- `git diff --check` 通过；四处 view 主源码中已无额外用量/权限提示文案。
  IDEA 对 Runtime 与建议子 Session 的菜单源码检查均未报告错误。
- 四处渲染回归断言已更新，包含设置页选择 Ultrafast 后仍无说明文字；
  保留写入档位与 tuple 的断言。自动化测试本轮未执行，未构建新 CLI。
- 原交付仅静态验证，未获提交授权；用户随后于 2026-10-12 明确要求将本地剩余工作区分批提交并推送。
- 本次代码提交为 `4940c2ad`（`fix: keep Ultrafast tier choices name-only`），包含四处展示、对应回归断言及 Runtime spec；不新增运行验收或发版结论。
- 不发布新版本或替换已安装 CLI，自动化测试仍按上述“未执行”边界记录。
