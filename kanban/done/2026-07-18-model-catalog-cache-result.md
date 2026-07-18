# Task Tree

- [done] 将CLI缓存刷新结果改为完整缓存对象
  - [done] 以`CodexModelsCache?`替换布尔返回值
  - [done] 将缓存和远端成功结果原样发布到当前目录
  - [done] 更新测试并验证模型目录模块

# Details

`null`仅表示`models_cache.json`不存在；存在的缓存无论模型列表是否为空都返回完整的解析结果并替换当前目录。远端成功响应同样原样发布。
