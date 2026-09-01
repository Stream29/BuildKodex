# Coroutine Resource Lifecycle

- 每个外部资源必须有唯一、显式的协程所有者；普通对象不会因`CoroutineScope`结束而自动释放，必须把清理注册进所有者的结构化生命周期。
- 长生命周期资源必须是创建方scope中可独立取消的子节点；父scope结束时级联取消，所有者Job只能在全部child和资源清理完成后结束。
- 短生命周期资源必须归属当前operation coroutine，并在该operation正常返回、失败或取消前完成释放；不得把文件句柄等临时资源提升到Session或Application生命周期。
- 资源清理必须放在有限的`NonCancellable`收尾中；只能屏蔽close、release、rollback等必要清理，不能用它维持正常工作或掩盖错误的任务所有权。
- 可挂起的`close()`不得只继承已经取消的调用方Job；其底层释放动作必须在取消后仍实际执行。
- 不得通过可取消的dispatcher或promise返回刚取得的资源，除非取消交接能取得该资源、完成释放并等待清理结束。
- 优先提供scoped open/use/close API，使资源不能逃逸operation边界；必须返回handle时，API需要显式绑定owner并提供可等待的关闭完成语义。
- 清理失败必须保留原始失败；存在原始失败时将清理失败附加为suppressed failure，不得用清理异常覆盖原始取消或业务异常。
- 生命周期测试必须覆盖取得资源时取消、使用期间取消、正常关闭、清理失败和owner结束，并验证owner完成时资源已经释放。
