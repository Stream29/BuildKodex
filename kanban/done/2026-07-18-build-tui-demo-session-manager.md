# Task Tree

- [done] 实现演示会话编排
  - [done] 从`CodexCliStorage`加载本地鉴权和模型缓存，并建立共享OpenAI客户端与模型目录
  - [done] 为每个新会话建立独立的内存storage、AgentState、AgentRuntime和工具资源
  - [done] 实现新建会话、会话切换和会话关闭
  - [done] 实现从完成turn快照fork新会话
  - [done] 实现当前会话checkout并同步清除被截断的瞬态展示状态
  - [done] 实现模型、reasoning和plan mode的设置更新
  - [done] 实现多轮消息发送、流式事件投影和现有工具runtime组合

# Details

会话管理器是TUI与core之间的适配层。持久化真相仍在每个会话的AgentStorage中；
只有尚未提交的流式输出和用于展示的完成turn快照列表属于临时UI会话状态。

`fork`创建新storage/thread，`checkout`修改当前storage/thread。checkout不会撤销
此前工具造成的外部文件系统副作用。所有会话在进程退出时关闭并丢弃内存状态。
