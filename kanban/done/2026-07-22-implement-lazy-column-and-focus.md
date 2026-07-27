# Task Tree

- [done] 实现通用LazyColumn与焦点协作
  - [done] 实现普通`verticalScroll`布局消费者
  - [done] 定义`LazyListScope`和稳定唯一key规则
  - [done] 实现长期存活的`LazyListState`
  - [done] 实现可变高度lazy measure、placement和overscan
  - [done] 实现stable-key位置恢复与fallback归一化
  - [done] 实现宽高变化、内容变化和高度缓存失效
  - [done] 暴露`LazyListLayoutInfo`与可见item信息
  - [done] 接入滚轮与paging输入
  - [done] 在Mosaic实现focus-search extension与relocation handshake
  - [done] 由LazyColumn提供beyond-bounds搜索和bring-into-view
  - [done] 覆盖可变高度、key迁移、resize、composition数量和跨视口焦点测试
  - [done] 运行相关格式化与多平台测试

# Details

- 状态：已完成。JVM、Linux x64和macOS ARM64测试、Mosaic格式检查与API检查均通过。
- `LazyColumn`接受任意Mosaic composable，不理解conversation或文本换行。
- 公开位置为首个可见item index和item内行偏移。
- 显式request定位优先于自动key恢复。
- 首版不支持无界滚动轴、intrinsic height、`reverseLayout`或`animateScrollToItem`。
