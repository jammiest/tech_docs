# Web 安全最佳实践

Web 安全是保护用户数据和应用程序完整性的关键。本指南将全面介绍现代 Web 应用的安全策略、防护措施和最佳实践。

## 安全架构设计

### 1. 安全层级模型

```javascript
// security-layers.js
class SecurityLayers {
  constructor() {
    this.layers = new Map();
    this.setupDefaultLayers();
  }

  setupDefaultLayers() {
    // 1. 网络层安全
    this.addLayer('network', {
      enabled: true,
      measures: [
        'HTTPS_ENFORCEMENT',
        'CSP_HEADERS',
        'RATE_LIMITING',
        'WAF_INTEGRATION'
      ]
    });

    // 2. 应用层安全
    this.addLayer('application', {
      enabled: true,
      measures: [
        'INPUT_VALIDATION',
        'OUTPUT_ENCODING',
        'AUTHENTICATION',
        'AUTHORIZATION'
      ]
    });

    // 3. 数据层安全
    this.addLayer('data', {
      enabled: true,
      measures: [
        'ENCRYPTION_AT_REST',
        'ENCRYPTION_IN_TRANSIT',
        'DATA_SANITIZATION',
        'ACCESS_CONTROLS'
      ]
    });

    // 4. 客户端安全
    this.addLayer('client', {
      enabled: true,
      measures: [
        'CSP',
        'XSS_PROTECTION',
        'CSRF_TOKENS',
        'SECURE_COOKIES'
      ]
    });
  }

  addLayer(name, config) {
    this.layers.set(name, {
      ...config,
      lastUpdated: Date.now()
    });
  }

  getSecurityReport() {
    const report = {};
    let totalMeasures = 0;
    let implementedMeasures = 0;

    for (const [layerName, layerConfig] of this.layers) {
      report[layerName] = {
        enabled: layerConfig.enabled,
        measures: layerConfig.measures,
        coverage: layerConfig.measures.length
      };
      
      totalMeasures += layerConfig.measures.length;
      if (layerConfig.enabled) {
        implementedMeasures += layerConfig.measures.length;
      }
    }

    return {
      layers: report,
      overallCoverage: Math.round((implementedMeasures / totalMeasures) * 100),
      totalMeasures,
      implementedMeasures
    };
  }

  validateSecurity() {
    const report = this.getSecurityReport();
    const issues = [];

    // 检查必需的安全措施
    if (!this.layers.get('network').measures.includes('HTTPS_ENFORCEMENT')) {
      issues.push('Missing HTTPS enforcement');
    }

    if (!this.layers.get('application').measures.includes('INPUT_VALIDATION')) {
      issues.push('Missing input validation');
    }

    // 检查是否启用关键层
    if (!this.layers.get('network').enabled) {
      issues.push('Network security layer disabled');
    }

    return {
      valid: issues.length === 0,
      issues,
      report
    };
  }
}

// 全局安全配置
export const securityLayers = new SecurityLayers();
```

### 2. 安全头配置

```javascript
// security-headers.js
const securityHeaders = {
  // 内容安全策略
  'Content-Security-Policy': [
    "default-src 'self'",
    "script-src 'self' 'unsafe-inline' https://cdn.example.com",
    "style-src 'self' 'unsafe-inline'",
    "img-src 'self' data: https:",
    "font-src 'self'",
    "connect-src 'self' https://api.example.com",
    "frame-ancestors 'none'",
    "form-action 'self'",
    "base-uri 'self'"
  ].join('; '),

  // XSS 保护
  'X-XSS-Protection': '1; mode=block',

  // 防止 MIME 类型嗅探
  'X-Content-Type-Options': 'nosniff',

  // 防止点击劫持
  'X-Frame-Options': 'DENY',

  // 引用策略
  'Referrer-Policy': 'strict-origin-when-cross-origin',

  // 权限策略
  'Permissions-Policy': [
    'geolocation=()',
    'microphone=()',
    'camera=()',
    'payment=()'
  ].join(', '),

  // 期望-CT
  'Expect-CT': 'max-age=86400, enforce'
};

// Express.js 中间件
export const securityHeadersMiddleware = (req, res, next) => {
  Object.entries(securityHeaders).forEach(([header, value]) => {
    res.setHeader(header, value);
  });
  next();
};

// Next.js 配置
export const nextSecurityHeaders = async () => {
  return [
    {
      source: '/(.*)',
      headers: Object.entries(securityHeaders).map(([key, value]) => ({
        key,
        value
      }))
    }
  ];
};

// Nginx 配置
export const nginxSecurityHeaders = `
add_header Content-Security-Policy "${securityHeaders['Content-Security-Policy']}";
add_header X-XSS-Protection "${securityHeaders['X-XSS-Protection']}";
add_header X-Content-Type-Options "${securityHeaders['X-Content-Type-Options']}";
add_header X-Frame-Options "${securityHeaders['X-Frame-Options']}";
add_header Referrer-Policy "${securityHeaders['Referrer-Policy']}";
add_header Permissions-Policy "${securityHeaders['Permissions-Policy']}";
`;
```

## 身份认证与授权

### 1. JWT 认证系统

```javascript
// auth/jwt-manager.js
import jwt from 'jsonwebtoken';
import crypto from 'crypto';

class JWTManager {
  constructor() {
    this.secret = process.env.JWT_SECRET;
    this.issuer = process.env.JWT_ISSUER || 'my-app';
    this.audience = process.env.JWT_AUDIENCE || 'my-app-users';
    this.refreshTokens = new Map();
  }

  // 生成访问令牌
  generateAccessToken(payload, options = {}) {
    const config = {
      issuer: this.issuer,
      audience: this.audience,
      expiresIn: '15m',
      ...options
    };

    return jwt.sign(payload, this.secret, config);
  }

  // 生成刷新令牌
  generateRefreshToken(userId) {
    const token = crypto.randomBytes(40).toString('hex');
    const expiresAt = Date.now() + 7 * 24 * 60 * 60 * 1000; // 7天

    this.refreshTokens.set(token, {
      userId,
      expiresAt,
      createdAt: Date.now()
    });

    return token;
  }

  // 验证访问令牌
  verifyAccessToken(token) {
    try {
      return jwt.verify(token, this.secret, {
        issuer: this.issuer,
        audience: this.audience
      });
    } catch (error) {
      throw new Error('Invalid access token');
    }
  }

  // 验证刷新令牌
  verifyRefreshToken(token) {
    const storedToken = this.refreshTokens.get(token);
    
    if (!storedToken) {
      throw new Error('Invalid refresh token');
    }

    if (storedToken.expiresAt < Date.now()) {
      this.refreshTokens.delete(token);
      throw new Error('Refresh token expired');
    }

    return storedToken;
  }

  // 刷新令牌对
  refreshTokenPair(refreshToken) {
    const storedToken = this.verifyRefreshToken(refreshToken);
    
    // 生成新的访问令牌
    const newAccessToken = this.generateAccessToken({
      userId: storedToken.userId
    });

    // 生成新的刷新令牌（可选刷新令牌轮换）
    const newRefreshToken = this.generateRefreshToken(storedToken.userId);
    
    // 删除旧的刷新令牌
    this.refreshTokens.delete(refreshToken);

    return {
      accessToken: newAccessToken,
      refreshToken: newRefreshToken,
      expiresIn: 900 // 15分钟
    };
  }

  // 撤销令牌
  revokeToken(token, type = 'access') {
    if (type === 'refresh') {
      this.refreshTokens.delete(token);
    }
    // 访问令牌无法撤销，只能等待过期
  }

  // 清理过期的刷新令牌
  cleanupExpiredTokens() {
    for (const [token, data] of this.refreshTokens.entries()) {
      if (data.expiresAt < Date.now()) {
        this.refreshTokens.delete(token);
      }
    }
  }

  // 安全配置检查
  validateConfig() {
    const issues = [];
    
    if (!this.secret || this.secret === 'default-secret') {
      issues.push('JWT secret is not properly configured');
    }

    if (this.secret && this.secret.length < 32) {
      issues.push('JWT secret is too short (min 32 characters)');
    }

    return {
      valid: issues.length === 0,
      issues
    };
  }
}

export const jwtManager = new JWTManager();

// Express.js 中间件
export const authenticateToken = (req, res, next) => {
  const authHeader = req.headers['authorization'];
  const token = authHeader && authHeader.split(' ')[1];

  if (!token) {
    return res.status(401).json({ error: 'Access token required' });
  }

  try {
    const decoded = jwtManager.verifyAccessToken(token);
    req.user = decoded;
    next();
  } catch (error) {
    return res.status(403).json({ error: 'Invalid access token' });
  }
};

// 速率限制中间件
export const rateLimitMiddleware = (windowMs = 900000, maxRequests = 100) => {
  const requests = new Map();

  return (req, res, next) => {
    const ip = req.ip || req.connection.remoteAddress;
    const now = Date.now();
    const windowStart = now - windowMs;

    // 清理过期的请求记录
    for (const [key, timestamps] of requests.entries()) {
      requests.set(key, timestamps.filter(time => time > windowStart));
    }

    const userRequests = requests.get(ip) || [];
    
    if (userRequests.length >= maxRequests) {
      return res.status(429).json({
        error: 'Too many requests',
        retryAfter: Math.ceil((userRequests[0] + windowMs - now) / 1000)
      });
    }

    userRequests.push(now);
    requests.set(ip, userRequests);

    // 设置速率限制头
    res.setHeader('X-RateLimit-Limit', maxRequests);
    res.setHeader('X-RateLimit-Remaining', maxRequests - userRequests.length);
    res.setHeader('X-RateLimit-Reset', Math.ceil((now + windowMs) / 1000));

    next();
  };
};
```

### 2. OAuth 2.0 集成

```javascript
// auth/oauth-manager.js
import crypto from 'crypto';

class OAuthManager {
  constructor() {
    this.clients = new Map();
    this.authCodes = new Map();
    this.setupDemoClients();
  }

  setupDemoClients() {
    // 演示客户端配置
    this.registerClient({
      clientId: 'web-app',
      clientSecret: crypto.randomBytes(32).toString('hex'),
      redirectUris: ['https://app.example.com/auth/callback'],
      grants: ['authorization_code', 'refresh_token'],
      scopes: ['profile', 'email', 'openid']
    });
  }

  registerClient(clientConfig) {
    const client = {
      clientId: clientConfig.clientId,
      clientSecret: clientConfig.clientSecret,
      redirectUris: clientConfig.redirectUris,
      grants: clientConfig.grants,
      scopes: clientConfig.scopes,
      createdAt: Date.now()
    };

    this.clients.set(client.clientId, client);
    return client;
  }

  validateClient(clientId, clientSecret, redirectUri) {
    const client = this.clients.get(clientId);
    
    if (!client) {
      throw new Error('Invalid client ID');
    }

    if (clientSecret && client.clientSecret !== clientSecret) {
      throw new Error('Invalid client secret');
    }

    if (redirectUri && !client.redirectUris.includes(redirectUri)) {
      throw new Error('Invalid redirect URI');
    }

    return client;
  }

  generateAuthCode(clientId, userId, scopes, redirectUri) {
    const code = crypto.randomBytes(20).toString('hex');
    const expiresAt = Date.now() + 10 * 60 * 1000; // 10分钟

    this.authCodes.set(code, {
      clientId,
      userId,
      scopes,
      redirectUri,
      expiresAt,
      createdAt: Date.now()
    });

    return code;
  }

  validateAuthCode(code, clientId, redirectUri) {
    const authCode = this.authCodes.get(code);
    
    if (!authCode) {
      throw new Error('Invalid authorization code');
    }

    if (authCode.expiresAt < Date.now()) {
      this.authCodes.delete(code);
      throw new Error('Authorization code expired');
    }

    if (authCode.clientId !== clientId) {
      throw new Error('Client mismatch');
    }

    if (redirectUri && authCode.redirectUri !== redirectUri) {
      throw new Error('Redirect URI mismatch');
    }

    return authCode;
  }

  exchangeCodeForToken(code, clientId, clientSecret) {
    const client = this.validateClient(clientId, clientSecret);
    const authCode = this.validateAuthCode(code, clientId);

    // 删除已使用的授权码
    this.authCodes.delete(code);

    // 生成访问令牌和刷新令牌
    const accessToken = jwtManager.generateAccessToken({
      userId: authCode.userId,
      scope: authCode.scopes
    });

    const refreshToken = jwtManager.generateRefreshToken(authCode.userId);

    return {
      access_token: accessToken,
      refresh_token: refreshToken,
      token_type: 'Bearer',
      expires_in: 900,
      scope: authCode.scopes.join(' ')
    };
  }

  // OAuth 2.0 授权端点
  async handleAuthorizationRequest(req, res) {
    const {
      response_type,
      client_id,
      redirect_uri,
      scope,
      state
    } = req.query;

    try {
      // 验证客户端
      const client = this.validateClient(client_id, null, redirect_uri);

      if (response_type !== 'code') {
        throw new Error('Unsupported response type');
      }

      // 检查用户认证（这里需要实现用户会话检查）
      if (!req.session.userId) {
        // 重定向到登录页面
        return res.redirect(`/login?redirect=${encodeURIComponent(req.originalUrl)}`);
      }

      // 生成授权码
      const scopes = (scope || '').split(' ').filter(s => s);
      const code = this.generateAuthCode(client_id, req.session.userId, scopes, redirect_uri);

      // 重定向回客户端
      const redirectUrl = new URL(redirect_uri);
      redirectUrl.searchParams.set('code', code);
      if (state) redirectUrl.searchParams.set('state', state);

      return res.redirect(redirectUrl.toString());
    } catch (error) {
      // 错误重定向
      const errorRedirect = new URL(redirect_uri);
      errorRedirect.searchParams.set('error', 'invalid_request');
      errorRedirect.searchParams.set('error_description', error.message);
      if (state) errorRedirect.searchParams.set('state', state);

      return res.redirect(errorRedirect.toString());
    }
  }
}

export const oauthManager = new OAuthManager();
```

## 输入验证与消毒

### 1. 综合验证系统

```javascript
// security/validation.js
import validator from 'validator';
import xss from 'xss';

class ValidationSystem {
  constructor() {
    this.rules = new Map();
    this.setupDefaultRules();
  }

  setupDefaultRules() {
    // 电子邮件验证
    this.addRule('email', (value) => ({
      valid: validator.isEmail(value),
      message: 'Invalid email address',
      sanitize: validator.normalizeEmail(value)
    }));

    // URL 验证
    this.addRule('url', (value) => ({
      valid: validator.isURL(value, { require_protocol: true }),
      message: 'Invalid URL',
      sanitize: validator.escape(value)
    }));

    // XSS 防护
    this.addRule('xss', (value) => ({
      valid: true, // 总是通过验证，进行消毒
      message: 'XSS attempt detected',
      sanitize: xss(value, {
        whiteList: {},
        stripIgnoreTag: true,
        stripIgnoreTagBody: ['script']
      })
    }));

    // SQL 注入防护
    this.addRule('sql', (value) => ({
      valid: !this.detectSQLInjection(value),
      message: 'SQL injection attempt detected',
      sanitize: validator.escape(value)
    }));

    // 数字验证
    this.addRule('number', (value, options = {}) => {
      const num = Number(value);
      const min = options.min || -Infinity;
      const max = options.max || Infinity;
      
      return {
        valid: !isNaN(num) && num >= min && num <= max,
        message: `Must be a number between ${min} and ${max}`,
        sanitize: num
      };
    });
  }

  addRule(name, validatorFn) {
    this.rules.set(name, validatorFn);
  }

  validate(value, rules, options = {}) {
    const results = {
      valid: true,
      errors: [],
      sanitized: value
    };

    for (const rule of rules) {
      let ruleName, ruleOptions;
      
      if (typeof rule === 'string') {
        ruleName = rule;
        ruleOptions = {};
      } else {
        ruleName = rule.name;
        ruleOptions = rule.options || {};
      }

      const validatorFn = this.rules.get(ruleName);
      if (!validatorFn) continue;

      const result = validatorFn(value, ruleOptions);
      
      if (!result.valid) {
        results.valid = false;
        results.errors.push({
          rule: ruleName,
          message: result.message
        });
      }

      if (result.sanitize !== undefined) {
        results.sanitized = result.sanitize;
      }
    }

    return results;
  }

  detectSQLInjection(value) {
    if (typeof value !== 'string') return false;
    
    const sqlKeywords = [
      'union', 'select', 'insert', 'update', 'delete', 'drop', 
      'exec', 'execute', 'where', 'having', '--', '/*', '*/', ';'
    ];
    
    const lowerValue = value.toLowerCase();
    return sqlKeywords.some(keyword => lowerValue.includes(keyword));
  }

  // 批量验证
  validateObject(obj, schema) {
    const results = {};
    const errors = [];
    let allValid = true;

    for (const [field, rules] of Object.entries(schema)) {
      const value = obj[field];
      const result = this.validate(value, rules);
      
      results[field] = result.sanitized;
      
      if (!result.valid) {
        allValid = false;
        errors.push({
          field,
          errors: result.errors
        });
      }
    }

    return {
      valid: allValid,
      errors,
      sanitized: results
    };
  }
}

export const validationSystem = new ValidationSystem();

// Express.js 中间件
export const validateRequest = (schema) => {
  return (req, res, next) => {
    const data = { ...req.body, ...req.query, ...req.params };
    const result = validationSystem.validateObject(data, schema);

    if (!result.valid) {
      return res.status(400).json({
        error: 'Validation failed',
        details: result.errors
      });
    }

    // 使用消毒后的数据
    req.sanitizedData = result.sanitized;
    next();
  };
};

// 使用示例
export const userValidationSchema = {
  email: ['email', 'xss'],
  password: [
    { name: 'length', options: { min: 8 } },
    'xss'
  ],
  age: [
    { name: 'number', options: { min: 0, max: 120 } }
  ]
};
```

### 2. CSP 违规报告

```javascript
// security/csp-monitor.js
class CSPMonitor {
  constructor() {
    this.violations = new Map();
    this.reportQueue = [];
    this.isReporting = false;
  }

  // 处理 CSP 违规报告
  async handleViolationReport(req, res) {
    try {
      const report = req.body;
      await this.processViolation(report);
      
      res.status(200).send('OK');
    } catch (error) {
      console.error('CSP violation processing error:', error);
      res.status(400).send('Bad Request');
    }
  }

  // 处理违规
  async processViolation(report) {
    const violation = this.parseViolation(report);
    
    // 记录违规
    this.recordViolation(violation);
    
    // 检查是否达到阈值
    if (this.shouldAlert(violation)) {
      await this.sendAlert(violation);
    }
    
    // 加入报告队列
    this.reportQueue.push(violation);
    this.processReportQueue();
  }

  parseViolation(report) {
    const { 'csp-report': cspReport } = report;
    
    return {
      documentUri: cspReport.document-uri,
      violatedDirective: cspReport.violated-directive,
      effectiveDirective: cspReport.effective-directive,
      originalPolicy: cspReport.original-policy,
      blockedUri: cspReport.blocked-uri,
      sourceFile: cspReport.source-file,
      lineNumber: cspReport.line-number,
      columnNumber: cspReport.column-number,
      statusCode: cspReport.status-code,
      referrer: cspReport.referrer,
      timestamp: Date.now()
    };
  }

  recordViolation(violation) {
    const key = this.getViolationKey(violation);
    const existing = this.violations.get(key) || { count: 0, firstSeen: Date.now() };
    
    this.violations.set(key, {
      ...existing,
      count: existing.count + 1,
      lastSeen: Date.now(),
      examples: [...(existing.examples || []).slice(0, 4), violation]
    });
  }

  getViolationKey(violation) {
    return `${violation.effectiveDirective}:${violation.blockedUri}`;
  }

  shouldAlert(violation) {
    const key = this.getViolationKey(violation);
    const data = this.violations.get(key);
    
    return data && data.count >= 5; // 5次以上触发警报
  }

  async sendAlert(violation) {
    // 发送警报到监控系统
    console.log('CSP Violation Alert:', violation);
    
    // 实际实现中应该发送到监控平台
    await fetch(process.env.CSP_ALERT_WEBHOOK, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        title: 'CSP Violation Alert',
        violation: violation,
        count: this.violations.get(this.getViolationKey(violation)).count
      })
    });
  }

  async processReportQueue() {
    if (this.isReporting || this.reportQueue.length === 0) return;
    
    this.isReporting = true;
    
    while (this.reportQueue.length > 0) {
      const violation = this.reportQueue.shift();
      
      try {
        await this.reportToAnalytics(violation);
      } catch (error) {
        console.error('Failed to report CSP violation:', error);
        this.reportQueue.unshift(violation);
        break;
      }
    }
    
    this.isReporting = false;
  }

  async reportToAnalytics(violation) {
    // 上报到分析平台
    await fetch('/api/analytics/csp-violations', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(violation)
    });
  }

  // 获取违规统计
  getViolationStats() {
    const stats = {};
    let totalViolations = 0;
    
    for (const [key, data] of this.violations.entries()) {
      stats[key] = data;
      totalViolations += data.count;
    }
    
    return {
      totalViolations,
      uniqueViolations: this.violations.size,
      violations: stats
    };
  }
}

export const cspMonitor = new CSPMonitor();

// Express.js 路由
export const cspReportRoute = (req, res) => {
  cspMonitor.handleViolationReport(req, res);
};
```

## 数据安全与加密

### 1. 加密服务

```javascript
// security/encryption.js
import crypto from 'crypto';

class EncryptionService {
  constructor() {
    this.algorithm = 'aes-256-gcm';
    this.key = Buffer.from(process.env.ENCRYPTION_KEY, 'hex');
    this.ivLength = 16;
    this.authTagLength = 16;
  }

  // 加密数据
  encrypt(data) {
    const iv = crypto.randomBytes(this.ivLength);
    const cipher = crypto.createCipheriv(this.algorithm, this.key, iv);
    
    let encrypted = cipher.update(data, 'utf8');
    encrypted = Buffer.concat([encrypted, cipher.final()]);
    
    const authTag = cipher.getAuthTag();
    
    return {
      iv: iv.toString('hex'),
      authTag: authTag.toString('hex'),
      content: encrypted.toString('hex')
    };
  }

  // 解密数据
  decrypt(encryptedData) {
    const { iv, authTag, content } = encryptedData;
    
    const decipher = crypto.createDecipheriv(
      this.algorithm, 
      this.key, 
      Buffer.from(iv, 'hex')
    );
    
    decipher.setAuthTag(Buffer.from(authTag, 'hex'));
    
    let decrypted = decipher.update(Buffer.from(content, 'hex'));
    decrypted = Buffer.concat([decrypted, decipher.final()]);
    
    return decrypted.toString('utf8');
  }

  // 哈希数据
  hash(data, salt = null) {
    const usedSalt = salt || crypto.randomBytes(16).toString('hex');
    const hash = crypto
      .createHash('sha256')
      .update(data + usedSalt)
      .digest('hex');
    
    return {
      hash,
      salt: usedSalt
    };
  }

  // 验证哈希
  verifyHash(data, hash, salt) {
    const newHash = this.hash(data, salt).hash;
    return crypto.timingSafeEqual(
      Buffer.from(newHash),
      Buffer.from(hash)
    );
  }

  // 生成安全随机数
  generateRandomBytes(length) {
    return crypto.randomBytes(length);
  }

  // 生成安全令牌
  generateSecureToken(length = 32) {
    return crypto.randomBytes(length).toString('hex');
  }

  // 密码哈希（使用 bcrypt）
  async hashPassword(password) {
    const saltRounds = 12;
    return await bcrypt.hash(password, saltRounds);
  }

  async verifyPassword(password, hash) {
    return await bcrypt.compare(password, hash);
  }

  // 加密配置文件验证
  validateConfig() {
    const issues = [];
    
    if (!this.key || this.key.length !== 32) {
      issues.push('Encryption key must be 32 bytes (256 bits)');
    }
    
    if (process.env.ENCRYPTION_KEY === 'default-key') {
      issues.push('Default encryption key detected - please change it');
    }
    
    return {
      valid: issues.length === 0,
      issues
    };
  }
}

export const encryptionService = new EncryptionService();

// 安全存储中间件
export const encryptSensitiveData = (req, res, next) => {
  const sensitiveFields = ['password', 'creditCard', 'ssn', 'token'];
  
  if (req.body) {
    for (const field of sensitiveFields) {
      if (req.body[field]) {
        req.body[field] = encryptionService.encrypt(req.body[field]);
      }
    }
  }
  
  next();
};
```

### 2. 安全会话管理

```javascript
// security/session-manager.js
import crypto from 'crypto';

class SessionManager {
  constructor() {
    this.sessions = new Map();
    this.cleanupInterval = setInterval(() => this.cleanupSessions(), 3600000); // 每小时清理一次
  }

  // 创建新会话
  createSession(userId, userAgent, ipAddress) {
    const sessionId = this.generateSessionId();
    const expiresAt = Date.now() + 24 * 60 * 60 * 1000; // 24小时
    
    const session = {
      sessionId,
      userId,
      userAgent,
      ipAddress,
      createdAt: Date.now(),
      expiresAt,
      lastActive: Date.now(),
      isValid: true
    };
    
    this.sessions.set(sessionId, session);
    return sessionId;
  }

  // 验证会话
  validateSession(sessionId, userAgent, ipAddress) {
    const session = this.sessions.get(sessionId);
    
    if (!session || !session.isValid) {
      throw new Error('Invalid session');
    }
    
    if (session.expiresAt < Date.now()) {
      session.isValid = false;
      throw new Error('Session expired');
    }
    
    // 检查用户代理和IP是否匹配
    if (session.userAgent !== userAgent || session.ipAddress !== ipAddress) {
      this.invalidateSession(sessionId);
      throw new Error('Session hijacking detected');
    }
    
    // 更新最后活动时间
    session.lastActive = Date.now();
    
    return session;
  }

  // 使会话失效
  invalidateSession(sessionId) {
    const session = this.sessions.get(sessionId);
    if (session) {
      session.isValid = false;
    }
  }

  // 使所有用户会话失效
  invalidateAllUserSessions(userId) {
    for (const [sessionId, session] of this.sessions.entries()) {
      if (session.userId === userId) {
        session.isValid = false;
      }
    }
  }

  // 生成安全会话ID
  generateSessionId() {
    return crypto.randomBytes(24).toString('hex');
  }

  // 清理过期会话
  cleanupSessions() {
    const now = Date.now();
    let cleanedCount = 0;
    
    for (const [sessionId, session] of this.sessions.entries()) {
      if (session.expiresAt < now || !session.isValid) {
        this.sessions.delete(sessionId);
        cleanedCount++;
      }
    }
    
    console.log(`Cleaned up ${cleanedCount} expired sessions`);
  }

  // 获取用户活动会话
  getUserSessions(userId) {
    const userSessions = [];
    
    for (const session of this.sessions.values()) {
      if (session.userId === userId && session.isValid) {
        userSessions.push({
          sessionId: session.sessionId,
          userAgent: session.userAgent,
          ipAddress: session.ipAddress,
          createdAt: session.createdAt,
          lastActive: session.lastActive,
          expiresAt: session.expiresAt
        });
      }
    }
    
    return userSessions;
  }

  // 会话安全报告
  getSecurityReport() {
    const totalSessions = this.sessions.size;
    let activeSessions = 0;
    let expiredSessions = 0;
    
    for (const session of this.sessions.values()) {
      if (session.isValid) {
        activeSessions++;
      } else {
        expiredSessions++;
      }
    }
    
    return {
      totalSessions,
      activeSessions,
      expiredSessions,
      cleanupInterval: '1 hour'
    };
  }
}

export const sessionManager = new SessionManager();

// Express.js 会话中间件
export const sessionMiddleware = (req, res, next) => {
  const sessionId = req.cookies?.sessionId;
  const userAgent = req.get('User-Agent');
  const ipAddress = req.ip || req.connection.remoteAddress;
  
  if (sessionId) {
    try {
      const session = sessionManager.validateSession(sessionId, userAgent, ipAddress);
      req.session = session;
    } catch (error) {
      // 清除无效的会话cookie
      res.clearCookie('sessionId');
    }
  }
  
  next();
};
```

## 客户端安全

### 1. CSP 配置生成器

```javascript
// security/csp-generator.js
class CSPGenerator {
  constructor() {
    this.directives = {
      'default-src': ["'self'"],
      'script-src': ["'self'"],
      'style-src': ["'self'", "'unsafe-inline'"],
      'img-src': ["'self'", "data:", "https:"],
      'font-src': ["'self'"],
      'connect-src': ["'self'"],
      'frame-src': ["'none'"],
      'object-src': ["'none'"],
      'base-uri': ["'self'"],
      'form-action': ["'self'"],
      'frame-ancestors': ["'none'"],
      'report-uri': ['/api/csp-report']
    };
  }

  // 添加源到指令
  addSource(directive, source) {
    if (!this.directives[directive]) {
      this.directives[directive] = [];
    }
    
    if (!this.directives[directive].includes(source)) {
      this.directives[directive].push(source);
    }
    
    return this;
  }

  // 移除源从指令
  removeSource(directive, source) {
    if (this.directives[directive]) {
      this.directives[directive] = this.directives[directive].filter(s => s !== source);
    }
    
    return this;
  }

  // 生成 CSP 头
  generateHeader() {
    const headerParts = [];
    
    for (const [directive, sources] of Object.entries(this.directives)) {
      if (sources.length > 0) {
        headerParts.push(`${directive} ${sources.join(' ')}`);
      }
    }
    
    return headerParts.join('; ');
  }

  // 生成 meta 标签
  generateMetaTag() {
    const csp = this.generateHeader();
    return `<meta http-equiv="Content-Security-Policy" content="${csp.replace(/"/g, '&quot;')}">`;
  }

  // 验证 CSP 配置
  validate() {
    const warnings = [];
    
    // 检查不安全的配置
    if (this.directives['script-src'].includes("'unsafe-eval'")) {
      warnings.push('Avoid unsafe-eval in script-src');
    }
    
    if (this.directives['style-src'].includes("'unsafe-inline'")) {
      warnings.push('Consider removing unsafe-inline from style-src');
    }
    
    if (this.directives['object-src'].includes("'self'")) {
      warnings.push('Avoid self in object-src - use none instead');
    }
    
    return {
      valid: warnings.length === 0,
      warnings
    };
  }

  // 为开发环境生成宽松配置
  forDevelopment() {
    return this
      .addSource('connect-src', "'self'")
      .addSource('script-src', "'unsafe-eval'")
      .addSource('style-src', "'unsafe-inline'");
  }

  // 为生产环境生成严格配置
  forProduction() {
    return this
      .removeSource('script-src', "'unsafe-eval'")
      .removeSource('style-src', "'unsafe-inline'")
      .addSource('report-uri', '/api/csp-report');
  }
}

export const cspGenerator = new CSPGenerator();

// 使用示例
export const generateCSP = (environment = 'production') => {
  const generator = new CSPGenerator()
    .addSource('default-src', "'self'")
    .addSource('script-src', 'https://cdn.example.com')
    .addSource('style-src', 'https://fonts.googleapis.com')
    .addSource('font-src', 'https://fonts.gstatic.com')
    .addSource('img-src', 'https://images.example.com');
  
  if (environment === 'development') {
    return generator.forDevelopment().generateHeader();
  } else {
    return generator.forProduction().generateHeader();
  }
};
```

### 2. 安全头扫描器

```javascript
// security/header-scanner.js
class HeaderScanner {
  constructor() {
    this.requiredHeaders = [
      'Content-Security-Policy',
      'X-Content-Type-Options',
      'X-Frame-Options',
      'X-XSS-Protection',
      'Strict-Transport-Security',
      'Referrer-Policy'
    ];
    
    this.headerStandards = {
      'Content-Security-Policy': {
        required: true,
        description: 'Prevents XSS and other code injection attacks',
        recommended: "default-src 'self'; script-src 'self'; object-src 'none'"
      },
      'X-Content-Type-Options': {
        required: true,
        description: 'Prevents MIME type sniffing',
        recommended: 'nosniff'
      },
      'X-Frame-Options': {
        required: true,
        description: 'Prevents clickjacking attacks',
        recommended: 'DENY'
      },
      'X-XSS-Protection': {
        required: true,
        description: 'Enables XSS protection in older browsers',
        recommended: '1; mode=block'
      },
      'Strict-Transport-Security': {
        required: false,
        description: 'Enforces HTTPS connections',
        recommended: 'max-age=31536000; includeSubDomains'
      },
      'Referrer-Policy': {
        required: false,
        description: 'Controls referrer information',
        recommended: 'strict-origin-when-cross-origin'
      }
    };
  }

  // 扫描 URL 的安全头
  async scanUrl(url) {
    try {
      const response = await fetch(url, { method: 'HEAD' });
      const headers = this.extractHeaders(response.headers);
      
      return this.analyzeHeaders(headers);
    } catch (error) {
      return {
        error: `Failed to scan URL: ${error.message}`,
        headers: {}
      };
    }
  }

  // 提取和分析头信息
  extractHeaders(headers) {
    const result = {};
    
    for (const [key, value] of headers.entries()) {
      result[key.toLowerCase()] = value;
    }
    
    return result;
  }

  analyzeHeaders(headers) {
    const results = {
      missing: [],
      present: [],
      issues: [],
      score: 100
    };
    
    for (const [header, standard] of Object.entries(this.headerStandards)) {
      const lowerHeader = header.toLowerCase();
      const headerValue = headers[lowerHeader];
      
      if (headerValue) {
        results.present.push({
          header,
          value: headerValue,
          description: standard.description
        });
        
        // 检查头值是否符合推荐配置
        if (standard.recommended && headerValue !== standard.recommended) {
          results.issues.push({
            header,
            issue: 'Non-recommended value',
            current: headerValue,
            recommended: standard.recommended
          });
          results.score -= 10;
        }
      } else if (standard.required) {
        results.missing.push({
          header,
          description: standard.description,
          recommended: standard.recommended
        });
        results.score -= 15;
      }
    }
    
    return {
      ...results,
      grade: this.getGrade(results.score)
    };
  }

  getGrade(score) {
    if (score >= 90) return 'A';
    if (score >= 75) return 'B';
    if (score >= 60) return 'C';
    if (score >= 40) return 'D';
    return 'F';
  }

  // 生成安全头报告
  generateReport(analysis) {
    const report = [];
    
    report.push('# Security Headers Report\n');
    report.push(`Overall Grade: ${analysis.grade} (Score: ${analysis.score}/100)\n`);
    
    if (analysis.present.length > 0) {
      report.push('\n## Present Headers\n');
      analysis.present.forEach(header => {
        report.push(`- **${header.header}**: ${header.value}`);
      });
    }
    
    if (analysis.missing.length > 0) {
      report.push('\n## Missing Headers\n');
      analysis.missing.forEach(header => {
        report.push(`- ❌ **${header.header}**: ${header.description}`);
        report.push(`  Recommended: \`${header.recommended}\``);
      });
    }
    
    if (analysis.issues.length > 0) {
      report.push('\n## Configuration Issues\n');
      analysis.issues.forEach(issue => {
        report.push(`- ⚠️ **${issue.header}**: ${issue.issue}`);
        report.push(`  Current: \`${issue.current}\``);
        report.push(`  Recommended: \`${issue.recommended}\``);
      });
    }
    
    return report.join('\n');
  }
}

export const headerScanner = new HeaderScanner();

// 使用示例
export const scanSecurityHeaders = async (url) => {
  const analysis = await headerScanner.scanUrl(url);
  const report = headerScanner.generateReport(analysis);
  
  return {
    analysis,
    report
  };
};
```

## 安全监控与响应

### 1. 实时安全监控

```javascript
// security/monitoring.js
class SecurityMonitor {
  constructor() {
    this.events = new Map();
    this.alertRules = new Map();
    this.setupDefaultRules();
  }

  setupDefaultRules() {
    // 暴力破解检测
    this.addAlertRule('brute_force', {
      condition: (events) => {
        const failedLogins = events.filter(e => e.type === 'login_failed');
        return failedLogins.length >= 5; // 5次失败登录
      },
      message: 'Possible brute force attack detected',
      severity: 'high'
    });

    // CSP 违规警报
    this.addAlertRule('csp_violation', {
      condition: (events) => {
        const violations = events.filter(e => e.type === 'csp_violation');
        return violations.length >= 3; // 3次CSP违规
      },
      message: 'Multiple CSP violations detected',
      severity: 'medium'
    });

    // 异常地理位置
    this.addAlertRule('geo_anomaly', {
      condition: (events) => {
        const locations = new Set(events.map(e => e.geoip?.country));
        return locations.size >= 3; // 来自3个不同国家
      },
      message: 'Suspicious geographic activity',
      severity: 'medium'
    });
  }

  addAlertRule(name, rule) {
    this.alertRules.set(name, rule);
  }

  // 记录安全事件
  recordEvent(type, data) {
    const event = {
      type,
      data,
      timestamp: Date.now(),
      id: crypto.randomBytes(16).toString('hex')
    };

    if (!this.events.has(type)) {
      this.events.set(type, []);
    }

    this.events.get(type).push(event);
    
    // 检查警报规则
    this.checkAlerts(type);
    
    return event;
  }

  // 检查警报
  checkAlerts(eventType) {
    for (const [ruleName, rule] of this.alertRules) {
      const relevantEvents = this.getEvents(rule.condition);
      
      if (rule.condition(relevantEvents)) {
        this.triggerAlert(ruleName, rule, relevantEvents);
      }
    }
  }

  getEvents(filter) {
    const allEvents = [];
    for (const events of this.events.values()) {
      allEvents.push(...events);
    }
    return filter ? allEvents.filter(filter) : allEvents;
  }

  // 触发警报
  async triggerAlert(ruleName, rule, events) {
    const alert = {
      id: crypto.randomBytes(16).toString('hex'),
      rule: ruleName,
      message: rule.message,
      severity: rule.severity,
      events: events.slice(-10), // 最后10个相关事件
      timestamp: Date.now()
    };

    // 发送警报
    await this.sendAlert(alert);
    
    // 记录警报
    this.recordEvent('security_alert', alert);
  }

  async sendAlert(alert) {
    // 实现警报发送逻辑
    console.log('Security Alert:', alert);
    
    // 发送到Slack、Email、SMS等
    if (process.env.SECURITY_ALERT_WEBHOOK) {
      await fetch(process.env.SECURITY_ALERT_WEBHOOK, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(alert)
      });
    }
  }

  // 安全事件查询
  queryEvents(options = {}) {
    let events = this.getEvents();
    
    if (options.type) {
      events = events.filter(e => e.type === options.type);
    }
    
    if (options.startTime) {
      events = events.filter(e => e.timestamp >= options.startTime);
    }
    
    if (options.endTime) {
      events = events.filter(e => e.timestamp <= options.endTime);
    }
    
    if (options.limit) {
      events = events.slice(-options.limit);
    }
    
    return events.sort((a, b) => b.timestamp - a.timestamp);
  }

  // 安全仪表板数据
  getDashboardData() {
    const last24Hours = Date.now() - 24 * 60 * 60 * 1000;
    const recentEvents = this.queryEvents({ startTime: last24Hours });
    
    const eventsByType = {};
    recentEvents.forEach(event => {
      if (!eventsByType[event.type]) {
        eventsByType[event.type] = 0;
      }
      eventsByType[event.type]++;
    });
    
    return {
      totalEvents: recentEvents.length,
      eventsByType,
      recentAlerts: this.queryEvents({ 
        type: 'security_alert', 
        limit: 5 
      })
    };
  }
}

export const securityMonitor = new SecurityMonitor();

// 中间件：记录HTTP请求
export const requestLoggerMiddleware = (req, res, next) => {
  const startTime = Date.now();
  
  res.on('finish', () => {
    const eventData = {
      method: req.method,
      url: req.url,
      status: res.statusCode,
      duration: Date.now() - startTime,
      userAgent: req.get('User-Agent'),
      ip: req.ip
    };
    
    if (res.statusCode >= 400) {
      securityMonitor.recordEvent('http_error', eventData);
    }
  });
  
  next();
};
```
