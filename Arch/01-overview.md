# 01 · 项目定位

## 一句话定义

QuickWin 是一个 **QuickJS + WebView2 的 C++ 封装层**，对外提供一套**类 Node.js 的 JavaScript 运行时 API**，用于在 Windows 上用 JS 写桌面应用（WebView2 作为 UI 承载）。

## 目标

1. **封装 QuickJS**：以静态库形式接入 QuickJS（quickjs-ng），负责 Runtime/Context 生命周期、模块系统、宿主原生能力注入。
2. **封装 WebView2**：负责窗口创建、WebView2 控件集成、与 JS 的双向通信（细节用户稍后提供）。
3. **提供类 Node.js API**：让业务 JS 代码以接近 Node.js 的写法使用系统能力。

## 非目标（明确不做）

1. **不是 Node.js 的完整替代**：不追求 API 全覆盖，只实现少量核心模块。
2. **不实现 Node.js 的异步 I/O 模型**：不引入 libuv，事件循环由本项目自行设计（细节待定）。
3. **不兼容 Node.js 生态**：不提供 npm/CommonJS 包加载、不做 `node_modules` 解析（除非后续明确要求）。
4. **只兼容 Windows**：不做 macOS / Linux 兼容，不写跨平台抽象层，不为旧版 Windows 做降级。

## 平台与实现取向

- 目标平台：**仅 Windows**（不考虑 Windows 老版本兼容）。
- 原生能力实现优先级：**能用 Windows API 就用 Windows API**（典型：Crypto 用 Windows 的 BCrypt / CNG，而非移植 OpenSSL）。
- 允许使用 **WinRT API**（如需要 Windows.Storage、Windows.Security.Cryptography 等）。
- 不编写 `#ifdef _WIN32 / __APPLE__ / __linux__` 之类的跨平台分支。

## 命名

编译产物（可执行程序）名称待定，暂称 **QuickWin**（与工程/解决方案同名）。
