# 02 · 依赖与环境

> 最近更新：用户将工程文件移到仓库根目录、源码迁到 `Src/`、接入 WebView2 NuGet 包并配置了一系列链接库。

## 0. 目录结构（当前实际）

```
d:\project\QuickWin\
├─ Arch\                      需求/架构文档（本目录）
├─ Doc\                       （空，待用）
├─ Src\                       源码根目录
│  └─ main.cpp
├─ packages\                  NuGet 还原目录（WebView2）
│  └─ Microsoft.Web.WebView2.1.0.4258.31\
├─ Build\                     构建输出目录（.gitignore 候选）
│  ├─ Debug\ / Release\       最终产物（exe/pdb）
│  └─ Temp\Debug\ / Temp\Release\  编译中间文件（obj 等）
├─ packages.config
├─ QuickWin.slnx
├─ QuickWin.vcxproj
├─ QuickWin.vcxproj.filters
└─ README.md
```

- **工程文件在仓库根目录**（`QuickWin.vcxproj`），不再位于 `QuickWin/` 子目录。
- **源码统一放 `Src/`**，vcxproj 中以 `Src\main.cpp` 形式引用；新增文件一律放 `Src/` 并在 vcxproj 登记。
- 解决方案 `QuickWin.slnx` 中项目路径已改为 `QuickWin.vcxproj`。

## 1. QuickJS（quickjs-ng）

| 项 | 值 |
| --- | --- |
| 源码目录 | `D:\sdk\quickjs` |
| 版本 | **0.17.0**（`quickjs.h`：`QJS_VERSION_MAJOR/MINOR/PATCH = 0/17/0`） |
| 来源 | quickjs-ng（Bellard 原版 QuickJS 的活跃 fork） |
| 头文件 | `quickjs.h`、`quickjs-libc.h`、`cutils.h` |
| Debug 静态库 | `D:\sdk\quickjs\build\Debug\qjs.lib` |
| Release 静态库 | `D:\sdk\quickjs\build\Release\qjs.lib` |

### 静态库编译方式（源自 `D:\sdk\quickjs\build\编译.txt`）

```
cmake -S . -B build -G "Ninja Multi-Config"
  CMAKE_C_COMPILER = C:/Program Files/LLVM/bin/clang.exe
  Debug:   -O0 -g -Xclang -gcodeview -D_DEBUG -D_MT -Xclang --dependent-lib=libcmtd
  Release: -O3 -DNDEBUG -D_MT -Xclang --dependent-lib=libcmt
```

推导出的**强制约束**：

1. 运行库必须静态链接 CRT：Debug `/MTd`（`MultiThreadedDebug`）、Release `/MT`（`MultiThreaded`）。
   否则 `libcmt` 与 `msvcrt` 冲突（LNK4098 / 符号重定义）。✅ 现有 x64 配置已满足。
2. 库为 **x64**（clang 默认 target `x86_64-pc-windows-msvc`，`CMAKE_GENERATOR_PLATFORM` 为空）。
   → 实际只应使用 x64 配置（vcxproj 仍保留 Win32 四个配置，见 §4 待办）。
3. 库由 clang 生成、遵循 MSVC ABI → 可被 v145 工具集直接链接。

### 当前 vcxproj 中的接入情况

- `AdditionalIncludeDirectories`：`D:\sdk\quickjs` ✅ 已配置（x64 Debug/Release）
- `AdditionalLibraryDirectories`：`D:\sdk\quickjs\build\$(Configuration)`
- `AdditionalDependencies` 首项：`qjs.lib`

→ 代码中可直接 `#include "quickjs.h"` / `#include <quickjs.h>`。

## 2. WebView2

| 项 | 值 |
| --- | --- |
| 引入方式 | **NuGet packages.config**（不是 PackageReference） |
| 包版本 | `Microsoft.Web.WebView2` **1.0.4258.31** |
| 还原目录 | `<repo>\packages\Microsoft.Web.WebView2.1.0.4258.31\` |
| Loader 链接方式 | `<WebView2LoaderPreference>Static</WebView2LoaderPreference>` |

包内容要点：

- 头文件：`build\native\include\`（Win32 C/C++ API：`WebView2.h`）、`build\native\include-winrt\`（C++/WinRT API）
- x64 库：`build\native\x64\WebView2LoaderStatic.lib`、`WebView2Loader.dll.lib`、`WebView2Loader.dll`
- `Microsoft.Web.WebView2.targets` → 转引 `build\Common.targets`，自动追加包含目录、库目录与 `WebView2Loader*`（Static 时为 `WebView2LoaderStatic.lib;version.lib`）

因此代码里可用 `#include <WebView2.h>`；静态链接 loader，运行时不必随行分发 `WebView2Loader.dll`。

## 3. 现有工程配置汇总（`QuickWin.vcxproj`）

| 项 | 值 |
| --- | --- |
| 解决方案 | `QuickWin.slnx`（工程路径 `QuickWin.vcxproj`，平台仅声明 `x64`） |
| 配置 | **仅 `Debug|x64` 与 `Release|x64`**（Win32 配置已删除） |
| 工具集 | `v145` |
| C++ 标准 | `stdcpp20` |
| 字符集 | Unicode |
| x64 Debug | `/MTd`、子系统 `Windows`、`PerMonitorHighDPIAware` |
| x64 Release | `/MT`、子系统 `Windows`、`PerMonitorHighDPIAware`、WPO |
| 输出目录 | `OutDir = $(ProjectDir)Build\$(Configuration)\` → `Build\Debug\` / `Build\Release\` |
| 中间目录 | `IntDir = $(ProjectDir)Build\Temp\$(Configuration)\` → `Build\Temp\Debug\` / `Build\Temp\Release\` |
| 链接系统库 | `qjs.lib;shlwapi.lib;dwmapi.lib;shell32.lib;usp10.lib;kernel32.lib;user32.lib;gdiplus.lib;windowsapp.lib;windowscodecs.lib;version.lib;urlmon.lib;ws2_32.lib;iphlpapi.lib;crypt32.lib;wtsapi32.lib;advapi32.lib` |

链接库清单已按后续模块需要铺开（网络 `ws2_32`/`iphlpapi`、加解密 `crypt32`、窗口合成 `dwmapi`、路径/Shell `shlwapi`/`shell32`、图像 `gdiplus`/`windowscodecs`、会话 `wtsapi32`、WinRT 伞库 `windowsapp.lib`）。

> 输出/中间目录写在**不带 Condition 的 PropertyGroup**（Label="输出目录"）里，对现有两个配置同时生效；用 `$(ProjectDir)` 而非 `$(SolutionDir)`，直接对工程单独构建（不经 slnx）时也成立。

## 4. 待办 / 风险

### 已修复（2026-10-09）

| # | 事项 | 处理 |
| --- | --- | --- |
| P1 | NuGet targets 相对路径错误 | 三处 `..\packages\...` → `packages\...`（ExtensionTargets Import、Error Condition、Error 文本参数） |
| P2 | 缺少 QuickJS 包含目录 | x64 Debug/Release 均加 `D:\sdk\quickjs` |
| P4 | Win32 配置与 x64 库不匹配 | 删除 `Debug|Win32` / `Release|Win32`（ProjectConfigurations、Configuration PropertyGroup、PropertySheets、ItemDefinitionGroup 四组）并同步 `.slnx` 去掉 `x86` 平台 |
| P3 | `windowsapp.lib` 与 `onecore.lib` 同时链接有冲突风险 | 已从两个 x64 配置的 `AdditionalDependencies` 中**移除 `onecore.lib`**，保留 `windowsapp.lib`（WinRT 伞库） |
| P5 | 旧输出目录残留 | 已删除 `Src\x64\`、`QuickWin\x64\`，并移除随之变空的 `QuickWin\` 目录 |

### 仍未处理

| # | 事项 | 说明 |
| --- | --- | --- |
| P6 | `Src\main.cpp` 目前为空文件 | 子系统为 `Windows`，无 `wWinMain`/`WinMain` 会 LNK1561；等实现阶段写真实入口，非配置问题 |
| P7 | 仓库无 `.gitignore` | 已有 GitHub 官方 `VisualStudio.gitignore`（忽略 `.vs/`、`*.user`、`packages/`、`*.obj`/`*.pdb`/`*.tlog` 等）；已追加 `[Bb]uild/` 覆盖本项目输出目录 |
