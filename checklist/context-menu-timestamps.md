# 右键菜单日期时间

- 范围仅为 Session Catalog、已持久化 Session Tab、主历史 Message、History Index Message；保留各入口原有开启条件和动作，不扩展非 Message、WorkGroup、pending/streaming 或 New Session。
- 在 Index 只读信息块内分行显示时间，不动态插入独立菜单项，以免加载后改变动作位置和焦点。主历史 Message 菜单同时显示 Index。
- Message 标签为 `Timestamp`；Session 标签为 `Created at`、`Updated at`。格式使用运行 CLI 设备的本地时区、24 小时制、`YYYY-MM-DD HH:mm:ss UTC±HH:mm`；按该事件实际偏移处理夏令时。时间类型遵循[日期与时间](datetime.md)。
- Message 必须读取对应 storage index 的 `timestamp.getExact(index)`，不得使用稀疏 timeline 的前值代替。
- 已打开 Session 从自身缓存 timeline 执行 `ceilToIndex(0)` 后精确读取首时间；按索引顺序取首条，不遍历求最小墙钟值。
- Catalog 的 Created at 只精确读取 `timestamp[0]`；缺少 0 时隐藏字段，不扫描后续记录、不为此打开 runtime 或增加缓存/指针。接受此时 Tab 与 Catalog 的 Created at 可见性不同。
- Updated at 取当前 timestamp timeline 最新记录，与 `lastActivityAt` 同义；保留 fork 继承、revert 回退和 rename/settings 既有写入语义，不新增或排除审计时间。
- 每次打开菜单使用独立 request 身份并重新读取；在本次打开期间保留快照，不持续订阅时间变化。通过准确 ViewModel 异步读取，避免渲染线程存储 I/O。
- 加载中、精确时间缺失或读取失败，均只隐藏对应时间字段；不显示占位，不影响 Index、已有动作、可读内容或其他成功字段。协程取消继续传播。
- 复用 Session/Agent/Message 身份、history generation 和 anchor 校验；目标失效时关闭菜单或丢弃结果。
- 窄菜单允许裁剪时间，不要求保留特定日期部分或增加展开入口。验证异步加载保留动作焦点、鼠标和键盘使用同一菜单路径。
- 验证四入口的精确取值、失败隔离、重开刷新、目标失效，以及实际 CLI 中的窄终端行为；不能用编译或单元测试代替运行验收。
