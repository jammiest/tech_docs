### JWT（JSON Web Token）简介

JWT（JSON Web Token）是一种开放标准（RFC 7519），用于在各方之间安全地传输信息作为 JSON 对象。它通常用于身份验证和授权场景，例如在用户登录后生成令牌，客户端在后续请求中携带该令牌以访问受保护的资源。

---

### JWT 结构

JWT 由三部分组成，用点（`.`）分隔：
- **Header（头部）**
- **Payload（载荷）**
- **Signature（签名）**

格式为：`Header.Payload.Signature`

#### 1. Header
头部通常包含两部分：
- `alg`：签名算法，如 HMAC SHA256 或 RSA。
- `typ`：令牌类型，固定为 "JWT"。

示例：
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```
Base64Url 编码后形成第一部分。

#### 2. Payload
载荷包含声明（claims），声明是关于实体（通常是用户）和其他数据的语句。声明分为三类：
- **Registered claims**：预定义声明，如 `iss`（签发者）、`exp`（过期时间）、`sub`（主题）等。
- **Public claims**：自定义声明，但应避免冲突。
- **Private claims**：自定义声明，用于在同意方之间共享信息。

示例：
```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```
Base64Url 编码后形成第二部分。

#### 3. Signature
签名用于验证令牌的完整性和真实性。签名通过以下方式生成：
$$ \text{Signature} = \text{HMAC-SHA256}(\text{base64UrlEncode(header)} + "." + \text{base64UrlEncode(payload)}, \text{secret}) $$
（对于 HMAC 算法）或使用私钥进行 RSA 签名。

---

### JWT 工作流程

1. **用户登录**：客户端发送凭据（如用户名和密码）到服务器。
2. **服务器验证**：服务器验证凭据，若有效则生成 JWT 并返回给客户端。
3. **客户端存储**：客户端（如浏览器）存储 JWT（通常在 localStorage 或 cookie 中）。
4. **后续请求**：客户端在请求头（如 `Authorization: Bearer <token>`）中携带 JWT。
5. **服务器验证**：服务器验证 JWT 的签名和有效期，若有效则处理请求。

---

### 优点与缺点

#### 优点：
- **无状态**：服务器不需要存储会话信息，所有必要信息都在令牌中。
- **跨域友好**：适用于分布式系统和微服务架构。
- **灵活性**：可包含自定义声明。

#### 缺点：
- **令牌大小**：载荷较大时，令牌长度可能增加。
- **无法撤销**：一旦签发，在过期前无法强制失效（除非使用黑名单或短期有效期）。
- **安全风险**：若密钥泄露或算法弱，可能导致安全问题。

---

### 代码示例（Node.js）

以下是一个使用 `jsonwebtoken` 库生成和验证 JWT 的示例：

```javascript
const jwt = require('jsonwebtoken');

// 生成 JWT
const payload = { userId: 123, username: 'john' };
const secret = 'your-secret-key';
const token = jwt.sign(payload, secret, { expiresIn: '1h' });
console.log('Generated Token:', token);

// 验证 JWT
jwt.verify(token, secret, (err, decoded) => {
  if (err) {
    console.error('Invalid token');
  } else {
    console.log('Decoded Payload:', decoded);
  }
});
```

---

### 安全建议

1. **使用强密钥**：对于 HMAC，使用足够长的随机密钥；对于 RSA，使用足够强度的私钥。
2. **设置短期有效期**：通过 `exp` 声明减少令牌被滥用的风险。
3. **HTTPS 传输**：避免令牌在传输过程中被窃取。
4. **避免敏感信息**：载荷仅包含必要信息，因为 Base64 可解码。

---

JWT 是现代 Web 开发中广泛使用的身份验证机制，正确实施可以提升系统的安全性和可扩展性。