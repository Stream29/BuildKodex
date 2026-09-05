# Task Tree

- [done] 将通知脚本移出PATH
  - [done] 迁移脚本并更新两个Hook命令
  - [done] 验证补全与通知入口并同步记录

# Details

- 用户授权将通知脚本放入`~/.kodex/`，避免影响Bash的`kodex-`补全。
- 移至`~/.kodex/hooks/notify`，不保留旧脚本或链接，不修改PATH、Bash配置或Broker服务。
- IDEA打开BuildKodex；脚本位于项目外，本次只迁移路径和更新usage。
- 新路径不存在；两个Hook都引用旧路径。用Bash语法检查、命令补全候选和实际通知调用验证。
- 按用户确认的目录迁移路线执行，不改产品代码。
- 迁移完成，脚本保持可执行；Bash语法检查通过，`compgen -c kodex- | sort -u`只返回`kodex-cli`。
- 两个新路径通知入口实际调用成功，Stop返回finish，Broker保持active；未修改PATH或Bash配置。
- 已同步[本机通知记录](../../shared-context/findings/local-kodex-notification-hooks.md)，未提交Git。
