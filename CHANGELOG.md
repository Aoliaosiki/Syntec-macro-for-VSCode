# Change Log

## [0.1.0] - 2026-11-25

### Added

- Syntec 数控系统宏程序语法高亮（`source.syntec`）。
- 代码片段（snippets）：常用宏指令与流程控制模板。
- 文件关联：`.mac`、`.pmc`、`.nc`、`.cnc` 扩展名。
- 无扩展名文件全量识别：`filenamePatterns` 使用 `**/*`，任意目录下无后缀文件均自动识别为 Syntec Macro；有后缀文件仍按扩展名优先识别。
- 代码折叠、括号配对、自动缩进等语言配置。
- 可选颜色主题 `Syntec Macro Color`。
- 语言图标（light / dark）。

### Changed

- README 按 PLC skill 的文档风格补全：版本徽章、目录、0.1.0 重点、测试与验证、发布与版本。
- 修正 `filenamePatterns` 示例为实际的 `**/*`，并标注旧目录限定模式已废弃。
- 修正代码片段数量为 19 个。
- 增加与 `Syntec-lad-skill-P` 的配合说明：`R`/`@` 地址、握手时序与报警位需要跨宏/PLC 核对。
- 仓库只保留最新 Release；旧版本发布后删除。

### Verified

- `package.json`、`syntaxes/syntec.tmLanguage.json`、`snippets/syntec.code-snippets` 均可正常解析。
- 内置函数 60 个，代码片段 19 个。
- `vsce package` 打包通过，生成 `syntec-macro-0.1.0.vsix`。
