# RESTful API 设计指南

## 概述

RESTful API 是一种基于 REST（Representational State Transfer）架构风格设计的 Web API。它使用标准的 HTTP 方法、状态码和资源导向的 URL 结构，为客户端和服务器之间的通信提供一致的接口。

## 核心原则

### 1. 资源导向 (Resource-Oriented)
**一切皆资源，使用名词而非动词**

```markdown
✅ 正确示例：
GET    /users          # 获取用户列表
GET    /users/{id}     # 获取特定用户
POST   /users          # 创建新用户
PUT    /users/{id}     # 更新用户
DELETE /users/{id}     # 删除用户

❌ 错误示例：
GET    /getUsers
POST   /createUser
POST   /updateUser
GET    /deleteUser
```

### 2. 统一接口 (Uniform Interface)
**使用标准的 HTTP 方法和状态码**

| HTTP 方法 | 语义 | 幂等性 | 安全性 |
|-----------|------|--------|--------|
| **GET** | 获取资源 | 是 | 是 |
| **POST** | 创建资源 | 否 | 否 |
| **PUT** | 更新/替换资源 | 是 | 否 |
| **PATCH** | 部分更新资源 | 否 | 否 |
| **DELETE** | 删除资源 | 是 | 否 |
| **HEAD** | 获取资源头信息 | 是 | 是 |
| **OPTIONS** | 获取支持的通信选项 | 是 | 是 |

### 3. 无状态 (Stateless)
**每个请求包含所有必要信息**

```markdown
✅ 无状态设计：
- 每个请求独立处理
- 服务器不保存客户端状态
- 身份验证信息通过Token传递

❌ 有状态设计：
- 服务器保存session状态
- 依赖服务器端状态管理
```

## URL 设计规范

### 资源命名
```markdown
# 使用名词复数形式
/users          # 而不是 /user
/products       # 而不是 /product

# 使用连字符(-)而非下划线(_)
/user-roles     # 而不是 /user_roles

# 保持URL小写
/users/admin    # 而不是 /Users/Admin
```

### 资源层级
```markdown
# 嵌套资源关系
GET /users/{userId}/orders          # 获取用户的订单
GET /users/{userId}/orders/{orderId} # 获取用户的特定订单

# 避免过深嵌套
GET /users/{userId}/orders/{orderId}/items/{itemId} # 避免这样设计
```

### 查询参数
```markdown
# 过滤
GET /users?role=admin&status=active

# 分页
GET /users?page=2&limit=20

# 排序
GET /users?sort=name,-created_at  # 名称升序，创建时间降序

# 搜索
GET /users?q=john&fields=name,email
```

## HTTP 状态码使用

### 2xx 成功
```markdown
- **200 OK**: 请求成功
- **201 Created**: 资源创建成功
- **202 Accepted**: 请求已接受，处理中
- **204 No Content**: 请求成功，无返回内容
```

### 4xx 客户端错误
```markdown
- **400 Bad Request**: 请求参数错误
- **401 Unauthorized**: 未认证
- **403 Forbidden**: 无权限访问
- **404 Not Found**: 资源不存在
- **409 Conflict**: 资源冲突
- **429 Too Many Requests**: 请求过于频繁
```

### 5xx 服务器错误
```markdown
- **500 Internal Server Error**: 服务器内部错误
- **502 Bad Gateway**: 网关错误
- **503 Service Unavailable**: 服务不可用
- **504 Gateway Timeout**: 网关超时
```

## 请求和响应格式

### 请求头
```http
GET /users/123 HTTP/1.1
Host: api.example.com
Authorization: Bearer <token>
Accept: application/json
Content-Type: application/json
If-None-Match: "686897696a7c876b7e"
```

### 响应头
```http
HTTP/1.1 200 OK
Content-Type: application/json
ETag: "686897696a7c876b7e"
Last-Modified: Wed, 21 Oct 2015 07:28:00 GMT
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1402425459
```

### JSON 响应格式
```json
{
  "data": {
    "id": "123",
    "type": "users",
    "attributes": {
      "name": "John Doe",
      "email": "john@example.com",
      "created_at": "2023-01-01T00:00:00Z"
    },
    "relationships": {
      "orders": {
        "links": {
          "self": "/users/123/relationships/orders",
          "related": "/users/123/orders"
        }
      }
    }
  },
  "links": {
    "self": "/users/123",
    "related": "/users"
  },
  "meta": {
    "copyright": "Copyright 2023 Example Corp.",
    "authors": ["John Doe"]
  }
}
```

## 版本管理

### URL 版本控制
```markdown
# 在URL中包含版本号
/api/v1/users
/api/v2/users

# 优点：简单明了
# 缺点：URL变得冗长
```

### Header 版本控制
```http
GET /users HTTP/1.1
Accept: application/vnd.example.v1+json
```

```markdown
# 优点：保持URL简洁
# 缺点：需要客户端设置正确的Header
```

## 认证和授权

### JWT 认证
```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### API Key
```http
X-API-Key: abc123def456ghi789
```

### OAuth 2.0
```http
Authorization: Bearer <access_token>
```

## 分页设计

### 基于偏移的分页
```json
{
  "data": [...],
  "pagination": {
    "total": 1000,
    "count": 20,
    "per_page": 20,
    "current_page": 2,
    "total_pages": 50,
    "links": {
      "first": "/users?page=1",
      "last": "/users?page=50",
      "prev": "/users?page=1",
      "next": "/users?page=3"
    }
  }
}
```

### 基于游标的分页
```json
{
  "data": [...],
  "pagination": {
    "next_cursor": "abc123",
    "has_more": true,
    "total": 1000
  }
}
```

## 错误处理

### 统一错误格式
```json
{
  "error": {
    "code": "invalid_parameter",
    "message": "The 'email' parameter is invalid",
    "target": "email",
    "details": [
      {
        "code": "empty_value",
        "message": "Email cannot be empty"
      }
    ]
  },
  "request_id": "req_123456789",
  "timestamp": "2023-01-01T00:00:00Z"
}
```

### 验证错误
```json
{
  "errors": [
    {
      "field": "email",
      "message": "Email is required",
      "code": "required"
    },
    {
      "field": "password",
      "message": "Password must be at least 8 characters",
      "code": "min_length"
    }
  ]
}
```

## 性能优化

### 字段选择
```http
GET /users?fields=id,name,email
```

### 关联资源包含
```http
GET /users/123?include=orders,profile
```

### 缓存控制
```http
# 客户端缓存
GET /users/123
If-None-Match: "686897696a7c876b7e"

# 服务器响应
HTTP/1.1 304 Not Modified
ETag: "686897696a7c876b7e"
```

## 安全考虑

### HTTPS 强制
```nginx
# Nginx 配置
server {
    listen 80;
    server_name api.example.com;
    return 301 https://$server_name$request_uri;
}
```

### CORS 配置
```http
Access-Control-Allow-Origin: https://example.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Allow-Credentials: true
```

### 速率限制
```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1402425459
```

## 文档规范

### OpenAPI/Swagger
```yaml
openapi: 3.0.0
info:
  title: User API
  version: 1.0.0
  description: User management API

paths:
  /users:
    get:
      summary: Get all users
      parameters:
        - in: query
          name: page
          schema:
            type: integer
          description: Page number
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserList'
```

### API Blueprint
```markdown
# Group Users

## User Collection [/users]

### Get Users [GET]

+ Response 200 (application/json)
  + Attributes (array[User])
```

## 测试策略

### 单元测试
```javascript
// 使用 Jest 测试 API
test('GET /users should return user list', async () => {
  const response = await request(app).get('/users')
  expect(response.status).toBe(200)
  expect(response.body).toHaveProperty('data')
  expect(Array.isArray(response.body.data)).toBe(true)
})
```

### 集成测试
```javascript
test('POST /users should create new user', async () => {
  const userData = {
    name: 'John Doe',
    email: 'john@example.com'
  }
  
  const response = await request(app)
    .post('/users')
    .send(userData)
  
  expect(response.status).toBe(201)
  expect(response.body.data).toHaveProperty('id')
})
```

## 监控和日志

### 访问日志
```log
2023-01-01T00:00:00.000Z GET /users 200 45ms
2023-01-01T00:00:01.000Z POST /users 201 120ms
2023-01-01T00:00:02.000Z GET /users/123 404 15ms
```

### 性能监控
```json
{
  "endpoint": "/users",
  "method": "GET",
  "response_time": 45,
  "status_code": 200,
  "timestamp": "2023-01-01T00:00:00Z"
}
```

## 最佳实践总结

### ✅ 应该做的
```markdown
1. 使用名词复数形式命名资源
2. 正确使用HTTP方法和状态码
3. 提供一致的错误响应格式
4. 实现版本管理策略
5. 提供详细的API文档
6. 实施适当的身份验证和授权
7. 设计合理的分页机制
8. 支持字段选择和关联包含
```

### ❌ 避免做的
```markdown
1. 在URL中使用动词
2. 使用非标准的状态码
3. 返回HTML错误页面
4. 暴露敏感信息在错误响应中
5. 创建过深的嵌套资源
6. 忽略版本管理
7. 缺乏适当的限流措施
8. 不提供API文档
```

## 工具推荐

### 开发工具
```markdown
- **Postman**: API测试和文档
- **Swagger/OpenAPI**: API规范
- **Insomnia**: API开发客户端
- **HTTPie**: 命令行HTTP客户端
```

### 监控工具
```markdown
- **Prometheus**: 指标收集
- **Grafana**: 数据可视化
- **ELK Stack**: 日志管理
- **New Relic**: APM监控
```

### 测试工具
```markdown
- **Jest/Mocha**: 测试框架
- **Supertest**: HTTP断言库
- **Artillery**: 负载测试
- **K6**: 性能测试
```

> 重要：RESTful API设计不仅仅是技术实现，更是关于创建一致、可预测且易于使用的接口。始终从客户端开发者的角度考虑API设计。

***
*相关阅读：./graphql-vs-rest.md | ./api-security.md | ./microservice-communication.md*