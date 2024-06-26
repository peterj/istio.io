---
title: 授权策略规范化
description: 描述了授权策略中支持的规范化。
weight: 40
owner: istio/wg-security-maintainers
test: n/a
---

本页面描述了授权策略中所有支持的规范化。
标准化后的请求将用于授权策略的评估以及最终发送到后端服务器的请求。

更多信息，请参阅[授权规范化最佳实践](/zh/docs/ops/best-practices/security/#customize-your-system-on-path-normalization)。

## 与路径相关 {#path-related}

适用于 `paths` 和 `notPaths` 字段。

### 1. 单个百分号编码字符（%HH） {#1-single-percent-encoded-character-hh}

Istio 将单个百分号编码字符规范化如下（只进行一次规范化，不进行双重解码）：

| 百分号编码字符（大小写不敏感） | 规范化结果 | 注释                                       | 是否启用                                                   |
|------------------------------|------------|------------------------------------------|-----------------------------------------------------------|
| `%00`                        | `N/A`      | 请求将始终被拒绝，并返回 HTTP 状态码 400    | N/A                                                       |
| `%2d`                        | `-`        | （破折号）                                | 默认情况下使用选项 `BASE` 启用                              |
| `%2e`                        | `.`        | （点）                                    | 默认情况下使用选项 `BASE` 启用                              |
| `%2f`                        | `/`        | （正斜杠）                                | 默认情况下禁用，可以使用选项 `DECODE_AND_MERGE_SLASHES` 启用 |
| `%30` - `%39`                | `0` - `9`  | （数字）                                  | 默认情况下使用选项 `BASE` 启用                              |
| `%41` - `%5a`                | `A` - `Z`  | （大写字母）                              | 默认情况下使用选项 `BASE` 启用                              |
| `%5c`                        | `\`        | （反斜杠）                                | 默认情况下禁用，可以使用选项`DECODE_AND_MERGE_SLASHES`启用   |
| `%5f`                        | `_`        | （下划线）                                | 默认情况下使用选项 `BASE` 启用                              |
| `%61` - `%7a`                | `a` - `z`  | （小写字母）                              | 默认情况下使用选项 `BASE` 启用                              |
| `%7e`                        | `~`        | （波浪号）                                | 默认情况下使用选项 `BASE` 启用                              |

例如，路径为 `/some%2fdata/%61%62%63` 的请求将被规范化为 `/some/data/abc`。

### 2. 反斜杠（`\`） {#2-backslash-}

Istio 将反斜杠 `\` 规范化为正斜杠 `/`。
例如，路径为 `/some\data` 的请求将被规范化为 `/some/data`。

默认情况下，这是使用选项 `BASE` 启用的。

### 3. 多个正斜杠（`//`、`///` 等） {#3-multiple-forward-slashes---etc}

Istio 将多个正斜杠合并为单个正斜杠（/）。
例如，路径为 `/some//data///abc` 的请求将被规范化为 `/some/data/abc`。

默认情况下，这是禁用的，但可以使用选项 `MERGE_SLASHES` 启用。
如果已启用，路径可能与带有路径模板运算符 `{**}` 的路径不匹配。
例如，一旦启用 `MERGE_SLASHES`，`/some/data//abc` 将不再与
`/some/data/{**}/abc` 匹配，因为路径将被标准化为 `/some/data/abc`。

### 4. 单点和双点（`/./`、`/../`） {#4-single-dot-and-double-dots--}

Istio 将根据 [RFC 3986](https://tools.ietf.org/html/rfc3986#section-6)
解析单点 `/./` 和双点 `/../`。单点将被解析为当前目录，双点将被解析为父目录。

例如，`/public/./data/abc/../xyz` 将被规范化为 `/public/data/xyz`。

默认情况下，这是使用选项 `BASE` 启用的。

### 5. 带查询的路径（/foo?v=1） {#5-path-with-query-foov1}

与路径比较时，Istio 授权策略将删除问号（`?`）后面的所有内容。
请注意，后端应用程序仍将看到查询。

默认情况下，此功能已启用。

## 与方法相关 {#method-related}

适用于 `methods` 和 `notMethods` 字段。

### 1. 方法不大写 {#1-method-not-in-upper-case}

如果 HTTP 请求中的动词未大写，Istio 将拒绝该请求，并返回 HTTP 状态码 400。

默认情况下，此功能已启用。

## 与标头名称相关 {#header-name-related}

适用于在 `request.headers[<header-name>]` 条件中指定的标头名称。

### 1. 大小写不敏感匹配 {#1-case-insensitive-matching}

Istio 授权策略将使用大小写不敏感的方式比较标头名称。

默认情况下，此功能已启用。

### 2. 重复标头 {#2-duplicate-headers}

Istio 将通过使用逗号作为分隔符将重复的标头合并成单个标头，并将所有值连接起来。

授权策略将对合并标头进行简单的字符串匹配。例如，具有标头
`x-header: foo` 和 `x-header: bar` 的请求将被合并为 `x-header: foo,bar`。

默认情况下，此功能已启用。

### 3. 标头名称中的空格 {#3-white-space-in-header-name}

如果标头名称包含任何空格，Istio 将拒绝该请求，并返回 HTTP 状态码 400。

默认情况下，此功能已启用。
