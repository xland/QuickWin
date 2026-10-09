# 04 · SQLite 集成

## 方式：源码集成（amalgamation）

| 项 | 值 |
| --- | --- |
| 位置 | `Src\SQLite\` |
| 文件 | `sqlite3.c`（合并版单文件）、`sqlite3.h` |
| 版本 | **SQLite 3.53.4** |
| Source ID | `2026-07-24 19:02:57 bf7c7f30031888f4e796e429ab3978879485813aaca6f641c7b33e4e09459bcc` |
| License | Public Domain |

不是 DLL / 不是静态库预编译包，随工程一起编译。

## 工程接入情况（已配置）

`QuickWin.vcxproj`：

```xml
<ItemGroup>
  <ClCompile Include="Src\main.cpp" />
  <ClCompile Include="Src\SQLite\sqlite3.c" />
</ItemGroup>
<ItemGroup>
  <ClInclude Include="Src\SQLite\sqlite3.h" />
</ItemGroup>
```

`QuickWin.vcxproj.filters` 同步登记（`.c` 归"源文件"筛选器，`.h` 归"头文件"）。

## 编译要点

- `sqlite3.c` 按扩展名由 MSVC **以 C 方式编译**（未设 `CompileAs`），无需额外开关。
- 与全工程一致使用 **`/MTd`(Debug) / `/MT`(Release)** 静态 CRT；无外部依赖，不需要额外链接库。
- 当前沿用工程默认 `PreprocessorDefinitions`（`_DEBUG` / `NDEBUG`）+ `/W3` + SDL 检查。
- SQLite 官方推荐的 amalgamation 适用场景无需特殊 define；工程未设置任何 `SQLITE_*` 宏。

## 待定（等需求明确后再配）

| # | 事项 | 说明 |
| --- | --- | --- |
| S1 | 是否追加编译宏 | 如 `SQLITE_THREADSAFE`、`SQLITE_ENABLE_FTS5`、`SQLITE_ENABLE_RTREE`、`SQLITE_ENABLE_COLUMN_METADATA`、`SQLITE_USE_URI` 等，取决于 JS 侧要暴露哪些能力 |
| S2 | JSON 支持 | 3.53 起 JSON 函数（JSON1）默认随 amalgamation 启用，一般无需额外 define |
| S3 | 是否需要 `sqlite3ext.h` / shell CLI | 当前只有 `sqlite3.c` + `sqlite3.h`，若后续要做可加载扩展需补 `sqlite3ext.h` |
| S4 | JS 侧如何暴露 | 是否做一个 `sqlite` 模块、同步还是异步 API、语句如何处理 —— **用户尚未说明**，见 99 待确认表 |
