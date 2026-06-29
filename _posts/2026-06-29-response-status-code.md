---
layout: post
title: 'FastAPI 响应状态码指南'
subtitle: '响应状态码'
date: 2026-06-29
categories: tech
author: yucol
tags: [FastAPI, 响应状态码]

cover: 'https://img.yucol.uk/2026/06/056bfa3e19d89ffb4372f59ae926fbca.webp'
cover_author: 'Willian Justen de Vasconcellos'
cover_author_link: 'https://unsplash.com/@willianjusten'
---


# FastAPI 响应状态码指南

在构建 RESTful API 时，正确使用 HTTP 状态码是至关重要的。它不仅是服务器与客户端通信的“语言”，更是衡量 API 设计是否规范、健壮的关键指标。FastAPI 作为一款现代化的 Web 框架，提供了极其便捷的方式来处理响应状态码。本文将深入解析 FastAPI 中关于响应状态码的一切，助你打造专业级的 API 接口。

---

## 为什么要关注 HTTP 状态码？

HTTP 状态码是服务器在处理客户端请求后返回的三位数字代码，它简洁明了地表达了请求的处理结果。一个设计良好的 API 应该利用状态码来传达语义，而不是仅仅依赖响应体中的自定义字段。

根据 FastAPI 官方文档的逻辑，状态码通常分为以下几类：

- **100-199 (信息)**：表示接收的请求正在处理中，通常很少直接使用。
- **200-299 (成功)**：表示请求已成功接收、理解并接受。这是最常用的类别。
    - `200 OK`：默认状态码，表示一切正常。
    - `201 Created`：表示资源创建成功，通常在 POST 请求后使用。
    - `204 No Content`：表示请求成功，但没有返回内容（常用于 DELETE 请求）。
- **300-399 (重定向)**：表示需要客户端采取进一步的操作才能完成请求。
- **400-499 (客户端错误)**：表示客户端请求有误。
    - `400 Bad Request`：通用的客户端错误。
    - `404 Not Found`：请求的资源不存在。
- **500-599 (服务器错误)**：表示服务器在处理请求时发生了错误，通常由框架或服务器自动返回。

---

## 在 FastAPI 中声明状态码

在 FastAPI 中，你可以通过路径操作装饰器（如 `@app.get()`）的 `status_code` 参数来声明响应状态码。这不仅会改变实际的 HTTP 响应状态，还会自动更新 OpenAPI 文档（Swagger UI），让 API 的使用者一目了然。

### 基础用法：使用数字

你可以直接传递一个三位数的整数作为 `status_code` 的值。

```python
from fastapi import FastAPI

app = FastAPI()

@app.post("/items/", status_code=201)
async def create_item(name: str):
    return {"name": name}
```

**说明**：在上述代码中，我们创建了一个 POST 接口，当用户添加新项目时，服务器将返回 `201 Created` 状态码，而不是默认的 `200 OK`。

### 进阶用法：使用 status 模块（推荐）

硬编码数字虽然可行，但容易出错且不易记忆。FastAPI 提供了 `status` 模块，其中包含了所有标准的 HTTP 状态码常量，利用 IDE 的自动补全功能可以极大提高开发效率。

```python
from fastapi import FastAPI, status

app = FastAPI()

@app.post("/items/", status_code=status.HTTP_201_CREATED)
async def create_item(name: str):
    return {"name": name}
```

这种方式不仅代码可读性更强，还能有效避免拼写错误。

---

## 状态码对文档和响应体的影响

FastAPI 是一个智能的框架，它会根据你选择的状态码自动推断文档信息：

- **文档自动生成**：当你指定 `status_code=201` 时，FastAPI 会在生成的 OpenAPI 文档中自动记录该接口可能返回 `201` 状态码。
- **响应体逻辑**：某些状态码（如 `204 No Content`）明确规定不能包含响应体。FastAPI 会识别这些状态码，并在文档中声明该响应没有响应体，从而避免产生歧义。

---

## 最佳实践与常见误区

1. **区分默认值与实际返回值**
在路径操作函数中声明的 `status_code` 是“默认”值。在后续的高级用法中（如依赖注入或异常处理），你可能会根据业务逻辑动态改变最终返回的状态码。
1. 善用 `201 Created` 与 `204 No Content`
    - 在创建资源（POST）时，优先使用 `201 Created`，这符合 RESTful 规范。
    - 在删除资源（DELETE）或更新资源（PUT/PATCH）且无需返回数据时，使用 `204 No Content` 可以减少网络传输的数据量。
2. **错误处理**
虽然 FastAPI 会自动处理大部分服务器错误（5xx），但对于客户端错误（4xx），建议结合 `HTTPException` 使用：