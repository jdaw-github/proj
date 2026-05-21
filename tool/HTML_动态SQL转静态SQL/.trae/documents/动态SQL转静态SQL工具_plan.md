# 动态SQL转静态SQL工具 开发总结

## 1. 最终实现

纯HTML单文件工具（`index.html`），支持将 SQL Server `sp_executesql` 和 MySQL/PostgreSQL/Oracle 的参数化动态 SQL 转换为格式化静态 SQL。

## 2. 迭代过程

### 迭代 1：基础自动/手动模式
- 创建HTML骨架、CSS样式、基础JS逻辑
- 自动检测 `sp_executesql` 前缀
- 手动模式解析 `:name` / `?` / `$name` 占位符

### 迭代 2：sp_executesql 解析
- 实现 `parseNStringArgs` 按逗号拆分 N'...' 参数
- 实现 `parseQuotedString` 正确处理 `''` → `'` 转义
- 实现参数替换

### 迭代 3：批量处理 go 分隔
- `splitByGo` 分割多条语句
- 每条语句独立 try-catch

### 迭代 4：参数格式扩展
- 命名参数 `@P0='value'` 格式支持（`parseValueToken` 中剥离 `@param=` 前缀）
- NULL 裸值支持
- 非 N'...' 的普通 '...' 字符串支持

### 迭代 5：类型感知
- 从参数声明中提取类型信息
- INT/BIGINT/BIT 等数值类型不加引号
- NVARCHAR/DATETIME/UNIQUEIDENTIFIER 等加引号

### 迭代 6：参数替换顺序
- 按参数名长度降序排列（`@P11` > `@P10` > ... > `@P1` > `@P0`）
- 解决 `@P1` 正则匹配污染 `@P10` 的问题

### 迭代 7：Smart Unescape
- 实现 `unescapeSqlLiterals` 词法扫描
- 优先级链：`N'...'`（原样保留） > `''`（→ `'`） > `'...'`（原样保留） > `--注释`（原样保留）
- 既能使 `IN (''断线'')` → `IN ('断线')`
- 又能保护 `N'...!= ''' + ...` 不被破坏

### 迭代 8：格式化调整
- 移除空格缩进的设置控件
- 改用 Tab 缩进（`\t`）
- `formatSql` 内部 try-catch 兜底

## 3. 最终架构

```
输入
  ↓ 检测：sp_executesql ? 
  ├─ 是 → parseSpExecutesql
  │         → parseNStringArgs (拆分参数)
  │         → 类型推断 + 参数替换 (长度降序)
  │         → unescapeSqlLiterals (词法扫描)
  │         → formatSql (sql-formatter)
  └─ 否 → extractGenericParams → 用户填值 → applyConversionWithParams → formatSql
输出
```

## 4. 文件清单

| 文件 | 说明 |
|------|------|
| `index.html` | 主程序，包含HTML/CSS/JS全部逻辑 |
| `.trae/documents/prd.md` | 产品需求文档 |
| `.trae/documents/arch.md` | 技术架构文档 |
| `.trae/documents/动态SQL转静态SQL工具_plan.md` | 开发计划与总结 |
