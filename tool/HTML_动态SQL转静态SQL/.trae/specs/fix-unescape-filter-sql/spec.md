# fix-unescape-filter-sql Spec

## Why
`unescapeSqlLiterals` 在处理 `N'...'` 内部拼接字符串时，将 `'''` 错误地转换为 `''`，导致字符串拼接引号未闭合，SQL 语法报错。

## What Changes
- 修复 `unescapeSqlLiterals` 的词法扫描逻辑
- 确保 `N'...'` 和 `'...'` 定界字符串内部内容完全原样保留

## Impact
- Affected specs: prd.md 2.4 Smart Unescape
- Affected code: `index.html` 中的 `unescapeSqlLiterals` 函数

## ADDED Requirements
无新需求

## MODIFIED Requirements
### Requirement: Smart Unescape 正确处理 N'...' 内部拼接
`unescapeSqlLiterals` 函数必须对所有 `'...'` 和 `N'...'` 定界字符串的内容原样保留，不得修改其内部的任何字符（包括单引号、双单引号等）。

#### Scenario: 嵌套 EXEC 的 FilterSql 参数
- **WHEN** 模板中包含 `N'WHERE sDefectName != ''' + @DefectType + '''`
- **THEN** 输出为 `N'WHERE sDefectName != ''' + '污渍' + '''`，即 `N'...'` 内部完全不变
- **AND** 拼接后的字符串能正确闭合

#### Scenario: IN 子句的字面量
- **WHEN** 模板中包含 `IN (''断线'', ''跳线'')`
- **THEN** 输出为 `IN ('断线', '跳线')`，`''` → `'` 正确转换
- **AND** 多余空格应被保留（来自模板原始内容）

## REMOVED Requirements
无
