# Task Tree

- [done] 拆分CLI基础组件
  - [done] 盘点当前界面中的一次性控件实现
  - [done] 明确并提取Button组件
  - [done] 明确并提取LazyColumn组件
  - [done] 迁移调用方并补齐通用行为测试

# Details

基础组件需要有独立契约和状态语义，不能继续散落在应用界面中。

Button已从Pressable文件独立；LazyColumn使用固定单行item并支持首尾锚定、滚轮、分页和首尾跳转。聊天历史采用末尾锚定，调用方继续持有滚动offset。
