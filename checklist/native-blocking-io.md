# Native Blocking IO

- 阻塞式IO资源的所有权、取消交接和清理遵循[Coroutine Resource Lifecycle](coroutine-resource-lifecycle.md)。
- Kotlin/Native的阻塞式文件、子进程与管道操作必须运行在`Dispatchers.IO`，不得占用`Dispatchers.Default`。
- Native IO dispatcher不得增加Kodex自有的并发上限；使用`Int.MAX_VALUE` view并保留kotlinx.coroutines运行库的内部安全上限。
- 保留按功能命名的dispatcher view，便于运行时指纹区分FileIO、ProcessClientIO、ProcessIO与ShellPipeIO。
