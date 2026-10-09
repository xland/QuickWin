# 03 · 运行时模块范围

## 原则

- 提供**少量** Node.js 模块，**尽量**对标 Node.js 的 API 形态（命名、参数、返回值、同步/异步命名惯例），
  但**不保证**完全兼容，也不追求覆盖 Node.js 全部方法。
- 全部基于 **Windows API / WinRT** 实现，不引入第三方跨平台库。

## 第一版模块清单

| 模块 | Node.js 对标 | 实现取向 | 状态 |
| --- | --- | --- | --- |
| `child_process` | `child_process` | Win32 `CreateProcess` + 匿名管道 | 待细化 |
| `console` | `console` | log/info/warn/error 等，输出目标待定（控制台 / 调试器 / WebView2） | 待细化 |
| `Crypto` | `crypto` | Windows BCrypt/CNG（不用 OpenSSL） | 待细化 |
| `Events` | `events` | `EventEmitter` | 待细化 |
| `File system` | `fs` | Win32 文件 API | 待细化 |
| `HTTP` | `http` | WinHTTP 或 WinRT `HttpClient`（待选型） | 待细化 |
| `Path` | `path` | 纯字符串处理，注意 Windows 路径语义 | 待细化 |
| `OS` | `os` | Win32 / `GetSystemInfo` 系列 | 待细化 |

## 待明确的共性问题（影响所有模块）

1. **模块引入方式**：`require('fs')`？ESM `import fs from 'fs'`？还是全局注入（如 `QuickWin.fs`）？
2. **同步 / 异步**：是否提供 `xxxSync` 系列？异步用回调、Promise，还是两者都提供？
3. **事件循环**：不引入 libuv，异步任务如何回到 JS 线程执行（宿主消息队列 + 任务队列？）。
4. **Buffer / 二进制**：是否提供 `Buffer` 类？编码（UTF-8 / UTF-16）转换策略？
5. **错误处理**：是否对标 Node 的 `Error` 的 `code` 字段（如 `ENOENT`）？

> 以上问题在 `99-open-questions.md` 同步留档，用户答复后回填到各模块文档。
