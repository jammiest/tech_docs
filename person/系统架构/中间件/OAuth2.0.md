### OAuth 2.0 架构概述

OAuth 2.0 是一种授权框架，允许第三方应用在用户授权下有限访问其受保护的资源，而无需共享用户的凭据（如密码）。它广泛应用于现代 Web 和移动应用的身份验证与授权场景。

#### 核心角色
OAuth 2.0 涉及以下四个关键角色：
1. **资源所有者（Resource Owner）**：通常是终端用户，拥有受保护资源的访问权限。
2. **客户端（Client）**：第三方应用，请求访问受保护资源。
3. **授权服务器（Authorization Server）**：验证用户身份并颁发访问令牌。
4. **资源服务器（Resource Server）**：托管受保护资源，并根据访问令牌提供访问。

#### 授权流程
OAuth 2.0 定义了多种授权类型（Grant Types），常见的有：
- **授权码模式（Authorization Code Grant）**：适用于 Web 应用，通过重定向和代码交换令牌。
- **隐式模式（Implicit Grant）**：适用于单页应用（SPA），直接返回访问令牌。
- **密码模式（Resource Owner Password Credentials Grant）**：用户直接提供凭据给客户端（不推荐用于第三方应用）。
- **客户端凭证模式（Client Credentials Grant）**：适用于机器对机器认证。

以下以最安全的授权码模式为例，说明其流程：

1. **用户发起授权请求**：客户端将用户重定向至授权服务器的授权端点（如 `/authorize`），附带参数：
   - `client_id`：客户端标识符。
   - `response_type`：设置为 `code`。
   - `redirect_uri`：授权成功后重定向的 URI。
   - `scope`：请求的权限范围。
   - `state`：随机值用于防跨站请求伪造（CSRF）。

   示例请求 URL：
   ```
   https://auth-server.com/authorize?client_id=123&response_type=code&redirect_uri=https://client.com/callback&scope=read&state=xyz
   ```

2. **用户认证与授权**：授权服务器验证用户身份（例如通过登录页面），并请求用户授权客户端的访问权限。

3. **颁发授权码**：用户同意后，授权服务器将用户重定向回 `redirect_uri`，并附加授权码（如 `?code=AUTH_CODE&state=xyz`）。

4. **交换访问令牌**：客户端使用授权码向授权服务器的令牌端点（如 `/token`）请求访问令牌，通过 POST 请求发送：
   - `grant_type=authorization_code`
   - `code=AUTH_CODE`
   - `redirect_uri`（必须与之前一致）
   - `client_id` 和 `client_secret`（用于客户端认证）

   示例请求体：
   ```json
   {
     "grant_type": "authorization_code",
     "code": "AUTH_CODE",
     "redirect_uri": "https://client.com/callback",
     "client_id": "123",
     "client_secret": "secret"
   }
   ```

5. **颁发令牌**：授权服务器验证请求后，返回访问令牌（和可选刷新令牌）：
   ```json
   {
     "access_token": "ACCESS_TOKEN",
     "token_type": "Bearer",
     "expires_in": 3600,
     "refresh_token": "REFRESH_TOKEN"
   }
   ```

6. **访问资源**：客户端使用访问令牌访问资源服务器，通常在 HTTP 头中携带：
   ```
   Authorization: Bearer ACCESS_TOKEN
   ```

#### 安全考虑
- 使用 HTTPS 保护所有通信。
- `state` 参数防止 CSRF 攻击。
- 令牌应设置合理有效期，刷新令牌用于续期。
- 避免在 URL 中传递敏感信息（如隐式模式）。

#### 数学表示（令牌有效期计算）
访问令牌的生命周期通常由 `expires_in` 字段定义。设当前时间为 $$t_0$$，令牌过期时间为：
$$
t_{\text{expire}} = t_0 + \Delta t
$$
其中 $$\Delta t$$ 是 `expires_in` 的值（以秒为单位）。客户端需在 $$t_{\text{expire}}$$ 前使用刷新令牌获取新访问令牌。

OAuth 2.0 是构建安全、可扩展授权系统的基石，需结合具体场景（如 OpenID Connect 扩展）实现完整身份验证。