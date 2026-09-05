# Task Tree

- [done] 配置Kodex专用桌面通知
  - [done] 核对现有通知链路并确认清理Codex注册
  - [done] 改名通知脚本、Broker与桌面身份
  - [done] 配置Stop与纯文本异常Hook
  - [done] 验证真实通知、配置及旧入口清理

# Details

- 用户要求配置异常Hook，并将原Stop通知改为Kodex专用、不兼容Codex。
- 用户确认删除`~/.codex/config.toml`中旧通知Hook注册；其他配置不动。
- 复用现有GJS Broker，不新增服务层；统一使用`io.github.stream29.Kodex`桌面身份。
- 本机脚本接受`stop`或`unhandled_error`参数；Stop仅解析Kodex JSON，异常直接读取stdin纯文本。
- 校验脚本、TOML/YAML、systemd与D-Bus；用真实桌面通知验证两个入口。
- 不改产品源码，不启动新版迁移真实Home，不提交Git。
- 已确定修改五处本机通知文件与两处配置；删除旧Codex注册及其信任状态，重载并启用改名后的用户服务。
- IDEA打开BuildKodex；本次通知脚本位于项目外，使用原生工具验证。
- 新服务已启用；Bash、desktop文件、systemd单元及Codex TOML检查通过，D-Bus接口可用。
- 实际调用两个脚本入口，D-Bus观测确认通知名称/desktop身份为Kodex；异常Unicode、多行、末尾换行与markup转义正确，Stop返回finish，旧Codex JSON被拒绝。
- 0.4.2预备二进制使用当前配置副本在隔离Home成功启动/退出；真实Home仍为0.4.1，未触发真实会话异常或模型请求。
- 旧服务为not-found，新服务active且enabled；没有遗留的旧通知路径引用。
- 结果见[本机通知配置](../../shared-context/findings/local-kodex-notification-hooks.md)；未修改产品源码或提交Git。
