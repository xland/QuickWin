# 99 · 待确认问题

> 用户答复后，结论回填到对应文档，并在此处标记为已解决。

## A. 用户已声明稍后提供

| # | 问题 | 状态 |
| --- | --- | --- |
| A1 | 编译产物（exe）如何加载 js 文件（命令行参数？内置入口？热重载？） | ⏳ 待提供 |
| A2 | 如何创建窗口、如何集成 WebView2 组件 | ⏳ 待提供 |

## B. 实现细节待用户决策（见 03 模块文档的"共性问题"）

| # | 问题 | 状态 |
| --- | --- | --- |
| B1 | 模块引入方式：`require()` / ESM `import` / 全局对象注入 | ⏳ |
| B2 | 是否提供同步 + 异步两套 API，异步结果是 callback 还是 Promise | ⏳ |
| B3 | 不使用 libuv 的前提下，事件循环与异步任务回调机制如何设计 | ⏳ |
| B4 | 是否提供 `Buffer`，字符串/二进制编码转换策略 | ⏳ |
| B5 | 错误对象是否对标 Node 的 `err.code`（`ENOENT` 等） | ⏳ |
| B6 | 是否使用 QuickJS 自带的 `quickjs-libc`（`std` / `os` 模块）作为底座，还是全部自研 | ⏳ 倾向自研（便于 API 对齐 Node、可控） |
| B7 | HTTP 模块底层选型：WinHTTP 还是 WinRT `Windows.Web.Http.HttpClient` | ⏳ |
| B8 | WebView2 引入方式：NuGet 包 / 本地 SDK 目录；Evergreen 还是 Fixed Version | ✅ **已定**：NuGet packages.config + `Microsoft.Web.WebView2 1.0.4258.31`，loader 静态链接；Evergreen/Fixed 分发策略仍待定 |
| B9 | 工程是否拆分多个子工程 | ✅ **已定**：单工程，工程文件在仓库根，源码统一放 `Src/` |

## C. 工程配置遗留问题

见 [02-build-env.md](./02-build-env.md) §4 的 P1~P5。其中 **P1（NuGet targets 相对路径）会导致构建直接失败**，待用户确认后修复。
