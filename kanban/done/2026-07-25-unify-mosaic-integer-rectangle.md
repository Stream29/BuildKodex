# Task Tree

- [done] 统一 Mosaic 整数矩形模型
  - [done] 将 ClipBounds 使用点迁移到 IntRect
  - [done] 合并裁剪所需的 intersect 与 contains 能力
  - [done] 删除 ClipBounds
  - [done] 消除 Surface 裁剪热路径上的临时 IntRect
  - [done] 刷新 ABI 并运行相关测试

# Details

- 保留公开的 `ui.unit.IntRect` 作为通用整数矩形。
- 不再维护裁剪专用的同形数据类型。
