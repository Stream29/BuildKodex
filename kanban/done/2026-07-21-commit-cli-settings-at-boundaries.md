# Task Tree

- [done] 按边界提交CLI设置
  - [done] 将界面编辑状态与已提交的`KodexAgentSettings`分离
  - [done] 在提交prompt、离开session和退出应用时提交一次设置
  - [done] 合并边界内的多次设置调整
  - [done] 覆盖提交边界和无变化场景测试

# Details

前端遵循update-commit：用户可连续调整plan mode等选项，边界到达前不向AgentStorage重复提交。

界面通过带revision的暂存配置即时投影；边界命令捕获对应revision并串行提交。Linux和macOS原生平台各通过30项CLI测试。
