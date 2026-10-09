# QuickWin · 架构与需求文档索引

> 本目录是 QuickWin 项目的**唯一需求与架构事实来源**。
> 全部需求记录完成前不写实现代码；实现阶段严格按本目录文档执行。

## 文档列表

| 文档 | 内容 |
| --- | --- |
| [01-overview.md](./01-overview.md) | 项目定位、目标与非目标 |
| [02-build-env.md](./02-build-env.md) | 目录结构、依赖 SDK、工程配置、链接库、遗留问题 |
| [03-module-scope.md](./03-module-scope.md) | 运行时模块清单与对标 Node.js 的范围 |
| [04-sqlite.md](./04-sqlite.md) | SQLite 源码集成方式与编译要点 |
| [99-open-questions.md](./99-open-questions.md) | 尚未确认、待用户补充的问题 |

## 记录状态

- [x] 项目定位与平台范围
- [x] QuickJS SDK 与工程配置约束
- [x] 运行时模块清单（第一版）
- [ ] JS 文件加载方式（用户稍后提供）
- [ ] 窗口创建与 WebView2 集成方式（用户稍后提供）
- [ ] 各模块 API 细节（逐个模块补充）
