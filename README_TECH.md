# Syntec Macro VS Code 扩展 — 完整技术文档（AI 编译 / 后续维护专用）

> **本文档是项目的唯一权威技术说明书**。任何 AI / 开发者要对本项目做改动，必须先完整阅读本文档。
> **强制约定：本项目每次执行正确、被确认有效的代码变动后，必须在本文件末尾「版本变更记录」追加一条记录，并在对应章节同步更新细节。**

---

## 目录

1. [项目定位与核心设计](#1-项目定位与核心设计)
2. [文件结构总览](#2-文件结构总览)
3. [VSCode 扩展加载机制（关键！）](#3-vscode-扩展加载机制关键)
4. [TextMate 语法高亮原理](#4-textmate-语法高亮原理)
5. [语法文件逐块解析（syntec.tmLanguage.json）](#5-语法文件逐块解析)
6. [语言配置原理（language-configuration.json）](#6-语言配置原理)
7. [代码片段原理（syntec.code-snippets）](#7-代码片段原理)
8. [颜色主题原理（syntec-color-theme.json）](#8-颜色主题原理)
9. [文件关联原理（package.json）](#9-文件关联原理)
10. [图标原理（icons/）](#10-图标原理)
11. [构建 / 打包 / 安装 / 同步全流程](#11-构建--打包--安装--同步全流程)
12. [正则与 JSON 转义速查（踩坑重灾区）](#12-正则与-json-转义速查)
13. [修改规范（AI 必读 SOP）](#13-修改规范ai-必读-sop)
14. [调试技巧](#14-调试技巧)
15. [常见问题 FAQ](#15-常见问题-faq)
16. [版本变更记录（每次有效改动必须追加）](#16-版本变更记录)

---

## 1. 项目定位与核心设计

本项目是一个 VS Code 语言扩展，为 **Syntec（新代）数控系统宏程序**提供编辑支持，包含：

- 语法高亮（G 码 + MACRO 宏语法）
- 代码片段（snippets）
- 无扩展名文件自动识别（`G0200`、`O1234` 等）
- 代码折叠、括号配对、自动缩进
- 专属颜色主题（可选）

**核心设计原则（不可破坏）：**

> **G 码部分严格对齐原生 NC-Gcode 扩展（`ml.nc-gcode-0.13.0`），宏程序（MACRO）部分单独叠加高亮。**

即：`syntaxes/syntec.tmLanguage.json` 中 `gcode` 区块的正则与 scope 名称，是从 `ml.nc-gcode` 扩展**原样照搬**的（scope 名、正则一字不改），这样 G 码部分的配色与原生 NC-Gcode 显示逐 token 一致。

- G 码部分使用**原生 scope**（如 `keyword.control.gcode.gcode`），颜色由**当前 VS Code 主题**决定，扩展自带的 Syntec Macro Color 主题只是额外补了同一套配色。
- 宏程序部分使用**自定义 scope**（所有 scope 均带 `.syntec` 后缀，如 `keyword.control.syntec`），颜色由扩展自带主题或用户主题按 scope 渲染。

---

## 2. 文件结构总览

```
d:/APP/VScode/Microsoft VS Code/new/   ← 项目根（源码目录）
├── package.json                      ← 扩展清单（入口，最重要的配置）
├── language-configuration.json       ← 注释/括号/折叠/缩进/单词定义
├── README.md                         ← 用户使用文档（面向使用者）
├── README_TECH.md                    ← ★ 本文件，开发/维护技术文档（面向开发者/AI）
├── MACRO开发应用手册.html            ← 旧版参考手册（不完整）
├── MACRO开发应用手册_完整版.html     ← ★ 完整版参考手册（权威语法来源）
├── syntaxes/
│   └── syntec.tmLanguage.json        ← TextMate 语法高亮规则（核心）
├── snippets/
│   └── syntec.code-snippets          ← 代码片段（18 个）
├── themes/
│   └── syntec-color-theme.json       ← 专属颜色主题
└── icons/
    ├── syntec-light.svg              ← 浅色主题图标
    └── syntec-dark.svg               ← 深色主题图标
```

> ⚠️ **注意**：项目根目录还有一个隐藏目录 `.codebuddy/` 是 IDE 数据，**不要删除**。

---

## 3. VSCode 扩展加载机制（关键！）

### 3.1 源码目录 ≠ 加载目录

VS Code 通过 `package.json` 声明扩展内容，但**运行时加载的是安装目录下的副本**，不是项目源码目录。

**本机安装目录（Windows）：**
```
C:\Users\<用户名>\.vscode\extensions\syntec-macro-0.1.0\
```

**修改源码后必须把文件复制到安装目录，否则改动不生效。** 这就是历史问题"改了没反应"的根因。

### 3.2 同步清单

每次修改以下文件后，必须对应同步：

| 源码文件 | 安装目录目标 |
| --- | --- |
| `syntaxes/syntec.tmLanguage.json` | `…\syntec-macro-0.1.0\syntaxes\syntec.tmLanguage.json` |
| `themes/syntec-color-theme.json` | `…\syntec-macro-0.1.0\themes\syntec-color-theme.json` |
| `snippets/syntec.code-snippets` | `…\syntec-macro-0.1.0\snippets\syntec.code-snippets` |
| `package.json` | `…\syntec-macro-0.1.0\package.json` |

同步后需重启 VS Code，或执行 `Ctrl+Shift+P` → `Developer: Reload Window`。

### 3.3 扩展清单结构（package.json）

```jsonc
{
  "name": "syntec-macro",              // 扩展唯一标识（决定安装目录名）
  "displayName": "Syntec Macro",
  "version": "0.1.0",                  // 版本号（安装目录名会带上）
  "publisher": "local",                // 发布者
  "engines": { "vscode": "^1.85.0" },
  "categories": ["Programming Languages", "Snippets"],
  "contributes": {                     // ★ 扩展贡献点，全部在这里声明
    "languages": [...],                // 语言定义
    "grammars": [...],                 // 语法文件
    "snippets": [...],                 // 代码片段
    "themes": [...]                    // 颜色主题
  }
}
```

> 安装目录名规则：`<publisher>.<name>-<version>`，即 `local.syntec-macro-0.1.0` → 实际目录 `syntec-macro-0.1.0`（publisher 为 local 时省略）。

---

## 4. TextMate 语法高亮原理

VS Code 语法高亮使用 **TextMate 语法**（`.tmLanguage.json`）。核心概念：

### 4.1 scope（作用域）

- 每个 token 都有一个 scope 名（如 `keyword.control.syntec`）。
- scope 是**点分隔的层级结构**，主题通过 scope 前缀匹配上色。
- 本项目宏程序部分所有 scope 都带 `.syntec` 后缀，避免与其它语言冲突。

### 4.2 语法文件顶层结构

```jsonc
{
  "$schema": "https://raw.githubusercontent.com/martinring/tmlanguage/master/tmlanguage.json",
  "name": "Syntec Macro",              // 语言显示名
  "scopeName": "source.syntec",        // ★ 顶层 scope（语言根）
  "fileTypes": ["mac", "pmc", "nc", "cnc"],  // 默认文件扩展（VSCode 中实际由 package.json 决定）
  "patterns": [ ... ],                 // ★ 顶层匹配规则（按顺序尝试）
  "repository": { ... }                // ★ 规则库，可被 include 引用
}
```

### 4.3 patterns 与 repository

- `patterns` 是规则列表，**从上到下依次尝试**（实际是"所有规则在同一位置竞争，取最先匹配且最靠前的"）。
- `repository` 是命名规则库，用 `{ "include": "#规则名" }` 引用。
- 顶层 `patterns` 通过 `include` 引入 repository 中的规则，例如：

```jsonc
"patterns": [
  { "include": "#strings" },      // 字符串（先匹配，避免括号等误判）
  { "include": "#comments" },     // 注释
  { "include": "#variables" },    // 变量
  { "include": "#keywords" },     // 关键字
  { "include": "#functions" },    // 函数
  { "include": "#gcode" },        // G 码
  ...
]
```

### 4.4 三种规则形态

**① match（单行匹配）：**

```jsonc
{
  "match": "\\b(?:IF|THEN|ELSE)\\b",
  "name": "keyword.control.syntec"
}
```

**② begin/end（块匹配，跨行）：** 用于 `(* ... *)`、`( ... )`、字符串等有边界的结构。

```jsonc
{
  "begin": "\\(\\*",              // 开始标记 (*
  "end": "\\*\\)",                // 结束标记 *)
  "name": "comment.block.syntec",
  "beginCaptures": { "0": { "name": "punctuation.definition.comment.begin.syntec" } },
  "endCaptures":   { "0": { "name": "punctuation.definition.comment.end.syntec" } },
  "patterns": [ ... ]             // 块内的子规则（可选）
}
```

**③ captures（捕获分组）：** 用 `( )` 分组，给不同分组不同 scope。

```jsonc
{
  "match": "^\\s*(@MACRO)\\b(?:\\s+([Gg][0-9]{2,4}))?",
  "captures": {
    "1": { "name": "keyword.control.syntec" },          // @MACRO
    "2": { "name": "entity.name.function.syntec" }      // G9000 宏名
  }
}
```

### 4.5 匹配优先级（重点！）

TextMate 规则按**位置与顺序**竞争：

1. 从当前扫描位置开始，所有可匹配的规则都在**同一位置**尝试；
2. **第一个能匹配上的规则胜出**（列表顺序优先）；
3. 若规则匹配位置不同，则**最靠前的匹配**胜出。

因此：**把更具体的规则放在前面**，把宽泛的规则放后面。例如：
- `variables` 中先匹配数组 `MAR[..]`、`@[..]`，再匹配简单变量 `MAR1`、`@10`；
- `keywords` 中先匹配长词 `END_IF`，再匹配可能冲突的短词；
- 顶层 `patterns` 中把 `strings`、`comments` 放前面，避免 `(` 被注释规则吞掉。

### 4.6 本项目的 scope 清单（宏程序部分）

| scope | 含义 | 颜色主题中 |
| --- | --- | --- |
| `keyword.control.syntec` | 控制流/文件头/宏指令 | 品红加粗 |
| `entity.name.function.syntec` | 宏名、子程序号 | 黄加粗 |
| `variable.other.syntec` | 变量 | 蓝斜体 |
| `keyword.operator.syntec` | 运算符 | 灰 |
| `punctuation.definition.statement.syntec` | 分号 | 灰 |
| `support.function.syntec` | 内置函数 | 黄加粗 |
| `constant.numeric.syntec` | 数字 | 浅绿 |
| `constant.language.syntec` | 常量 TRUE/FALSE/ON/OFF | 蓝加粗 |
| `string.quoted.double.syntec` | 字符串 | 橙 |
| `comment.line.double-slash.syntec` | `//` 注释 | 绿斜体 |
| `comment.block.syntec` | `(* *)` 和 `( )` 注释 | 绿斜体 |
| `constant.character.escape.syntec` | 注释内转义 | 橙 |
| `punctuation.definition.*.syntec` | 注释/字符串边界符 | 橙 |

---

## 5. 语法文件逐块解析

文件：`syntaxes/syntec.tmLanguage.json`

### 5.1 顶层 patterns（匹配顺序）

```jsonc
"patterns": [
  { "include": "#strings" },      // ① 字符串 "..."（最先，避免括号吞字符串）
  { "include": "#comments" },     // ② 注释 // (* *) ( )
  { "include": "#fileHeader" },   // ③ %@MACRO 文件头
  { "include": "#macroDirectives" }, // ④ @ 开头宏指令
  { "include": "#subProgram" },   // ⑤ O1234 子程序号
  { "include": "#variables" },    // ⑥ 变量 # @ MAR AR
  { "include": "#keywords" },     // ⑦ 控制流关键字
  { "include": "#functions" },    // ⑧ 内置函数
  { "include": "#gcode" },        // ⑨ G 码（照搬原生）
  { "include": "#operators" },    // ⑩ 运算符
  { "include": "#numbers" }       // ⑪ 数字
]
```

### 5.2 repository 各规则说明

| 规则名 | 匹配内容 | 正则（JSON 转义后实际含义） |
| --- | --- | --- |
| `comments` | `(* ... *)` 块注释 | begin `\( \*` end `\* \)` |
| | `//...` 行注释 | `//.*$` |
| | `( ... )` Syntec 注释 | begin `(?<![A-Za-z0-9])\(` end `\)`（**前置断言避免误吞 `X(...)` 运算括号**） |
| `strings` | `"..."` 字符串 | begin `"` end `"` |
| `fileHeader` | `%@MACRO` 文件头 | `^\s*%@\s*MACRO` |
| `macroDirectives` | `@ENDMACRO/@ENDPROGRAM/@END` | `^\s*@(ENDMACRO\|ENDPROGRAM\|END)(?=\s\|$)` |
| | `@MACRO G9000` | `^\s*(@MACRO)\b(?:\s+([Gg][0-9]{2,4}))?` |
| | 其他 `@指令` | `^\s*@[A-Za-z][A-Za-z0-9_]*` |
| `subProgram` | `O1234` | `^\s*[Oo][0-9]{1,5}\b` |
| `variables` | `MAR[..]`/`AR[..]` 数组 | `\b(?:MAR\|AR)\[[^\]]*\]` |
| | `@[..]` 数组 | `@\[[^\]]*\]` |
| | `#[..]` 数组 | `#\[[^\]]*\]` |
| | `MAR1`/`AR1` | `\b(?:MAR\|AR)\d+` |
| | `@10` | `@\d+` |
| | `#100` | `#\d+` |
| | `#ABC` 字母变量 | `#[A-Za-z][A-Za-z0-9_]*` |
| `keywords` | 控制流 | `\b(?:IF\|THEN\|ELSEIF\|ELSE\|END_IF\|FOR\|TO\|BY\|DO\|END_FOR\|WHILE\|END_WHILE\|REPEAT\|UNTIL\|END_REPEAT\|CASE\|OF\|END_CASE\|GOTO\|EXIT\|RETURN\|STOP)\b` |
| | 紧跟数字的 THEN/ELSE/DO/OF | `(?<=\d)(?:THEN\|ELSE\|DO\|OF)\b` |
| | 紧跟数字的 TO/BY | `\b(?:TO\|BY)(?=\d)` |
| | 逻辑运算符 | `\b(?:AND\|OR\|XOR\|NOT\|MOD)\b` |
| | 常量 | `\b(?:TRUE\|FALSE\|ON\|OFF)\b` |
| `functions` | 60 个内置函数（详见下方） | 见 5.3 |
| `gcode` | G/N/M/坐标/F/S/T/P/Q/E/C/W/K/L | 见 5.4（照搬原生） |
| `operators` | 运算符 | `:=` `<>` `<=` `>=` `=` `<` `>` `+` `-` `*` `/` |
| | 分号 | `;` |
| `numbers` | 数字 | `(?<![0-9.])[0-9]+(?:\.[0-9]*)?(?:[Ee][+-]?[0-9]+)?(?![0-9])` |

### 5.3 functions 规则（核心，60 个函数）

```jsonc
{
  "comment": "内置函数（依据手册函数表 Function List，覆盖 ABS~DRVDATA 共 60 个）",
  "match": "\\b(?:READRREGBIT|SETRREGBIT|GETTRAPARG|DBDELETE|DBINSERT|DBLOAD|DBNEW|DBOPEN|DBSAVE|DRAWHOLE|DRVDATA|SYSDATA|STR2INT|SCANTEXT|SYSVAR|SETDRAW|STKTOP|STDAX|READDI|READDO|READABIT|RANDOM|SETDO|SETABIT|SIGN|SLEEP|PUSH|PRINT|PARAM|GETARG|FLOOR|CEIL|CLOSE|ALARM|ROUND|POW|POP|OPEN|MSG|SQRT|MAX|MIN|LN|EXP|ABS|ACOS|ASIN|ATAN|ATAN2|AXID|COS|TAN|SIN|WAIT|CHKINF|CHKMN|CHKSN|CHKMT|CHKMI|STD)\\s*(?=\\()",
  "name": "support.function.syntec"
}
```

**设计要点：**
- `\s*(?=\()` 前瞻：只有**后面跟括号**的函数名才被识别（`FUNC(...)` 形式），避免误匹配普通单词。
- **长函数名优先**（`READRREGBIT` 在 `READDI` 前，`DBDELETE` 在 `DBLOAD` 前）：防止前缀子串抢先匹配。
- 60 个函数清单（依据《MACRO开发应用手册_完整版.html》函数表）：

```
ABS ACOS ALARM ASIN ATAN ATAN2 AXID CEIL COS CLOSE
DBOPEN DBLOAD DBSAVE DBNEW DBINSERT DBDELETE DRAWHOLE DRVDATA EXP FLOOR
GETARG GETTRAPARG LN MAX MIN MSG OPEN PARAM POP POW
PRINT PUSH RANDOM READDI READDO READABIT READRREGBIT ROUND SCANTEXT SETDO
SETABIT SETRREGBIT SETDRAW SIGN SIN SLEEP SQRT STD STDAX
STKTOP SYSVAR TAN WAIT CHKMN CHKSN CHKMT CHKMI CHKINF
STR2INT SYSDATA
```

### 5.4 gcode 规则（照搬原生 NC-Gcode）

> ⚠️ **这些规则从 `ml.nc-gcode-0.13.0/syntaxes/gcode.tmLanguage.json` 原样照搬**，scope 名、正则一字不改，保证 G 码配色与原生扩展完全一致。**不要改动这些正则和 scope 名。**

| scope | 匹配 |
| --- | --- |
| `constant.gcode` | `N10` 行号 |
| `keyword.control.gcode.gcode` | `G00` `G54.1` 等 G 码 |
| `keyword.string.gcode` | `R/Z` 坐标 |
| `string.xcode.gcode` | `X/A/U/I` 坐标 |
| `comment.type.ycode.gcode` | `Y/B/V/J` 坐标 |
| `support.function.mcode.gcode` | `M03` `M30` 等 M 码 |
| `support.type.dDfFhHsStTcode.gcode` | `F/S/T/D/H` |
| `variable.parameter.pqecode.gcode` | `P/Q/E` 参数 |
| `string.control.mcode.gcode` | `C/W/K/L` 循环字 |

### 5.5 关键边界情况处理

1. **`X(...)` 运算括号 vs `( ... )` 注释**：Syntec 中 `(` 既作注释又作运算括号。规则用 `(?<![A-Za-z0-9])\(` 前置断言，即 `(` 前面不是字母/数字才当作注释。所以 `X(1.0+3.0)` 的括号**不会被**误判为注释。
2. **`#1=1` 无空格赋值**：`keywords` 中 `(?<=\d)(?:THEN|ELSE|DO|OF)\b` 处理 `IF#1=2THEN` 这种紧凑写法。
3. **`TO 5` / `BY 1`**：`\b(?:TO|BY)(?=\d)` 只匹配后面紧跟数字的 TO/BY（FOR 循环的步进），避免误匹配普通单词。
4. **注释优先级**：`strings` 和 `comments` 放最前，避免 `"..."` 内的 `#100` 被变量规则抢匹配（实际在字符串块内子规则会覆盖）。

---

## 6. 语言配置原理

文件：`language-configuration.json`（对应 package.json 中 `languages[].configuration`）

```jsonc
{
  "comments": {                              // 注释
    "lineComment": "//",                     // 行注释
    "blockComment": ["(*", "*)"]             // 块注释
  },
  "brackets": [["[", "]"]],                  // 括号配对
  "autoClosingPairs": [                      // 自动闭合
    { "open": "[", "close": "]" },
    { "open": "\"", "close": "\"", "notIn": ["string", "comment"] }
  ],
  "surroundingPairs": [["[", "]"], ["\"", "\""]],
  "folding": {                               // ★ 代码折叠
    "markers": {
      "start": "^\\s*(IF\\b.*|FOR\\b.*|WHILE\\b.*|REPEAT\\b.*|CASE\\b.*)",
      "end": "^\\s*(END_IF|END_FOR|END_WHILE|END_REPEAT|END_CASE)"
    }
  },
  "indentationRules": {                      // 自动缩进
    "increaseIndentPattern": "^\\s*(IF\\b.*|FOR\\b.*|WHILE\\b.*|REPEAT\\b.*|CASE\\b.*|ELSEIF\\b.*|ELSE\\b.*).*$",
    "decreaseIndentPattern": "^\\s*(END_IF|END_FOR|END_WHILE|END_REPEAT|END_CASE|ELSEIF\\b.*|ELSE\\b.*).*$"
  },
  "wordPattern": "#?@?(?:MAR|AR)?[A-Za-z0-9_]+\\d*"  // 双击选词规则
}
```

> **注意**：这里的正则也是 JSON 转义后的（`\\s` 实际是 `\s`）。折叠/缩进 marker 使用 Oniguruma 正则。

---

## 7. 代码片段原理

文件：`snippets/syntec.code-snippets`（对应 package.json 中 `snippets[].path`）

**结构：** JSON 对象，每个 key 是片段名，value 包含 `prefix`（触发前缀）、`body`（内容数组）、`description`。

```jsonc
{
  "If Then": {
    "prefix": ["if"],
    "body": [
      "IF (${1:条件}) THEN",
      "  ${2:// 陈述列表}",
      "END_IF;"
    ],
    "description": "IF 条件判断（条件用 ( ) 括起）"
  }
}
```

**Tab 停靠点语法：**
- `${1:默认值}` — 第 1 个停靠点，带默认文本
- `${2:...}` — 第 2 个停靠点，按 Tab 依次跳转
- `$1` — 引用第 1 个停靠点的值（同步复制）
- 多个片段可引用同一停靠点编号实现同步编辑

**现有 18 个片段清单：**

| 片段名 | prefix | 用途 |
| --- | --- | --- |
| Macro Main Program | `macro` `header` `@macro` | 主程序模板 |
| Macro External G-code | `msub` `gcode` `extg` | 外扩 G 码 |
| Macro Sub O-code | `osub` `m98` | 子程序 |
| If Then | `if` | IF 判断 |
| If ElseIf Else | `ifelse` | 多分支 |
| For Loop | `for` `loop` | FOR 循环 |
| While Loop | `while` | WHILE 循环 |
| Repeat Until | `repeat` | REPEAT/UNTIL |
| Case Select | `case` | CASE 分支 |
| Goto Label | `goto` | GOTO 跳转 |
| Exit Loop | `exit` | EXIT 跳出 |
| Variable Assign | `assign` `set` | 变量赋值 |
| Register Variable | `reg` `rreg` | R 寄存器 |
| App Variable | `ar` `mar` `app` | AR/MAR 变量 |
| Get Argument | `getarg` `arg` | GETARG |
| Alarm Msg | `alarm` | ALARM 报警 |
| Msg Prompt | `msg` | MSG 提示 |
| Print Output | `print` `open` | OPEN/PRINT/CLOSE |
| System Data | `sysdata` `sys` | SYSDATA/DRVDATA |

---

## 8. 颜色主题原理

文件：`themes/syntec-color-theme.json`（对应 package.json 中 `themes[].path`）

### 8.1 结构

```jsonc
{
  "name": "Syntec Macro Color",
  "type": "dark",                     // 主题类型 dark/light
  "colors": { ... },                  // UI 颜色（编辑器背景、状态栏等）
  "tokenColors": [ ... ]              // ★ 语法 token 配色
}
```

### 8.2 tokenColors（按 scope 匹配）

```jsonc
"tokenColors": [
  {
    "comment": "============= G 码部分 =============",
    "scope": "constant.gcode",
    "settings": { "foreground": "#9CDCFE" }
  },
  {
    "scope": "keyword.control.syntec",
    "settings": { "foreground": "#C586C0", "fontStyle": "bold" }
  },
  ...
]
```

- `scope` 可以是字符串或数组（多个 scope 同色）。
- `settings.foreground` 是十六进制颜色，`settings.fontStyle` 可选 `bold` / `italic` / `underline`。
- **scope 匹配规则**：`keyword.control.syntec` 会匹配精确 scope；`keyword.control`（不带后缀）会匹配所有以 `keyword.control` 开头的 token。

### 8.3 当前配色表

| 元素 | scope | 颜色 |
| --- | --- | --- |
| G 码 | `keyword.control.gcode.gcode` | `#569CD6` 加粗 |
| 行号 N | `constant.gcode` | `#9CDCFE` |
| X/A/U/I 坐标 | `string.xcode.gcode` | `#4EC9B0` |
| Y/B/V/J 坐标 | `comment.type.ycode.gcode` | `#DCDCAA` |
| Z/R 坐标 | `keyword.string.gcode` | `#CE9178` |
| M 码 | `support.function.mcode.gcode` | `#C586C0` 加粗 |
| F/S/T/D/H | `support.type.dDfFhHsStTcode.gcode` | `#4EC9B0` |
| P/Q/E | `variable.parameter.pqecode.gcode` | `#569CD6` |
| C/W/K/L | `string.control.mcode.gcode` | `#569CD6` |
| 宏指令/控制流 | `keyword.control.syntec` | `#C586C0` 加粗 |
| 宏名/子程序号 | `entity.name.function.syntec` | `#DCDCAA` 加粗 |
| 变量 | `variable.other.syntec` | `#9CDCFE` 斜体 |
| 运算符/分号 | `keyword.operator.syntec` / `punctuation.definition.statement.syntec` | `#D4D4D4` |
| 内置函数 | `support.function.syntec` | `#DCDCAA` 加粗 |
| 数字 | `constant.numeric.syntec` | `#B5CEA8` |
| 常量 | `constant.language.syntec` | `#569CD6` 加粗 |
| 字符串 | `string.quoted.double.syntec` | `#CE9178` |
| 注释 | `comment.line.double-slash.syntec` / `comment.block.syntec` | `#6A9955` 斜体 |
| 注释/字符串边界符 | `punctuation.definition.*.syntec` 等 | `#CE9178` |

---

## 9. 文件关联原理

文件：`package.json` → `contributes.languages`

### 9.1 三种关联方式（优先级从高到低）

```jsonc
"languages": [
  {
    "id": "syntec",
    "aliases": ["Syntec Macro", "Syntec", "数控宏程序"],
    "extensions": [".mac", ".pmc", ".nc", ".cnc"],   // ① 扩展名
    "filenames": ["G0200"],                            // ② 精确文件名
    "filenamePatterns": [                              // ③ glob 路径模式
      "**/*"
    ],
    "configuration": "./language-configuration.json",
    "icon": { "light": "./icons/syntec-light.svg", "dark": "./icons/syntec-dark.svg" }
  }
]
```

### 9.2 关联规则说明

- `**/*`：匹配**所有无扩展名文件**（任意目录、任意文件名，如 `G0200`、`O1234`、`9001` 等），不再局限于 `Macro/`、`NCFILES/` 目录或 `G`/`O` 开头
- 原理：VS Code 中 `extensions`（含内置 `.txt`→plaintext 等）优先级**高于** `filenamePatterns`，因此 `**/*` 只会命中「无任何扩展名关联」的文件，`.mac`/`.nc`/`.txt` 等有后缀文件仍按其扩展名识别
- `filenames` 中的 `G0200` 为历史精确匹配保留（已被 `**/*` 覆盖，可省略）
- 冲突处理：`.nc`/`.cnc` 也被原生 NC-Gcode 关联，冲突时以安装顺序为准，用户可手动切换语言模式

---

## 10. 图标原理

- 文件：`icons/syntec-light.svg`（浅色主题）、`icons/syntec-dark.svg`（深色主题）
- 用途：资源管理器中 .mac/.pmc/.nc/.cnc 文件图标、语言模式图标
- 要求：SVG 格式，符合 VSCode 图标规范（16x16 视觉网格）

---

## 11. 构建 / 打包 / 安装 / 同步全流程

### 11.1 本机调试（最快）

```powershell
# 1. 修改源码文件
# 2. 同步到安装目录
$ext = "$env:USERPROFILE\.vscode\extensions\syntec-macro-0.1.0"
Copy-Item "syntaxes\syntec.tmLanguage.json" "$ext\syntaxes\" -Force
Copy-Item "themes\syntec-color-theme.json"  "$ext\themes\" -Force
Copy-Item "snippets\syntec.code-snippets"   "$ext\snippets\" -Force
# 3. 重载窗口
# Ctrl+Shift+P → Developer: Reload Window
```

### 11.2 打包 .vsix（分发）

```powershell
npm install -g @vscode/vsce    # 首次
vsce package                   # 生成 syntec-macro-0.1.0.vsix
```

接收方：扩展面板 → `...` → 从 VSIX 安装。

> 打包注意：`vsce package` 会包含目录内所有文件。建议在 `.vscodeignore` 中排除 `MACRO*.html`、`README_TECH.md` 等非必要文件。

### 11.3 直接发文件夹

压缩 `C:\Users\<你>\.vscode\extensions\syntec-macro-0.1.0\` 为 zip，接收方解压到自己的 `%USERPROFILE%\.vscode\extensions\`，重启 VS Code。

### 11.4 检查安装目录

```powershell
Get-ChildItem "$env:USERPROFILE\.vscode\extensions" | Where-Object { $_.Name -like '*syntec*' }
```

---

## 12. 正则与 JSON 转义速查（踩坑重灾区）

### 12.1 JSON 转义（最重要！）

tmLanguage.json、language-configuration.json 中的正则写在 JSON 字符串里，**所有反斜杠要写双份**：

| 实际正则 | JSON 中写法 |
| --- | --- |
| `\b`（词边界） | `\\b` |
| `\s`（空白） | `\\s` |
| `\(`（字面左括号） | `\\(` |
| `\)`（字面右括号） | `\\)` |
| `\.`（字面点） | `\\.` |
| `\d`（数字） | `\\d` |
| `\*`（字面星号） | `\\*` |

> ⚠️ 若写成单反斜杠（如 `\b`），JSON 解析会报错或变成退格符。**这是新增/修改规则时最常见的错误。**

### 12.2 Oniguruma 正则特性（VSCode 使用）

VS Code 使用 **Oniguruma** 正则引擎，支持（但注意与 JS 正则的差异）：

- ✅ 正向后顾 `(?<=...)`、负向后顾 `(?<!...)`
- ✅ 正向前瞻 `(?=...)`、负向前瞻 `(?!...)`
- ✅ 非捕获组 `(?:...)`
- ✅ 词边界 `\b`
- ✅ 字符类 `[^...]`
- ✅ 嵌套量词、交替 `|`
- ❌ 不支持 JS 的某些特性（如具名组引用等，本项目未用到）

### 12.3 常用模式速查

```jsonc
// 词边界 + 非捕获组 + 交替
"\\b(?:IF|THEN|ELSE)\\b"

// 前瞻：函数名后必须跟 (
"\\b(?:SIN|COS)\\s*(?=\\()"

// 后顾：避免 X(...) 被当注释
"(?<![A-Za-z0-9])\\("

// 行首 + 空白
"^\\s*@(ENDMACRO|ENDPROGRAM)(?=\\s|$)"

// 字符类排除
"\\[[^\\]]*\\]"     // 匹配 [ ... ] 不含 ]
```

---

## 13. 修改规范（AI 必读 SOP）

### 13.1 新增内置函数

1. 先在《MACRO开发应用手册_完整版.html》函数表确认函数存在；
2. 打开 `syntaxes/syntec.tmLanguage.json` 的 `functions` 规则；
3. **按长度降序**插入到 `(?:...)` 非捕获组中（长词在前，防止前缀冲突）；
4. 遵循 JSON 转义（`\\b`、`\\s`、`\\(`）；
5. 同步到安装目录 + Reload Window；
6. 在 `README_TECH.md` §16 追加变更记录。

### 13.2 新增/修改关键字

- 控制流关键字 → `keywords` 的 `keyword.control.syntec` 规则；
- 逻辑运算符 → `keywords` 的 `keyword.operator.syntec` 规则；
- 常量 → `keywords` 的 `constant.language.syntec` 规则；
- 注意 `\b` 边界，避免误匹配普通单词。

### 13.3 新增代码片段

1. 在 `snippets/syntec.code-snippets` 添加新 key；
2. `prefix` 用数组（可多个触发词）；
3. `body` 用数组（每行一个元素）；
4. 使用 `${1:默认值}` 做 Tab 停靠点，`$1` 引用同步；
5. 同步安装目录 + 重载。

### 13.4 修改颜色

1. 打开 `themes/syntec-color-theme.json`；
2. 找到对应 scope 的 `tokenColors` 条目；
3. 修改 `foreground`（#RRGGBB）或 `fontStyle`；
4. 同步安装目录 + 重载。

### 13.5 修改后必须做的三件事（强制）

```text
① 同步文件到安装目录
② Reload Window
③ 在 README_TECH.md 版本变更记录追加条目
```

---

## 14. 调试技巧

### 14.1 检查 token scope（最重要）

`Ctrl+Shift+P` → `Developer: Inspect Editor Tokens and Scopes`，然后点击代码任意位置，可看到该 token 的 scope、所属规则、主题颜色来源。

**用途：**
- 确认某个 token 是否被识别、scope 是否正确；
- 确认颜色是来自语法规则还是主题；
- 排查"没高亮"问题（先看 scope，再看主题）。

### 14.2 判断是否加载了新语法

- 修改后若某 token 行为未变化，先确认**已同步安装目录**并 **Reload Window**；
- 用 Inspector 看 scope 名是否与预期一致。

### 14.3 正则测试

在 PowerShell 中测试（注意 PowerShell 单引号字符串不转义）：

```powershell
$pat = '\\b(?:SIN|COS)\\s*(?=\\()'   # 注意：从 JSON 读出来的才是对的正则
[regex]::Matches('G01 X(1.0) COS(30);', $pat)
```

> ⚠️ 最准确的方法是直接从 `syntec.tmLanguage.json` 读取 match 字段测试，避免手写转义错误。

### 14.4 查看语法错误

`Help` → `Toggle Developer Tools`（F12），Console 面板可看到 JSON 解析错误。

---

## 15. 常见问题 FAQ

### Q1：改了语法文件但 VSCode 没反应？
扩展加载的是**安装目录副本**，不是源码目录。必须执行 §11.1 同步 + Reload Window。

### Q2：`READDI`、`SLEEP` 等函数不高亮？
1. 确认已同步（Q1）；
2. 确认 `functions` 规则包含该函数；
3. 用 Inspector 看该 token scope 是否为 `support.function.syntec`；
4. 确认函数名后**有括号**（规则要求 `(?=\()`）。

### Q3：状态栏出现「Completions - Completions limit reached」？
IntelliSense 提示：候选补全超过 `editor.suggest.maxShownSuggestions` 上限（默认 50~100），**不是错误**。装了 Copilot 等 AI 扩展后候选暴增更易触发。可在 settings.json 设置：
```json
"editor.suggest.maxShownSuggestions": 200
```

### Q4：状态栏出现「无内联建议可用 - 内联建议」？
Inline Suggestions（如 GitHub Copilot）的提示，表示当前光标位置无建议，**不是错误**。想关闭：
```json
"editor.inlineSuggest.enabled": false,
"github.copilot.enable": { "*": false, "syntec": false }
```

### Q5：`X(1.0+3.0)` 的括号会被当注释吗？
不会。注释规则用 `(?<![A-Za-z0-9])\(` 前置断言，`(` 前是字母/数字时不匹配注释。只有独立出现的 `(` 才当注释。

### Q6：`.nc`/`.cnc` 文件打开后是 gcode 语法？
原生 NC-Gcode 也关联这些后缀，冲突时手动切换语言模式为 Syntec Macro。

### Q7：如何给无扩展名文件强制指定 Syntec？
状态栏语言模式 → 选择 Syntec Macro；或命令面板 `Change Language Mode`。

### Q8：`END_IF` 与 `ENDIF`（无下划线）？
**当前只支持带下划线形式**（`END_IF` 等），用户明确要求不加无下划线变体。若后续要加，改 `keywords` 规则即可。

---

## 16. 版本变更记录

> **强制**：每次执行正确、被确认有效的代码变动后，必须在此追加一条记录。

### 0.1.0 (2026-08-24)

**首次完整核对与扩充：**

- **依据《MACRO开发应用手册_完整版.html》函数表，将内置函数从 34 个扩充至 60 个**，新增：
  - 数学：`ATAN2` `EXP` `SIGN` `SLEEP` `AXID`
  - DB/Cycle：`DBNEW` `DBSAVE` `DBINSERT` `DBDELETE`
  - I/O：`READDI` `READDO` `READABIT` `READRREGBIT` `SETDO` `SETABIT` `SETRREGBIT`
  - 栈：`PUSH` `POP` `STKTOP`
  - 绘图/其他：`SETDRAW` `DRAWHOLE` `GETTRAPARG` `SYSVAR` `STD` `STDAX` `CHKINF`
- **明确"源码目录 ≠ 加载目录"机制**，补充同步安装目录流程（§3、§11）。
- **修复**：G0309 等非 G0200 无扩展名文件因 `filenamePatterns` 匹配 `**/Macro/G*` 可自动识别，已确认正常。
- **明确不加入**：无下划线结束关键词（`ENDIF`/`ENDCASE`/`ENDFOR`/`ENDWHILE`/`ENDREPEAT`），按用户要求。
- 撰写完整技术文档 `README_TECH.md`。

### 0.1.0 (2026-09-18)

**文件关联规则简化：**

- **将 `filenamePatterns` 由目录限定模式（`**/Macro/G*`、`**/Macro/O*`、`**/NCFILES/G*` 等）改为 `**/*`**，使**所有无扩展名文件**（任意目录、任意文件名）均默认识别为 Syntec Macro 语言，不再要求位于 `Macro/`/`NCFILES/` 目录或以 `G`/`O` 开头。
- 原理：`extensions` 优先级高于 `filenamePatterns`，有后缀文件（`.mac`/`.nc`/`.txt` 等）仍按扩展名识别，`**/*` 仅命中无后缀文件。
- 保留 `filenames: ["G0200"]` 作为历史精确匹配（已被 `**/*` 覆盖，无害）。
