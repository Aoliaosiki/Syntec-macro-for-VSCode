# Syntec Macro — VS Code 扩展（0.1.0）

[![Version](https://img.shields.io/badge/version-0.1.0-blue)](CHANGELOG.md)
[![VS Code](https://img.shields.io/badge/VS%20Code-%5E1.85.0-007ACC)](package.json)
[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE)

为 **Syntec 数控系统宏程序**（无扩展名宏文件，如 `G0200`、`G0001`）提供语言支持，包括语法高亮、代码片段、文件自动关联、括号配对、代码折叠、自动缩进与专属颜色主题。

- G 码部分与原生 NC-Gcode 扩展逐 token 对齐。
- 宏程序部分单独高亮：`#`/`@`/`AR`/`MAR`、控制流、60 个内置函数、`%@MACRO` 文件头。
- 任意目录下的无扩展名文件自动识别，有后缀文件仍按扩展名优先。
- 19 个代码片段、括号配对、代码折叠、自动缩进、语言图标与专属主题。

> 当前版本：`0.1.0`。仓库：[`Aoliaosiki/Syntec-macro-for-VSCode`](https://github.com/Aoliaosiki/Syntec-macro-for-VSCode)。
> 最新安装包见 [Releases](https://github.com/Aoliaosiki/Syntec-macro-for-VSCode/releases)；旧版本发布后删除，只保留最新。
> 配套 PLC 梯形图 skill：[`Syntec-lad-skill-P`](https://github.com/Aoliaosiki/Syntec-lad-skill-P)，宏程序与 PLC 的 `R`/`@` 地址、握手、报警位需要一起核对。

> 📘 **开发者 / AI 维护者请看 [`README_TECH.md`](./README_TECH.md)** —— 包含全部原理、文件逐块解析、正则与 JSON 转义速查、修改规范 SOP、调试技巧与版本变更记录。**任何代码改动后必须在该文档 §16 追加记录。**

**核心设计：G 码部分严格对齐原生 [NC-Gcode 扩展](https://marketplace.visualstudio.com/items?itemName=ML.nc-gcode)（`ml.nc-gcode-0.13.0`），宏程序（MACRO）部分单独叠加高亮。**

> 语法支持范围以《MACRO开发应用手册》（Syntec 控制器宏程序开发手册）为准，本扩展的高亮规则与手册逐项核对。

## 目录

- [0.1.0 重点](#010-重点)
- [特性](#特性)
- [支持的语法元素](#支持的语法元素)
- [安装与使用](#安装与使用)
- [颜色主题](#颜色主题)
- [文件关联说明](#文件关联说明)
- [与 PLC skill 的配合](#与-plc-skill-的配合)
- [代码片段](#代码片段)
- [常见问题（FAQ）](#常见问题faq)
- [项目结构](#项目结构)
- [扩展配置文件速览](#扩展配置文件速览)
- [测试与验证](#测试与验证)
- [发布与版本](#发布与版本)
- [版本记录](#版本记录)
- [版权与许可](#版权与许可)

## 0.1.0 重点

### G 码对齐原生 NC-Gcode

语法文件中的 G/M/N/坐标/进给/转速/刀具/参数/循环字/注释规则，scope 命名与正则照搬 `ml.nc-gcode`（`ml.nc-gcode-0.13.0`）。扩展主题不覆盖这些 scope，颜色由当前 VS Code 主题按原生配色渲染。

### 宏程序高亮与 60 个内置函数

`#`/`@`/`AR`/`MAR` 变量（含动态数组）、`IF/FOR/WHILE/REPEAT/CASE` 控制流、运算符、`%@MACRO` 文件头，以及手册 Function List 全量 60 个内置函数。

```text
%@MACRO
#1 := 0
IF #1 < 10 THEN
  #1 := #1 + 1
END_IF
M30
```

### 无扩展名文件全量识别

`filenamePatterns` 使用 `**/*`：任意目录下没有后缀的文件（如 `G0200`、`O1234`）都会按 Syntec Macro 打开。`.mac`、`.pmc`、`.nc`、`.cnc` 等有后缀文件仍按扩展名优先识别。

> 需要限定范围时，可把 `**/*` 收窄为 `**/Macro/G*`、`**/NCFILES/O*` 等模式，但会失去“任意目录无后缀文件自动识别”。

---

## 特性

- ✅ **G 码高亮与原生 NC-Gcode 完全一致**：语法文件中的 G/M/N/坐标/F/S/T/P/Q/C/W/K/L/注释规则**原样照搬** `ml.nc-gcode` 的 scope 命名与正则，配色与原生扩展逐 token 一致，不额外自定义覆盖
- ✅ **宏程序高亮**：`#`/`@`/`MAR`/`AR` 变量（含动态数组）、控制流关键字、运算符、**手册函数表全量 60 个内置函数**、`%@MACRO` 文件头、`//` `(* *)` `( ... )` 注释
- ✅ **无扩展名文件自动识别**：`filenamePatterns` 为 `**/*`，任意目录下的无扩展名文件（如 `G0200`、`O1234`）均自动按 Syntec 语言打开；有后缀文件仍按扩展名优先识别
- ✅ **代码片段**：主程序、外扩 G 码、子程序、`if/ifelse/for/while/repeat/case/goto/exit`、变量赋值、`ar/mar`、`alarm/msg/print/sysdata` 等 19 个模板
- ✅ **括号配对、代码折叠**（`IF…END_IF`、`FOR…END_FOR`、`WHILE…END_WHILE`、`REPEAT…END_REPEAT`、`CASE…END_CASE`）、**自动缩进**
- ✅ **语言图标与显示名称**（Syntec Macro）

---

## 支持的语法元素

### G 码部分（scope 与原生 NC-Gcode 一致）

| 原生 scope | 匹配内容 |
| --- | --- |
| `keyword.control.gcode.gcode` | G 码：`G00` `G01` `G54.1` `G66.1` `G65` `G500`（含小数点子码，整体高亮） |
| `constant.gcode` | 行号：`N10` `N56` |
| `string.xcode.gcode` | X/A/U/I 坐标（同组同色） |
| `comment.type.ycode.gcode` | Y/B/V/J 坐标（同组同色） |
| `keyword.string.gcode` | Z/R 坐标（同组同色） |
| `support.function.mcode.gcode` | M 码：`M03` `M05` `M30` `M98` `M99` |
| `support.type.dDfFhHsStTcode.gcode` | F 进给、S 转速、T 刀具、D/H 补偿 |
| `variable.parameter.pqecode.gcode` | P/Q/E 参数 |
| `string.control.mcode.gcode` | C/W/K/L 循环字 |
| `comment.gcode` | `( ... )` 注释（Syntec 中 `(` 兼作数学运算括号，故 `X(...)` 中的运算括号不会被误判为注释） |

> 这些 scope **不**在扩展主题中覆盖，颜色由当前 VS Code 主题（如 Dark+）按原生配色渲染，因此与原生 NC-Gcode 扩展的显示完全一致。

### 宏程序部分（扩展自定义高亮）

| 类别 | 示例 |
| --- | --- |
| 文件头/宏指令 | `%@MACRO`、`@MACRO G0200`、`@ENDMACRO`、`@ENDPROGRAM`、`@END` |
| 子程序号 | `O1234` |
| 宏变量 | `#100`（本地）、`@1`（公共）、`AR1`（App 区域）、`MAR1`（App 永久储存）、`#[#1+1]`/`@[#1]`/`AR[#1]`（动态）、`#ABC`（字母变量） |
| 控制流 | `IF…ELSEIF…ELSE…END_IF`、`FOR 变量:=初值 TO 终值 BY 步长 DO…END_FOR`、`WHILE…END_WHILE`、`REPEAT…UNTIL…END_REPEAT`、`CASE…OF…END_CASE`、`GOTO`、`EXIT`、`RETURN`、`STOP` |
| 运算符 | 赋值 `:=`；比较 `=` `<>` `>` `<` `>=` `<=`；逻辑 `AND` `OR` `XOR` `NOT`；算术 `+ - * / MOD`；分号 `;` 语句结束符 |
| 常量 | `TRUE` `FALSE` `ON` `OFF` |
| 注释 | `// 行注释`、`(* 块注释 *)`、`( ... )`（与运算括号兼容） |
| 字符串 | `"..."` |

### 内置函数（手册函数表 Function List 全量 60 个）

| 分类 | 函数 |
| --- | --- |
| 数学 | `ABS` `ACOS` `ASIN` `ATAN` `ATAN2` `CEIL` `COS` `SIN` `TAN` `EXP` `FLOOR` `LN` `MAX` `MIN` `POW` `ROUND` `SIGN` `SQRT` `RANDOM` |
| 报警/提示 | `ALARM` `MSG` |
| 参数/引数 | `PARAM` `GETARG` `GETTRAPARG` |
| 字符串 | `SCANTEXT` `STR2INT` |
| 系统/驱动数据 | `SYSDATA` `DRVDATA` `SYSVAR` `AXID` |
| I/O 读写 | `READDI` `READDO` `READABIT` `READRREGBIT` `SETDO` `SETABIT` `SETRREGBIT` |
| 文件输出 | `OPEN` `PRINT` `CLOSE` |
| Cycle 档案(DB) | `DBOPEN` `DBLOAD` `DBSAVE` `DBNEW` `DBINSERT` `DBDELETE` |
| 栈操作 | `PUSH` `POP` `STKTOP` |
| 绘图 | `SETDRAW` `DRAWHOLE` |
| 机械校验 | `CHKMN` `CHKSN` `CHKMT` `CHKMI` `CHKINF` |
| 其他 | `WAIT` `SLEEP` `STD` `STDAX` |

> 函数高亮支持以手册函数表为准。实际可用性（版本号、参数规则等）请查阅《MACRO开发应用手册》各函数说明。

---

## 安装与使用

### 方式一：复制到已安装目录（Windows，本机调试，最常用）

VS Code 加载的是 **安装目录** 下的副本，**不是**本项目文件夹。修改源码后必须同步到安装目录，否则改动不生效（这是"改了没反应"的根因）。

```powershell
# 语法文件（本次更新 60 个函数后必须同步）
Copy-Item "syntaxes\syntec.tmLanguage.json" "$env:USERPROFILE\.vscode\extensions\syntec-macro-0.1.0\syntaxes\" -Force
# 主题
Copy-Item "themes\syntec-color-theme.json"  "$env:USERPROFILE\.vscode\extensions\syntec-macro-0.1.0\themes\" -Force
# 片段（若改过 snippets）
Copy-Item "snippets\syntec.code-snippets"   "$env:USERPROFILE\.vscode\extensions\syntec-macro-0.1.0\snippets\" -Force
```

同步后重启 VS Code，或 `Ctrl+Shift+P` → `Developer: Reload Window`。

> 安装目录固定为 `C:\Users\<你>\.vscode\extensions\syntec-macro-0.1.0\`（不同机器用户名不同）。

### 方式二：打包为 .vsix（分发）

```powershell
npm install -g @vscode/vsce   # 安装打包工具（只需一次）
vsce package                  # 在本目录打包，生成 syntec-macro-0.1.0.vsix
```

别人拿到 `.vsix` 后：扩展面板 → 右上角 `...` → **从 VSIX 安装**。

> 打包前注意：`vsce package` 会包含文件夹内所有文件（含 `MACRO开发应用手册*.html`、测试文件等）。若不需要可加 `.vscodeignore` 排除。

### 方式三：直接发文件夹（无需 Node.js）

把 `C:\Users\<你>\.vscode\extensions\syntec-macro-0.1.0\` 整个文件夹压缩为 zip 发给别人，解压到对方：
```
C:\Users\<别人>\.vscode\extensions\
```
重启 VS Code 即可。

> ⚠️ **发给别人后的配置**：建议切到 **Syntec Macro Color** 主题（`Ctrl+K` `Ctrl+T`）获得完整宏程序配色。不切主题时 G 码高亮（原生 scope）仍正常，但宏程序（`#`/`@` 变量、`IF` 等）颜色取决于对方当前主题。

### 安装目录检查

```powershell
Get-ChildItem "$env:USERPROFILE\.vscode\extensions" | Where-Object { $_.Name -like '*syntec*' }
```

确认是否存在 `syntec-macro-0.1.0` 文件夹。

---

## 颜色主题

### G 码部分

G 码颜色由**当前 VS Code 主题**（Dark+、Dark Modern 等）决定，**不需要启用扩展主题**，与原生 NC-Gcode 显示一致。

### 宏程序部分（Syntec Macro Color 主题）

按 `Ctrl+K` `Ctrl+T` 选择 **Syntec Macro Color** 后：

| 元素 | 颜色 |
| --- | --- |
| `%@MACRO`、`@MACRO`、`@ENDMACRO`、控制流关键字 | 品红 `#C586C0`（加粗） |
| `@MACRO G0200` 宏名、`O1234` 子程序号 | 黄 `#DCDCAA`（加粗） |
| `#` 局部变量、`@` 公共变量、`AR/MAR` | 蓝 `#9CDCFE`（斜体） |
| 逻辑/算术/比较运算符、分号 | 灰 `#D4D4D4` |
| 内置函数 | 黄 `#DCDCAA`（加粗） |
| 数字 | 浅绿 `#B5CEA8` |
| 常量 `TRUE/FALSE/ON/OFF` | 蓝 `#569CD6`（加粗） |
| 字符串 | 橙 `#CE9178` |
| 注释 `//` `(* *)` | 绿 `#6A9955`（斜体） |

> 各 G 码 scope 的颜色表见 `themes/syntec-color-theme.json` 中「G 码部分」注释段。

---

## 文件关联说明

无扩展名文件通过 `filenamePatterns` 自动关联：

```json
"filenamePatterns": [
  "**/*"
]
```

即任意目录下没有后缀的文件（如 `G0200`、`G0309`、`O1234`）均自动识别为 Syntec Macro。

> 这是 0.1.0 的最终规则。旧文档里的 `**/Macro/G*`、`**/NCFILES/O*` 等目录限定模式已不再使用；如确实只想识别特定目录，可在 `package.json` 中收窄，但会失去全量识别能力。

也支持有扩展名文件：`.mac`、`.pmc`、`.nc`、`.cnc`。

手动指定：状态栏右下角**语言模式** → 选择 **Syntec Macro**，或命令面板 `Change Language Mode`。

> ⚠️ 原生 NC-Gcode 也关联 `.nc`/`.cnc`。若打开 `.nc`/`.cnc` 显示的是 gcode 语法，手动切换语言模式为 **Syntec Macro** 即可。

---

## 与 PLC skill 的配合

宏程序负责 G/M/T 入口、参数读写、报警触发与握手；PLC 负责寄存器、互锁与执行。涉及 `R`/`@` 地址、握手时序、报警位时，应与配套的 PLC 梯形图 skill 一起核对：

- PLC skill：[`Aoliaosiki/Syntec-lad-skill-P`](https://github.com/Aoliaosiki/Syntec-lad-skill-P)
- 宏程序 master 与同功能 M/O 入口要同步修改，PLC 侧使用同一地址时必须一起确认。
- 宏里写 `R`、读 `@` 前，先确认 PLC 的读写时序与掉电保持要求，不要只改一侧。
- 报警号、报警文字与 PLC 报警位需要交叉核对，避免宏触发了未定义的报警。

本扩展只提供宏程序语言的编辑体验，不修改控制器、不上传 CNC，也不替代 PLC 编程与校验工具。

---

## 代码片段

在 Syntec 文件中输入前缀后按 `Tab` 或 `Ctrl+Space` 触发：

| 前缀 | 说明 |
| --- | --- |
| `macro` / `header` / `@macro` | 主程序模板（`%@MACRO` 开头，`M30` 结束） |
| `msub` / `gcode` / `extg` | 外扩 G 码模板（`M99` 返回） |
| `osub` / `m98` | 子程序模板（O 编号） |
| `if` | IF 条件判断 |
| `ifelse` | IF / ELSEIF / ELSE 多分支 |
| `for` / `loop` | FOR 循环 |
| `while` | WHILE 条件循环 |
| `repeat` | REPEAT / UNTIL 循环 |
| `case` | CASE 多分支选择 |
| `goto` | GOTO 跳转 |
| `exit` | EXIT 跳出循环 |
| `assign` / `set` | 变量赋值（`:=`） |
| `reg` / `rreg` | R 寄存器 / @ 变量操作 |
| `ar` / `mar` / `app` | AR / MAR App 变量 |
| `getarg` / `arg` | GETARG 读取引数 |
| `alarm` | ALARM 系统报警 |
| `msg` | MSG 提示信息 |
| `print` / `open` | OPEN / PRINT / CLOSE 文档输出 |
| `sysdata` / `sys` | SYSDATA / DRVDATA 读取系统数据 |

---

## 常见问题（FAQ）

### Q1：改了语法/片段文件，但 VSCode 里没反应？

**扩展加载的是安装目录的副本，不是项目文件夹。** 修改后必须执行「方式一」的 `Copy-Item` 同步到 `%USERPROFILE%\.vscode\extensions\syntec-macro-0.1.0\`，再 `Ctrl+Shift+P` → `Developer: Reload Window`。

### Q2：`READDI`、`SLEEP`、`SETDO` 等新函数不高亮？

- 确认已同步语法文件（见 Q1）；
- 确认扩展版本是 0.1.0 且 functions 规则已更新（含 60 个函数）；
- 确认当前打开的文件被识别为 Syntec Macro（状态栏语言模式）；
- 若安装了旧版 .vsix，先卸载再重装。

### Q3：状态栏出现「Completions - Completions limit reached」？

是 IntelliSense 提示：候选补全条目超过 `editor.suggest.maxShownSuggestions` 上限（默认 50~100），列表被截断，**不是错误**。若装了 GitHub Copilot 等 AI 扩展，候选数会暴增、更容易触发。可设置：

```json
"editor.suggest.maxShownSuggestions": 200
```

### Q4：状态栏出现「无内联建议可用 - 内联建议」？

是 Inline Suggestions（如 GitHub Copilot）的提示，表示当前光标位置无建议，**不是错误**。若想关闭：

```json
"editor.inlineSuggest.enabled": false,
"github.copilot.enable": { "*": false, "syntec": false }
```

### Q5：`( ... )` 会被当成注释吗？

Syntec 中 `(` 兼作数学运算括号。高亮规则用前置断言 `(?<![A-Za-z0-9])\(` 避免误判：`X(1.0+3.0)` 中的运算括号**不会**被当注释，而独立成行的 `( 注释 )` 会被当注释。

### Q6：`.nc`/`.cnc` 文件打开后是 gcode 语法？

原生 NC-Gcode 也关联这些后缀，冲突时以安装的扩展顺序为准。手动切语言模式为 **Syntec Macro** 即可。

### Q7：想要 G 码 + 宏程序完整的双色体系？

切到 **Syntec Macro Color** 主题（`Ctrl+K` `Ctrl+T`）。该主题同时为 G 码 scope（原生配色）与宏程序 scope（.syntec）着色。

---

## 项目结构

```
├── package.json                      # 扩展清单（语言/语法/片段/主题/文件关联）
├── language-configuration.json       # 注释/括号/折叠/缩进/单词规则
├── syntaxes/
│   └── syntec.tmLanguage.json        # TextMate 语法高亮（G 码部分照搬原生 NC-Gcode scope）
├── snippets/
│   └── syntec.code-snippets          # 19 个代码片段
├── themes/
│   └── syntec-color-theme.json       # 宏程序部分专属颜色主题
├── icons/
│   ├── syntec-light.svg              # 浅色主题图标
│   └── syntec-dark.svg               # 深色主题图标
├── LICENSE                           # MIT 许可证
├── CHANGELOG.md                      # 版本变更记录
└── README.md                         # 使用说明
```

---

## 扩展配置文件速览

### package.json（核心贡献点）

| 字段 | 说明 |
| --- | --- |
| `languages[].extensions` | 关联 `.mac` `.pmc` `.nc` `.cnc` |
| `languages[].filenames` | `G0200`（无扩展名） |
| `languages[].filenamePatterns` | `**/*`（匹配所有无后缀文件） |
| `grammars` | 语法文件路径 `./syntaxes/syntec.tmLanguage.json` |
| `snippets` | 片段文件路径 `./snippets/syntec.code-snippets` |
| `themes` | 主题文件路径 `./themes/syntec-color-theme.json` |

### language-configuration.json

- 注释：行 `//`，块 `(* ... *)`
- 折叠：`IF/FOR/WHILE/REPEAT/CASE` 起、`END_IF` 等止
- 缩进：`IF/FOR/WHILE/REPEAT/CASE/ELSEIF/ELSE` 增、`END_*`/`ELSE*` 减
- 单词规则：`#?@?(?:MAR|AR)?[A-Za-z0-9_]+\d*`

---

## 测试与验证

发布前至少完成以下检查，结果应全部通过：

| 检查项 | 方法 | 结果 |
| --- | --- | --- |
| `package.json` 可解析 | `Get-Content package.json -Raw \| ConvertFrom-Json` | ✅ |
| 语法文件可解析 | `Get-Content syntaxes\syntec.tmLanguage.json -Raw \| ConvertFrom-Json` | ✅ |
| 片段文件可解析 | `Get-Content snippets\syntec.code-snippets -Raw \| ConvertFrom-Json` | ✅ |
| 内置函数数量 | 统计语法文件 functions 正则中的函数名 | 60 个 |
| 代码片段数量 | 统计 `syntec.code-snippets` 顶层键 | 19 个 |
| VSIX 打包 | `npx @vscode/vsce package --out syntec-macro-0.1.0.vsix` | ✅ |
| 无扩展名文件识别 | `package.json` 的 `filenamePatterns` 为 `**/*`；再在 VS Code 打开 `G0200` 手工确认语言模式 | 配置正确，需手工确认 |

> 以上是发布前的最低检查，不替代在真实 VS Code 窗口里的手工验证。修改语法或片段后，必须同步安装目录并执行 `Developer: Reload Window`。

---

## 发布与版本

版本号唯一来源是 `package.json` 的 `version`；Git tag 必须与它一致（当前为 `v0.1.0`）。`.github/workflows/release.yml` 会在推送 `v*` tag 时校验版本、打包 VSIX 并创建 GitHub Release。

### 本地打包

```powershell
npx @vscode/vsce package --out syntec-macro-0.1.0.vsix
```

### 发布 GitHub Release

```powershell
git tag -a v0.1.0 -m "Syntec Macro 0.1.0"
git push origin v0.1.0
```

也可以手动上传已打包的 VSIX：

```powershell
gh release create v0.1.0 syntec-macro-0.1.0.vsix `
  --title "v0.1.0 — Syntec Macro" `
  --notes-file CHANGELOG.md
```

仓库只保留最新 Release。发布新版本后删除旧 Release 与旧 tag：

```powershell
gh release delete <old-tag> --yes --cleanup-tag
```

---

## 版本记录

| 版本 | 日期 | 说明 |
| --- | --- | --- |
| 0.1.0 | 2026-11-25 | 首个公开版；G 码对齐原生 NC-Gcode；宏程序高亮；按《MACRO开发应用手册》完整版核对函数表，内置函数扩展至 60 个（新增 `ATAN2` `AXID` `DBNEW` `DBSAVE` `DBINSERT` `DBDELETE` `SLEEP` `SIGN` `EXP` `READDI` `READDO` `READABIT` `READRREGBIT` `SETDO` `SETABIT` `SETRREGBIT` `SETDRAW` `DRAWHOLE` `PUSH` `POP` `STKTOP` `STD` `STDAX` `SYSVAR` `GETTRAPARG` `CHKINF`） |

> 仓库只保留最新 Release。发布新版本后会删除旧版本 Release 与对应 tag，避免用户下载到过期安装包。

---

## 版权与许可

**Copyright (c) 2026 Aoliaosiki**

本项目基于 [MIT License](./LICENSE) 发布。

你被允许自由使用、复制、修改、合并、发布、分发、再许可和/或销售本软件的副本，但须在所有副本或主要部分中保留上述版权声明与本许可声明。本软件按「原样」提供，不附带任何明示或暗示的担保。

### 第三方与参考资料声明

- G 码部分的 TextMate scope 命名与正则参考 [`ml.nc-gcode`](https://marketplace.visualstudio.com/items?itemName=ML.nc-gcode) 扩展的实现。
- 语法规则的核对依据为《MACRO开发应用手册》（Syntec 控制器厂商文档），该文档**不随本仓库或扩展包分发**，版权归原厂或原作者所有。
