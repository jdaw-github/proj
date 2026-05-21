# 动态SQL转静态SQL — 解析流程分析

## 概述

这是一个纯前端单页应用，核心功能是将参数化的**动态SQL**（如 SQL Server 的 `sp_executesql` 调用、MySQL/PostgreSQL 的命名参数、问号占位符等）转换为可直接执行的**静态SQL**（参数已被具体值替换），并自动格式化输出。

整个工具的设计体现了**双模式引擎**的思路：遇到 `sp_executesql` 自动解析所有参数 → 替换 → 输出；遇到其他格式则让用户手动填写参数值。

---

## 核心转换链路（全流程）

> 输入：原始动态SQL字符串
>
> ↓ **入口 detect**
>
> 步骤1：模式检测（自动模式 / 手动模式）
>
> ↓ **自动模式**（sp_executesql）
>
> 步骤2：按 `GO` 分隔多条语句
>
> 步骤3：从 sp_executesql 中剥离前缀 `exec sp_executesql`
>
> 步骤4：用 `parseNStringArgs` 解析 `N'...'` 参数列表 → 得到模板SQL + 参数声明 + 参数值
>
> 步骤5：从参数声明字符串中提取参数名和类型
>
> 步骤6：对模板SQL执行 **Smart Unescape**（`unescapeSqlLiterals`）
>
> 步骤7：按参数名长度**降序**替换（防止部分匹配问题）
>
> 步骤8：类型感知替换——数值类型不加引号，字符串类型加单引号并转义
>
> 步骤9：使用 `sql-formatter` 格式化
>
> 步骤10：多语句用 `go` 连接输出
>
> ↓ **手动模式**（非 sp_executesql）
>
> 步骤A：提取所有参数占位符（`:name / @name / $1 / ?`）
>
> 步骤B：渲染参数配置表格让用户填写
>
> 步骤C：用户填写后，逐个替换占位符为填写的值
>
> 步骤D：格式化输出

---

## 分步深度解析

### 第一步：模式检测

**代码位置**：[index.html:L386](file:///d:/1.HC/VScode/MyAiProj/tool/HTML_动态SQL转静态SQL/index.html#L386-L403)

```javascript
/^exec\s+sp_executesql\s+/im.test(rawSql.trim())
```

- 使用正则 `exec sp_executesql`（不区分大小写、多行模式）检测输入
- 匹配 → **自动模式**（`convertSpExecutesql`）
- 不匹配 → **手动模式**（`extractGenericParams` + `convertGenericSql`）
- 输入框的 `input` 事件实时更新右上角的徽章（Auto/Manual 标签）

**设计要点**：
- 为什么用 `im` 标志？`i` 忽略大小写（SQL 不区分大小写），`m` 让 `^` 也能匹配行首（用户可能前面有空白行）
- 这是整个流程的路由器，决定了后续走哪条处理链路

---

### 第二步：按 `GO` 分隔（批量处理）

**代码位置**：[index.html:L405-L407](file:///d:/1.HC/VScode/MyAiProj/tool/HTML_动态SQL转静态SQL/index.html#L405-L407)

```javascript
function splitByGo(text) {
    return text.split(/\n\s*go\s*(\n|$)/gi);
}
```

- 用正则 `\n\s*go\s*(\n|$)` 分割字符串
- 解释：匹配"换行 → 任意空白 → go → 任意空白 → 换行或结尾"
- 每条语句独立处理、独立 try-catch，一条失败不影响其他

**设计要点**：
- SSMS（SQL Server Management Studio）中使用 `GO` 分隔批处理，这里模拟了相同语义
- 每个 segment 单独调 `parseSpExecutesql`，保证隔离性

---

### 第三步（自动模式）：解析 `sp_executesql` 参数列表

这是整个工具最复杂的部分，涉及多层嵌套解析。

#### 3a. 剥离前缀

**代码位置**：[index.html:L446-L452](file:///d:/1.HC/VScode/MyAiProj/tool/HTML_动态SQL转静态SQL/index.html#L446-L452)

```javascript
const prefixMatch = trimmed.match(/^exec\s+sp_executesql\s+/i);
const rest = trimmed.slice(prefixMatch[0].length);
const args = parseNStringArgs(rest);
```

- 去掉 `exec sp_executesql ` 前缀，剩余部分就是参数列表
- 要求至少 2 个参数：SQL模板 + 参数声明；第3个及之后是参数值

#### 3b. `parseNStringArgs` — 顶级参数分割

**代码位置**：[index.html:L569-L587](file:///d:/1.HC/VScode/MyAiProj/tool/HTML_动态SQL转静态SQL/index.html#L569-L587)

```javascript
function parseNStringArgs(text) {
    const args = [];
    let i = 0;
    while (i < text.length) {
        // 跳过空白和逗号分隔符
        while (i < text.length && (text[i] === ' ' || text[i] === ',' || ...)) i++;
        if (i >= text.length) break;
        const result = parseValueToken(text, i);
        if (result) {
            args.push(result.value);
            i = result.endIndex;
        } else break;
    }
    return args;
}
```

- 用 `while` 循环遍历，跳过空白 + 逗号分隔符
- 对每个位置调用 `parseValueToken` 解析一个完整的参数值
- 将解析出的参数值收集到 `args` 数组中

**示例**：对于输入 `N'SELECT ...', N'@P0 int', 1`
- 第一次调用 parseValueToken 解析出 `SELECT * FROM users WHERE id = @P0`
- 第二次解析出 `@P0 int`
- 第三次解析出 `1`

#### 3c. `parseValueToken` — 单参数值解析（多态分发）

**代码位置**：[index.html:L589-L614](file:///d:/1.HC/VScode/MyAiProj/tool/HTML_动态SQL转静态SQL/index.html#L589-L614)

```javascript
function parseValueToken(text, start) {
    // 1. 尝试匹配命名参数 @name=...
    const namedMatch = text.slice(start).match(/^@\w+=\s*/);
    if (namedMatch) {
        // 递归解析等号后面的值
        return parseValueToken(text, vStart);
    }
    // 2. 尝试解析 N'...' 或 '...' 字符串
    const nOrPlain = parseNOrPlainString(text, start);
    if (nOrPlain) return nOrPlain;
    // 3. 尝试 NULL 关键字
    if (nullMatch) return { value: 'NULL', endIndex: ... };
    // 4. 兜底：匹配一个单词（数值、标识符等）
    if (wordMatch) return { value: wordMatch[0], endIndex: ... };
}
```

这是一个**多态解析器**，按优先级尝试四种情况：
1. **命名参数语法** `@name=value` — 递归调用自身解析等号后面的值
2. **N'...' 或 '...' 字符串** — 委托给 `parseNOrPlainString`
3. **NULL 字面量**
4. **普通单词**（数值、标识符）

#### 3d. `parseNOrPlainString` → `parseQuotedString` — 带引号字符串解析

**代码位置**：[index.html:L616-L643](file:///d:/1.HC/VScode/MyAiProj/tool/HTML_动态SQL转静态SQL/index.html#L616-L643)

```javascript
function parseNOrPlainString(text, start) {
    // N'...' 或 '...'
}

function parseQuotedString(text, start, quote) {
    let result = '';
    let i = start;
    while (i < text.length) {
        if (text[i] === quote) {
            if (i + 1 < text.length && text[i + 1] === quote) {
                result += quote;  // '' → ' 保留
                i += 2;
            } else {
                return { value: result, endIndex: i + 1 };  // 闭合引号
            }
        } else {
            result += text[i];
            i++;
        }
    }
    throw new Error('未闭合的引号字符串');
}
```

这里要特别说明：

- `N'...'` 是 `parseNOrPlainString` 中检测到 `N` + `'` 后，调用 `parseQuotedString(text, start+2, "'")` ——跳过了 `N'` 两个字符
- 内部遇到 `''` → 合并为一个 `'`（这是 ANSI SQL 的转义规则）
- 遇到单个 `'` → 字符串结束
- 如果到末尾还没找到闭合引号 → 抛出异常

**示例解析过程**：

输入：`N'SELECT * FROM users WHERE id = @P0', N'@P0 int', 1`

| 调用 | 起始位置 | 解析结果 |
|------|---------|---------|
| parseValueToken | 0 | 匹配 N' → parseQuotedString → `SELECT * FROM users WHERE id = @P0` |
| parseValueToken | 跳过 `, ` → 匹配 N' → | `@P0 int` |
| parseValueToken | 跳过 `, ` → 匹配单词 → | `1` |

---

### 第四步（自动模式）：参数声明解析

**代码位置**：[index.html:L461-L466](file:///d:/1.HC/VScode/MyAiProj/tool/HTML_动态SQL转静态SQL/index.html#L461-L466)

```javascript
const paramDecls = paramDeclStr.split(',')
    .map(s => {
        const m = s.trim().match(/^(@\w+)\s+(\w+)/);
        return m ? { name: m[1], type: m[2].toUpperCase() } : null;
    })
    .filter(Boolean);
```

- 参数声明字符串 `@P0 int, @P1 varchar(50)` 按逗号分割
- 每个分段用正则 `^(@\w+)\s+(\w+)` 提取参数名和类型关键字
- 结果存入 `paramDecls` 数组（`{name, type}`）

**注意**：`(\w+)` 只能匹配类型名的第一个单词，所以 `varchar(50)` 只会被匹配为 `varchar`，括号内精度信息被丢弃。但这不影响类型感知判断，因为关键的类型大类（INT、VARCHAR等）都在第一个单词中。

---

### 第五步（自动模式）：Smart Unescape

**代码位置**：[index.html:L495-L567](file:///d:/1.HC/VScode/MyAiProj/tool/HTML_动态SQL转静态SQL/index.html#L495-L567)

这是工具中**最核心、最精妙**的函数。它处理的问题是：

在 `N'...'` 内部的 SQL 模板中，字符串字面量的 `''`（两个单引号）是 SQL 的转义写法，表示一个单引号字符。在转换时需要把 `''` 还原为 `'`。但问题是——**`N'...'` 外层的包裹符也是单引号**，我们不能把外层的 `N'...'` 给拆了。

**函数 `unescapeSqlLiterals` 原理**：

使用**词法扫描**（逐字符遍历），维护一个状态机：

```
状态机状态：
- OUTSIDE_STRING  : 普通字符，直接输出
- INSIDE_REGULAR  : 在 '...' 内部（非 N 前缀）
- INSIDE_N_STRING : 在 N'...' 内部
```

```javascript
// 伪代码流程
for each char at position i:
    如果遇到 N' 开头 → 进入 N'...' 区域
        内部：'' → 检查后面是否是 + @name（拼接语法）→ 是则退出
              否则保留 ''
        内部：单 ' → 退出 N'...' 区域
        内部：其他字符 → 原样输出
    
    如果遇到 '''（三个引号）→ 进入普通字符串区域
        内部：'' → 保留
        内部：单 ' → 退出
    
    如果遇到 ''（两个引号）→ 转换为一个 '
    如果遇到 ' → 进入普通字符串区域
    
    如果遇到 -- → 跳过整行注释（原样保留）
    其他 → 直接输出
```

**关键设计决策**：

1. **N'...' 内部不把 `''` 转成 `'`**：因为 `N'...'` 包裹的是 SQL 模板的完整文本，内部的 `''` 本身就是要保留的转义表示法。例如 `N'SELECT * FROM users WHERE name = ''admin'''`，内部的 `''` 需要保留到后面替换参数时再用。

   等等，这里需要更精确的理解。让我们看具体逻辑：

   - `N'...'` 内部遇到 `''` 时：会检查后面是否是 `+ @`（字符串拼接语法），如果是则输出一个 `'` 并退出；否则输出 `''` 保留原样
   - 换句话说：**N'...' 内部的 `''` 只有出现在拼接边界时才被转换，否则保留**
   
   这个设计是因为 `sp_executesql` 的调用中，参数值可能通过 `N'...' + @param` 拼接，此时 `N'...'` 内部的 `''` 需要在边界处解开。

2. **普通 '...' 字符串内部**：`''` → `'`（标准的 ANSI SQL 转义还原）

3. **`'''` 三个引号**：这是 SQL 中引号字符串内部包含一个转义单引号的常见写法，比如 `'''abc'` 等价于 `'abc'`。

> **⚠️ 重要说明**：经过对 `fix-unescape-filter-sql` 规格文档的查看，之前的版本存在一个 bug：当 `N'...'` 内部拼接字符串使用 `'''+` 时（三引号），`unescapeSqlLiterals` 可能错误转义。后来确认词法扫描逻辑本身是正确的，修复点在于理解：**模板中的 `''` 数量决定输出时的 `'` 数量**（4个输入引号→2个输出引号，5个输入引号→3个输出引号）。

---

### 第六步（自动模式）：参数替换

**代码位置**：[index.html:L471-L490](file:///d:/1.HC/VScode/MyAiProj/tool/HTML_动态SQL转静态SQL/index.html#L471-L490)

```javascript
// 按参数名长度降序排列
const indexedParams = paramNames.map((name, index) => ({ name, index }));
indexedParams.sort((a, b) => b.name.length - a.name.length);

let resultSql = unescapeSqlLiterals(template);
indexedParams.forEach(({ name, index }) => {
    const value = index < paramValues.length ? paramValues[index] : '';
    const decl = paramDecls[index];
    const isNumeric = decl && numericTypes.has(decl.type);

    let replacement;
    if (value === 'NULL' || value === 'null') {
        replacement = 'NULL';
    } else if (isNumeric) {
        replacement = value;              // 数值：不加引号
    } else {
        replacement = `'${escapeSingleQuote(value)}'`;  // 字符串：加引号并转义
    }
    const regex = new RegExp(escapeRegex(name), 'g');
    resultSql = resultSql.replace(regex, replacement);
});
```

**三步处理**：

1. **排序**：按参数名**长度降序**替换。为什么？防止 `@P1` 被错误地匹配到 `@P10` 中的 `@P1` 部分。先替换 `@P10` 再替换 `@P1` 就不会有这个问题。

2. **类型感知**：
   - `numericTypes = new Set(['INT', 'BIGINT', 'SMALLINT', 'TINYINT', 'BIT', 'FLOAT', 'REAL', 'DECIMAL', 'NUMERIC', 'MONEY', 'SMALLMONEY'])`
   - 数值类型 → 值直接拼入 SQL，**不加引号**
   - 字符串类型 → 值用单引号包裹，并且内部 `'` 转义为 `''`
   - NULL → 直接输出 `NULL` 关键字

3. **转义**：
   - `escapeSingleQuote`：`'` → `''`
   - `escapeRegex`：对参数名中的特殊正则字符转义

---

### 第七步：SQL 格式化

**代码位置**：[index.html:L757-L770](file:///d:/1.HC/VScode/MyAiProj/tool/HTML_动态SQL转静态SQL/index.html#L757-L770)

```javascript
function formatSql(sql, dbType) {
    if (typeof window.sqlFormatter !== 'undefined' && window.sqlFormatter.format) {
        const langMap = { mysql: 'mysql', postgresql: 'postgresql', oracle: 'sql', sqlserver: 'tsql' };
        return window.sqlFormatter.format(sql, { language: langMap[dbType] || 'sql', indent: '\t' });
    }
    return sql;
}
```

- 使用 `sql-formatter@15.3.1`（CDN 引入）进行格式化
- 数据库类型映射到 formatter 的语言参数
- 缩进使用 Tab（`\t`）
- 如果 formatter 加载失败，降级返回原文本

---

### 手动模式处理（非 sp_executesql）

#### 参数提取 `extractGenericParams`

**代码位置**：[index.html:L656-L678](file:///d:/1.HC/VScode/MyAiProj/tool/HTML_动态SQL转静态SQL/index.html#L656-L678)

```javascript
const cleaned = sql.replace(/'[^']*'|"[^"]*"/g, m => ' '.repeat(m.length));
const regex = /(:\w+|@\w+|\$\w+|\?)/g;
```

- 先清除字符串字面量内部的文本（替换为等长空格），避免误匹配字符串内的占位符
- 使用正则匹配 `:name` / `@name` / `$1` / `?` 四种占位符格式
- `?` 按出现顺序命名为 `param_0`、`param_1`...

#### 实时替换 `applyConversionWithParams`

**代码位置**：[index.html:L711-L738](file:///d:/1.HC/VScode/MyAiProj/tool/HTML_动态SQL转静态SQL/index.html#L711-L738)

- 使用 `String.split` + 正则遍历，将 SQL 文本拆分为"普通段 + 占位符段"的交替序列
- 每个占位符查找对应的参数配置，调用 `formatValue` 生成替换值
- 重新拼接后格式化输出

#### 值格式化 `formatValue`

**代码位置**：[index.html:L740-L747](file:///d:/1.HC/VScode/MyAiProj/tool/HTML_动态SQL转静态SQL/index.html#L740-L747)

```javascript
function formatValue(param, dbType) {
    if (param.type === 'null') return 'NULL';
    if (param.type === 'number') return param.value || 'NULL';
    if (param.type === 'date') {
        if (dbType === 'oracle') return `TO_DATE('${param.value}', 'YYYY-MM-DD')`;
    }
    return `'${escapeSingleQuote(param.value)}'`;
}
```

- `null` → `NULL`
- `number` → 直接数值
- `date` + Oracle → `TO_DATE(...)`
- 字符串 → `'转义后的值'`

---

## 完整示例推演

### 示例：SQL Server sp_executesql

**输入**：
```sql
exec sp_executesql N'SELECT * FROM users WHERE id = @P0 AND name = @P1 AND status IN (''断线'',''重连'')', N'@P0 int, @P1 nvarchar(50), @P2 int', 1, N''''断线''''', 3
```

**步骤解析**：

1. **模式检测**：匹配 `exec sp_executesql` → 自动模式

2. **parseNStringArgs 解析参数列表**：
   - arg[0] = `SELECT * FROM users WHERE id = @P0 AND name = @P1 AND status IN (''断线'',''重连'')`
   - arg[1] = `@P0 int, @P1 nvarchar(50), @P2 int`
   - arg[2] = `1`
   - arg[3] = `''''断线''''`   ← 注意：这里保留了原始转义

3. **参数声明解析**：
   - `@P0` → type: `INT`
   - `@P1` → type: `NVARCHAR`
   - `@P2` → type: `INT`

4. **Smart Unescape 模板**：
   - N'...' 内部的 `''断线'',''重连''` → 保留 `''`（因为不在拼接边界）
   - 输出：`SELECT * FROM users WHERE id = @P0 AND name = @P1 AND status IN (''断线'',''重连'')`

5. **参数替换**（按长度降序：@P2 > @P1 > @P0）：
   - `@P0` (INT) → `1`（不加引号）
   - `@P1` (NVARCHAR) → `'''断线'''`（加引号，内部 `'` → `''`）
   
   等等，这里 arg[3] 的值是 `''''断线''''`，经过 `escapeSingleQuote` 后变成什么？
   
   实际上，`escapeSingleQuote` 只在 `formatValue` 中调用，而自动模式下参数值替换走的是 `parseSpExecutesql` 中的直接替换逻辑：
   
   ```javascript
   replacement = `'${escapeSingleQuote(value)}'`;
   ```
   
   对于 `@P1`，其值是 arg[1]（注意 `P1` 在声明中是第1个索引，值从第0个索引开始），对应的是第1个索引的值。让我重新梳理索引...

   实际上索引对应关系是：
   - `paramDecls`：`[0: @P0 int, 1: @P1 nvarchar(50), 2: @P2 int]`
   - `paramValues`：`[0: '1', 1: '''''断线''''', 2: ('3' — 未提供)]`
   
   但 `paramNames` 是 `['@P0', '@P1', '@P2']`
   
   替换时，`@P0` 取 `paramValues[0]` = `'1'`，类型 INT → 不加引号直接放 `1`
   `@P1` 取 `paramValues[1]` = `''''断线''''`，类型 NVARCHAR → `'''...'''`（加引号并转义）
   `@P2` 取 `paramValues[2]` — 不存在，取 `''`

6. **格式化**：使用 sql-formatter 排版

7. **输出**：
   ```sql
   SELECT
       *
   FROM
       users
   WHERE
       id = 1
       AND name = '''断线'''
       AND status IN ('断线', '重连')
   ```

---

## 关键设计决策总结

| 决策 | 原因 | 代码位置 |
|------|------|---------|
| 双模式（自动/手动） | sp_executesql 有固定的参数传递结构，可全自动；其他格式需要人工辅助 | L386-L403 |
| 参数按长度降序替换 | 防止 `@P1` 错误匹配到 `@P10` 中的 `@P1` | L472 |
| N'...' 内部 `''` 不转换 | N'...' 包裹的是模板文本，内部的 `''` 可能是 SQL 字面量转义 | L503-L511 |
| 类型感知 ± 引号 | 数值 `1` 写成 `'1'` 会导致隐式类型转换，影响索引使用 | L477-L487 |
| 批量 GO 分隔 | 实际工作中经常需要一次转换多条 sp_executesql 语句 | L405-L407 |
| 词法扫描而非正则 | 引号嵌套、注释等复杂场景只能用状态机精确处理 | L495-L567 |
| 先 Unescape 再替换 | 如果先替换参数再 Unescape，可能破坏参数值中的内容 | L474 |
| 字符串替换前清除引号内容 | 避免将字符串内部的 `:name` 误识别为占位符 | L661 |

---

## 函数调用关系图

```
convertSql()                                  ← 入口
│
├─ sp_executesql 检测 ─→ convertSpExecutesql()
│                           │
│                           ├─ splitByGo()                    ← 分割GO批处理
│                           ├─ parseSpExecutesql()            ← 单条解析
│                           │     ├─ parseNStringArgs()       ← 外层参数分割
│                           │     │     └─ parseValueToken()  ← 多态参数值解析
│                           │     │           ├─ parseNOrPlainString()
│                           │     │           │     └─ parseQuotedString()
│                           │     │           └─ NULL / 单词匹配
│                           │     ├─ 参数声明解析 (纯正则)
│                           │     ├─ unescapeSqlLiterals()    ← Smart Unescape
│                           │     └─ 参数替换 (类型感知 + 降序)
│                           └─ formatSql()                    ← sql-formatter
│
└─ 非 sp_executesql ─→ extractGenericParams()  ← 提取占位符
                        └─ renderParams()        ← 渲染配置表格
                        └─ applyConversionWithParams()  ← 用户填写后实时替换
                              └─ formatValue()  ← 类型感知格式化值
                              └─ formatSql()    ← sql-formatter
```
