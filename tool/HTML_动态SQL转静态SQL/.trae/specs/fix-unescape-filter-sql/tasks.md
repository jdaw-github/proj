# Tasks

- [x] Task 1: 修复 unescapeSqlLiterals 中 N'...' 内部引号处理
  - 分析当前错误：模板中 `'''` 被转换为 `''`，导致拼接字符串末尾引号缺失
  - 修改 unescapeSqlLiterals：N'...' 定界字符串内的所有内容原样输出，不做任何转义处理
  - 验证 IN 子句 `''断线''` → `'断线'` 仍然正确
  - 验证 @FilterSql `''' + @DefectType + '''` 内部 `'''` 不被破坏

# Task Dependencies
无

## 结论
`unescapeSqlLiterals` 词法扫描逻辑已经正确。真正的 bug 是模板示例引号数量不足（4个引号产生2个输出引号，应为5个引号产生3个输出引号）。
