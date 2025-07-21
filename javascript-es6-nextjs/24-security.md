# บทที่ 24: Security

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ comprehensive security practices สำหรับ web applications
- ทำความเข้าใจ authentication, authorization, และ data protection
- เรียนรู้ security vulnerabilities และ countermeasures
- สร้าง secure Next.js applications ด้วย enterprise-level security

## 🔐 Authentication & Authorization

### **🔑 Advanced Authentication System**

```javascript
// ============= JWT AUTHENTICATION =============
// lib/auth/jwt-manager.js
import jwt from 'jsonwebtoken';
import crypto from 'crypto';

export class JWTManager {
  constructor(options = {}) {
    this.options = {
      accessTokenSecret: process.env.JWT_ACCESS_SECRET || 'access-secret',
      refreshTokenSecret: process.env.JWT_REFRESH_SECRET || 'refresh-secret',
      accessTokenExpiry: '15m',
      refreshTokenExpiry: '7d',
      issuer: 'your-app.com',
      audience: 'your-app-users',
      algorithm: 'HS256',
      ...options
    };
    
    this.blacklistedTokens = new Set();
    this.refreshTokens = new Map(); // In production, use Redis
  }
  
  generateTokenPair(payload) {
    const now = Math.floor(Date.now() / 1000);
    const jti = crypto.randomUUID();
    
    const commonClaims = {
      iss: this.options.issuer,
      aud: this.options.audience,
      iat: now,
      jti
    };
    
    // Generate access token
    const accessToken = jwt.sign(
      {
        ...payload,
        ...commonClaims,
        type: 'access',
        exp: now + this.parseExpiry(this.options.accessTokenExpiry)
      },
      this.options.accessTokenSecret,
      { algorithm: this.options.algorithm }
    );
    
    // Generate refresh token
    const refreshTokenId = crypto.randomUUID();
    const refreshToken = jwt.sign(
      {
        userId: payload.sub || payload.userId,
        ...commonClaims,
        type: 'refresh',
        tokenId: refreshTokenId,
        exp: now + this.parseExpiry(this.options.refreshTokenExpiry)
      },
      this.options.refreshTokenSecret,
      { algorithm: this.options.algorithm }
    );
    
    // Store refresh token
    this.refreshTokens.set(refreshTokenId, {
      userId: payload.sub || payload.userId,
      createdAt: now,
      lastUsed: now,
      userAgent: payload.userAgent,
      ipAddress: payload.ipAddress
    });
    
    return {
      accessToken,
      refreshToken,
      accessTokenExpiry: now + this.parseExpiry(this.options.accessTokenExpiry),
      refreshTokenExpiry: now + this.parseExpiry(this.options.refreshTokenExpiry)
    };
  }
  
  verifyAccessToken(token) {
    try {
      // Check if token is blacklisted
      if (this.blacklistedTokens.has(token)) {
        throw new Error('Token has been revoked');
      }
      
      const decoded = jwt.verify(token, this.options.accessTokenSecret, {
        issuer: this.options.issuer,
        audience: this.options.audience,
        algorithms: [this.options.algorithm]
      });
      
      if (decoded.type !== 'access') {
        throw new Error('Invalid token type');
      }
      
      return {
        valid: true,
        payload: decoded,
        expired: false
      };
    } catch (error) {
      if (error.name === 'TokenExpiredError') {
        return {
          valid: false,
          payload: null,
          expired: true,
          error: 'Token expired'
        };
      }
      
      return {
        valid: false,
        payload: null,
        expired: false,
        error: error.message
      };
    }
  }
  
  verifyRefreshToken(token) {
    try {
      const decoded = jwt.verify(token, this.options.refreshTokenSecret, {
        issuer: this.options.issuer,
        audience: this.options.audience,
        algorithms: [this.options.algorithm]
      });
      
      if (decoded.type !== 'refresh') {
        throw new Error('Invalid token type');
      }
      
      // Check if refresh token exists in store
      const tokenData = this.refreshTokens.get(decoded.tokenId);
      if (!tokenData) {
        throw new Error('Refresh token not found');
      }
      
      // Update last used timestamp
      tokenData.lastUsed = Math.floor(Date.now() / 1000);
      
      return {
        valid: true,
        payload: decoded,
        tokenData
      };
    } catch (error) {
      return {
        valid: false,
        payload: null,
        error: error.message
      };
    }
  }
  
  refreshAccessToken(refreshToken, userAgent, ipAddress) {
    const verification = this.verifyRefreshToken(refreshToken);
    
    if (!verification.valid) {
      throw new Error(verification.error);
    }
    
    const { payload, tokenData } = verification;
    
    // Validate request context
    if (tokenData.userAgent !== userAgent || tokenData.ipAddress !== ipAddress) {
      // Suspicious activity - revoke all tokens for user
      this.revokeAllUserTokens(payload.userId);
      throw new Error('Suspicious activity detected');
    }
    
    // Generate new access token
    const newTokenPair = this.generateTokenPair({
      sub: payload.userId,
      userId: payload.userId,
      userAgent,
      ipAddress
    });
    
    return {
      accessToken: newTokenPair.accessToken,
      accessTokenExpiry: newTokenPair.accessTokenExpiry
    };
  }
  
  revokeToken(token) {
    this.blacklistedTokens.add(token);
    
    // Clean up old blacklisted tokens periodically
    if (this.blacklistedTokens.size > 10000) {
      this.cleanupBlacklist();
    }
  }
  
  revokeRefreshToken(tokenId) {
    this.refreshTokens.delete(tokenId);
  }
  
  revokeAllUserTokens(userId) {
    // Revoke all refresh tokens for user
    for (const [tokenId, tokenData] of this.refreshTokens.entries()) {
      if (tokenData.userId === userId) {
        this.refreshTokens.delete(tokenId);
      }
    }
  }
  
  parseExpiry(expiry) {
    if (typeof expiry === 'number') return expiry;
    
    const units = { s: 1, m: 60, h: 3600, d: 86400 };
    const match = expiry.match(/^(\d+)([smhd])$/);
    
    if (!match) throw new Error('Invalid expiry format');
    
    return parseInt(match[1]) * units[match[2]];
  }
  
  cleanupBlacklist() {
    // Remove tokens that have definitely expired
    // This is a simplified version - in production, track token expiry
    this.blacklistedTokens.clear();
  }
  
  getActiveTokens(userId) {
    const userTokens = [];
    for (const [tokenId, tokenData] of this.refreshTokens.entries()) {
      if (tokenData.userId === userId) {
        userTokens.push({
          tokenId,
          createdAt: tokenData.createdAt,
          lastUsed: tokenData.lastUsed,
          userAgent: tokenData.userAgent,
          ipAddress: tokenData.ipAddress
        });
      }
    }
    return userTokens;
  }
}

// ============= MULTI-FACTOR AUTHENTICATION =============
// lib/auth/mfa-manager.js
import speakeasy from 'speakeasy';
import QRCode from 'qrcode';
import crypto from 'crypto';

export class MFAManager {
  constructor(options = {}) {
    this.options = {
      appName: 'Your App',
      tokenWindow: 2, // Allow 2 time steps before/after current
      backupCodeLength: 8,
      backupCodeCount: 10,
      ...options
    };
  }
  
  async generateTOTPSecret(userEmail) {
    const secret = speakeasy.generateSecret({
      name: userEmail,
      issuer: this.options.appName,
      length: 32
    });
    
    return {
      secret: secret.base32,
      qrCode: await QRCode.toDataURL(secret.otpauth_url)
    };
  }
  
  verifyTOTPToken(token, secret) {
    return speakeasy.totp.verify({
      secret,
      encoding: 'base32',
      token,
      window: this.options.tokenWindow
    });
  }
  
  generateBackupCodes() {
    const codes = [];
    for (let i = 0; i < this.options.backupCodeCount; i++) {
      codes.push(this.generateBackupCode());
    }
    return codes;
  }
  
  generateBackupCode() {
    const chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789';
    let result = '';
    for (let i = 0; i < this.options.backupCodeLength; i++) {
      result += chars.charAt(Math.floor(Math.random() * chars.length));
    }
    return result;
  }
  
  hashBackupCode(code) {
    return crypto.createHash('sha256').update(code).digest('hex');
  }
  
  verifyBackupCode(code, hashedCodes) {
    const hashedInput = this.hashBackupCode(code.toUpperCase());
    return hashedCodes.includes(hashedInput);
  }
  
  async sendSMSCode(phoneNumber) {
    const code = Math.floor(100000 + Math.random() * 900000).toString();
    
    // In production, integrate with SMS service
    console.log(`SMS Code for ${phoneNumber}: ${code}`);
    
    return {
      code: crypto.createHash('sha256').update(code).digest('hex'),
      expiresAt: Date.now() + 5 * 60 * 1000 // 5 minutes
    };
  }
  
  verifySMSCode(inputCode, hashedCode, expiresAt) {
    if (Date.now() > expiresAt) {
      return { valid: false, reason: 'Code expired' };
    }
    
    const hashedInput = crypto.createHash('sha256').update(inputCode).digest('hex');
    return {
      valid: hashedInput === hashedCode,
      reason: hashedInput === hashedCode ? 'Valid' : 'Invalid code'
    };
  }
}

// ============= RBAC AUTHORIZATION =============
// lib/auth/rbac-manager.js

export class RBACManager {
  constructor() {
    this.roles = new Map();
    this.permissions = new Map();
    this.userRoles = new Map();
    this.userPermissions = new Map();
    this.setupDefaultRoles();
  }
  
  setupDefaultRoles() {
    // Define permissions
    const permissions = [
      // User management
      { id: 'user.read', name: 'Read Users', description: 'View user information' },
      { id: 'user.create', name: 'Create Users', description: 'Create new users' },
      { id: 'user.update', name: 'Update Users', description: 'Modify user information' },
      { id: 'user.delete', name: 'Delete Users', description: 'Remove users' },
      
      // Content management
      { id: 'content.read', name: 'Read Content', description: 'View content' },
      { id: 'content.create', name: 'Create Content', description: 'Create new content' },
      { id: 'content.update', name: 'Update Content', description: 'Modify content' },
      { id: 'content.delete', name: 'Delete Content', description: 'Remove content' },
      { id: 'content.publish', name: 'Publish Content', description: 'Publish content' },
      
      // System administration
      { id: 'system.admin', name: 'System Admin', description: 'Full system access' },
      { id: 'system.settings', name: 'System Settings', description: 'Modify system settings' },
      { id: 'system.logs', name: 'System Logs', description: 'View system logs' }
    ];
    
    permissions.forEach(permission => {
      this.permissions.set(permission.id, permission);
    });
    
    // Define roles
    this.createRole('admin', 'Administrator', 'Full system access', [
      'user.read', 'user.create', 'user.update', 'user.delete',
      'content.read', 'content.create', 'content.update', 'content.delete', 'content.publish',
      'system.admin', 'system.settings', 'system.logs'
    ]);
    
    this.createRole('editor', 'Editor', 'Content management access', [
      'user.read',
      'content.read', 'content.create', 'content.update', 'content.publish'
    ]);
    
    this.createRole('author', 'Author', 'Content creation access', [
      'content.read', 'content.create', 'content.update'
    ]);
    
    this.createRole('viewer', 'Viewer', 'Read-only access', [
      'content.read'
    ]);
  }
  
  createRole(id, name, description, permissionIds = []) {
    const role = {
      id,
      name,
      description,
      permissions: new Set(permissionIds),
      inherits: new Set()
    };
    
    this.roles.set(id, role);
    return role;
  }
  
  addPermissionToRole(roleId, permissionId) {
    const role = this.roles.get(roleId);
    if (role && this.permissions.has(permissionId)) {
      role.permissions.add(permissionId);
      return true;
    }
    return false;
  }
  
  removePermissionFromRole(roleId, permissionId) {
    const role = this.roles.get(roleId);
    if (role) {
      role.permissions.delete(permissionId);
      return true;
    }
    return false;
  }
  
  setRoleInheritance(childRoleId, parentRoleId) {
    const childRole = this.roles.get(childRoleId);
    const parentRole = this.roles.get(parentRoleId);
    
    if (childRole && parentRole) {
      childRole.inherits.add(parentRoleId);
      return true;
    }
    return false;
  }
  
  assignRoleToUser(userId, roleId) {
    if (!this.userRoles.has(userId)) {
      this.userRoles.set(userId, new Set());
    }
    
    if (this.roles.has(roleId)) {
      this.userRoles.get(userId).add(roleId);
      this.refreshUserPermissions(userId);
      return true;
    }
    return false;
  }
  
  removeRoleFromUser(userId, roleId) {
    const userRoles = this.userRoles.get(userId);
    if (userRoles) {
      userRoles.delete(roleId);
      this.refreshUserPermissions(userId);
      return true;
    }
    return false;
  }
  
  grantPermissionToUser(userId, permissionId) {
    if (!this.userPermissions.has(userId)) {
      this.userPermissions.set(userId, new Set());
    }
    
    if (this.permissions.has(permissionId)) {
      this.userPermissions.get(userId).add(permissionId);
      return true;
    }
    return false;
  }
  
  revokePermissionFromUser(userId, permissionId) {
    const userPermissions = this.userPermissions.get(userId);
    if (userPermissions) {
      userPermissions.delete(permissionId);
      return true;
    }
    return false;
  }
  
  refreshUserPermissions(userId) {
    const userRoles = this.userRoles.get(userId) || new Set();
    const allPermissions = new Set();
    
    // Collect permissions from roles (including inherited)
    for (const roleId of userRoles) {
      const rolePermissions = this.getRolePermissions(roleId);
      rolePermissions.forEach(permission => allPermissions.add(permission));
    }
    
    // Add direct user permissions
    const directPermissions = this.userPermissions.get(userId) || new Set();
    directPermissions.forEach(permission => allPermissions.add(permission));
    
    // Cache computed permissions
    this.userPermissions.set(userId, allPermissions);
  }
  
  getRolePermissions(roleId, visited = new Set()) {
    const role = this.roles.get(roleId);
    if (!role || visited.has(roleId)) {
      return new Set();
    }
    
    visited.add(roleId);
    const permissions = new Set(role.permissions);
    
    // Add inherited permissions
    for (const parentRoleId of role.inherits) {
      const inheritedPermissions = this.getRolePermissions(parentRoleId, visited);
      inheritedPermissions.forEach(permission => permissions.add(permission));
    }
    
    return permissions;
  }
  
  hasPermission(userId, permissionId) {
    const userPermissions = this.userPermissions.get(userId);
    return userPermissions ? userPermissions.has(permissionId) : false;
  }
  
  hasAnyPermission(userId, permissionIds) {
    return permissionIds.some(permissionId => this.hasPermission(userId, permissionId));
  }
  
  hasAllPermissions(userId, permissionIds) {
    return permissionIds.every(permissionId => this.hasPermission(userId, permissionId));
  }
  
  getUserRoles(userId) {
    return Array.from(this.userRoles.get(userId) || []).map(roleId => this.roles.get(roleId));
  }
  
  getUserPermissions(userId) {
    return Array.from(this.userPermissions.get(userId) || []).map(permissionId => this.permissions.get(permissionId));
  }
  
  // Policy-based access control
  evaluatePolicy(userId, resource, action, context = {}) {
    const permission = `${resource}.${action}`;
    
    // Basic permission check
    if (!this.hasPermission(userId, permission)) {
      return { allowed: false, reason: 'Insufficient permissions' };
    }
    
    // Resource-specific policies
    const policies = this.getResourcePolicies(resource);
    for (const policy of policies) {
      const result = policy(userId, action, context);
      if (!result.allowed) {
        return result;
      }
    }
    
    return { allowed: true };
  }
  
  getResourcePolicies(resource) {
    // Example policies
    const policies = {
      user: [
        // Users can only modify their own profile
        (userId, action, context) => {
          if (action === 'update' && context.targetUserId !== userId) {
            const isAdmin = this.hasPermission(userId, 'system.admin');
            return {
              allowed: isAdmin,
              reason: isAdmin ? 'Admin override' : 'Cannot modify other users'
            };
          }
          return { allowed: true };
        }
      ],
      
      content: [
        // Authors can only modify their own content
        (userId, action, context) => {
          if (['update', 'delete'].includes(action) && context.authorId !== userId) {
            const isEditor = this.hasPermission(userId, 'content.publish');
            return {
              allowed: isEditor,
              reason: isEditor ? 'Editor override' : 'Cannot modify content by other authors'
            };
          }
          return { allowed: true };
        }
      ]
    };
    
    return policies[resource] || [];
  }
}
```

## 🛡️ Security Vulnerabilities & Protection

### **🔒 Security Middleware & Protection**

```javascript
// ============= SECURITY MIDDLEWARE =============
// lib/security/security-middleware.js
import rateLimit from 'express-rate-limit';
import helmet from 'helmet';
import cors from 'cors';
import crypto from 'crypto';

export class SecurityMiddleware {
  constructor(options = {}) {
    this.options = {
      rateLimiting: {
        windowMs: 15 * 60 * 1000, // 15 minutes
        max: 100, // limit each IP to 100 requests per windowMs
        ...options.rateLimiting
      },
      cors: {
        origin: process.env.ALLOWED_ORIGINS?.split(',') || ['http://localhost:3000'],
        credentials: true,
        ...options.cors
      },
      csp: {
        defaultSrc: ["'self'"],
        scriptSrc: ["'self'", "'unsafe-inline'"],
        styleSrc: ["'self'", "'unsafe-inline'"],
        imgSrc: ["'self'", "data:", "https:"],
        ...options.csp
      },
      ...options
    };
    
    this.setupMiddleware();
  }
  
  setupMiddleware() {
    // Rate limiting
    this.rateLimiter = rateLimit({
      windowMs: this.options.rateLimiting.windowMs,
      max: this.options.rateLimiting.max,
      message: { error: 'Too many requests, please try again later' },
      standardHeaders: true,
      legacyHeaders: false,
      // Custom key generator for more sophisticated rate limiting
      keyGenerator: (req) => {
        return req.ip + ':' + (req.user?.id || 'anonymous');
      }
    });
    
    // Helmet for security headers
    this.helmetMiddleware = helmet({
      contentSecurityPolicy: {
        directives: this.options.csp
      },
      hsts: {
        maxAge: 31536000,
        includeSubDomains: true,
        preload: true
      }
    });
    
    // CORS
    this.corsMiddleware = cors(this.options.cors);
  }
  
  // CSRF Protection
  generateCSRFToken() {
    return crypto.randomBytes(32).toString('hex');
  }
  
  verifyCSRFToken(token, sessionToken) {
    if (!token || !sessionToken) return false;
    return crypto.timingSafeEqual(
      Buffer.from(token, 'hex'),
      Buffer.from(sessionToken, 'hex')
    );
  }
  
  csrfProtection() {
    return (req, res, next) => {
      if (['GET', 'HEAD', 'OPTIONS'].includes(req.method)) {
        return next();
      }
      
      const token = req.headers['x-csrf-token'] || req.body._csrf;
      const sessionToken = req.session?.csrfToken;
      
      if (!this.verifyCSRFToken(token, sessionToken)) {
        return res.status(403).json({ error: 'Invalid CSRF token' });
      }
      
      next();
    };
  }
  
  // Input Sanitization
  sanitizeInput() {
    return (req, res, next) => {
      const sanitize = (obj) => {
        if (typeof obj === 'string') {
          // Basic XSS protection
          return obj
            .replace(/<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/gi, '')
            .replace(/javascript:/gi, '')
            .replace(/on\w+\s*=/gi, '');
        }
        
        if (typeof obj === 'object' && obj !== null) {
          const sanitized = {};
          for (const [key, value] of Object.entries(obj)) {
            sanitized[key] = sanitize(value);
          }
          return sanitized;
        }
        
        return obj;
      };
      
      if (req.body) req.body = sanitize(req.body);
      if (req.query) req.query = sanitize(req.query);
      if (req.params) req.params = sanitize(req.params);
      
      next();
    };
  }
  
  // SQL Injection Protection
  validateSQL() {
    return (req, res, next) => {
      const sqlInjectionPattern = /(\b(SELECT|INSERT|UPDATE|DELETE|DROP|CREATE|ALTER|EXEC|UNION|SCRIPT)\b)|(\-\-)|(\;)/gi;
      
      const checkForSQLInjection = (value) => {
        if (typeof value === 'string') {
          return sqlInjectionPattern.test(value);
        }
        if (typeof value === 'object' && value !== null) {
          return Object.values(value).some(checkForSQLInjection);
        }
        return false;
      };
      
      if (checkForSQLInjection(req.body) || 
          checkForSQLInjection(req.query) || 
          checkForSQLInjection(req.params)) {
        return res.status(400).json({ error: 'Invalid input detected' });
      }
      
      next();
    };
  }
  
  // Request size limiting
  requestSizeLimit(maxSize = '10mb') {
    return (req, res, next) => {
      const contentLength = parseInt(req.headers['content-length'] || '0');
      const maxBytes = this.parseSize(maxSize);
      
      if (contentLength > maxBytes) {
        return res.status(413).json({ error: 'Request too large' });
      }
      
      next();
    };
  }
  
  parseSize(size) {
    const units = { b: 1, kb: 1024, mb: 1024 * 1024, gb: 1024 * 1024 * 1024 };
    const match = size.toString().toLowerCase().match(/^(\d+)(b|kb|mb|gb)?$/);
    
    if (!match) throw new Error('Invalid size format');
    
    return parseInt(match[1]) * (units[match[2]] || 1);
  }
  
  // Security headers
  securityHeaders() {
    return (req, res, next) => {
      // Remove server information
      res.removeHeader('X-Powered-By');
      
      // Add security headers
      res.setHeader('X-Content-Type-Options', 'nosniff');
      res.setHeader('X-Frame-Options', 'DENY');
      res.setHeader('X-XSS-Protection', '1; mode=block');
      res.setHeader('Referrer-Policy', 'strict-origin-when-cross-origin');
      res.setHeader('Permissions-Policy', 'geolocation=(), microphone=(), camera=()');
      
      next();
    };
  }
  
  // Apply all security middleware
  applyAll() {
    return [
      this.helmetMiddleware,
      this.corsMiddleware,
      this.rateLimiter,
      this.securityHeaders(),
      this.sanitizeInput(),
      this.validateSQL(),
      this.requestSizeLimit()
    ];
  }
}

// ============= ENCRYPTION & HASHING =============
// lib/security/crypto-utils.js
import bcrypt from 'bcryptjs';
import crypto from 'crypto';

export class CryptoUtils {
  constructor(options = {}) {
    this.options = {
      saltRounds: 12,
      algorithm: 'aes-256-gcm',
      keyLength: 32,
      ivLength: 16,
      tagLength: 16,
      ...options
    };
  }
  
  // Password hashing
  async hashPassword(password) {
    const salt = await bcrypt.genSalt(this.options.saltRounds);
    return bcrypt.hash(password, salt);
  }
  
  async verifyPassword(password, hash) {
    return bcrypt.compare(password, hash);
  }
  
  // Symmetric encryption
  encrypt(text, key) {
    const keyBuffer = typeof key === 'string' ? 
      crypto.scryptSync(key, 'salt', this.options.keyLength) : key;
    
    const iv = crypto.randomBytes(this.options.ivLength);
    const cipher = crypto.createCipher(this.options.algorithm, keyBuffer, { iv });
    
    let encrypted = cipher.update(text, 'utf8', 'hex');
    encrypted += cipher.final('hex');
    
    const tag = cipher.getAuthTag();
    
    return {
      encrypted,
      iv: iv.toString('hex'),
      tag: tag.toString('hex')
    };
  }
  
  decrypt(encryptedData, key) {
    const keyBuffer = typeof key === 'string' ? 
      crypto.scryptSync(key, 'salt', this.options.keyLength) : key;
    
    const decipher = crypto.createDecipherGCM(
      this.options.algorithm,
      keyBuffer,
      Buffer.from(encryptedData.iv, 'hex')
    );
    
    decipher.setAuthTag(Buffer.from(encryptedData.tag, 'hex'));
    
    let decrypted = decipher.update(encryptedData.encrypted, 'hex', 'utf8');
    decrypted += decipher.final('utf8');
    
    return decrypted;
  }
  
  // Generate secure random key
  generateKey() {
    return crypto.randomBytes(this.options.keyLength);
  }
  
  // HMAC signing
  sign(message, secret) {
    return crypto
      .createHmac('sha256', secret)
      .update(message)
      .digest('hex');
  }
  
  verifySignature(message, signature, secret) {
    const expectedSignature = this.sign(message, secret);
    return crypto.timingSafeEqual(
      Buffer.from(signature, 'hex'),
      Buffer.from(expectedSignature, 'hex')
    );
  }
  
  // Secure random generation
  generateSecureRandom(length = 32) {
    return crypto.randomBytes(length).toString('hex');
  }
  
  generateNumericCode(length = 6) {
    const max = Math.pow(10, length) - 1;
    const min = Math.pow(10, length - 1);
    return Math.floor(Math.random() * (max - min + 1)) + min;
  }
  
  // Key derivation
  deriveKey(password, salt, iterations = 100000) {
    return crypto.pbkdf2Sync(password, salt, iterations, this.options.keyLength, 'sha256');
  }
  
  // Secure comparison
  secureCompare(a, b) {
    if (a.length !== b.length) return false;
    return crypto.timingSafeEqual(Buffer.from(a), Buffer.from(b));
  }
}

// ============= AUDIT LOGGING =============
// lib/security/audit-logger.js

export class AuditLogger {
  constructor(options = {}) {
    this.options = {
      logLevel: 'info',
      enableConsole: true,
      enableFile: true,
      logFile: 'audit.log',
      enableDatabase: false,
      ...options
    };
    
    this.eventTypes = {
      AUTH_LOGIN: 'auth.login',
      AUTH_LOGOUT: 'auth.logout',
      AUTH_FAILED: 'auth.failed',
      PERMISSION_DENIED: 'permission.denied',
      DATA_ACCESS: 'data.access',
      DATA_MODIFY: 'data.modify',
      SYSTEM_CHANGE: 'system.change',
      SECURITY_VIOLATION: 'security.violation'
    };
  }
  
  async log(eventType, userId, details = {}, req = null) {
    const logEntry = {
      timestamp: new Date().toISOString(),
      eventType,
      userId,
      sessionId: req?.sessionID,
      ipAddress: req?.ip || req?.connection?.remoteAddress,
      userAgent: req?.headers['user-agent'],
      method: req?.method,
      url: req?.originalUrl,
      details,
      severity: this.getSeverity(eventType)
    };
    
    // Add request fingerprint for tracking
    if (req) {
      logEntry.fingerprint = this.generateFingerprint(req);
    }
    
    await this.writeLog(logEntry);
    
    // Alert on high severity events
    if (logEntry.severity === 'high') {
      await this.sendSecurityAlert(logEntry);
    }
  }
  
  getSeverity(eventType) {
    const severityMap = {
      [this.eventTypes.AUTH_LOGIN]: 'low',
      [this.eventTypes.AUTH_LOGOUT]: 'low',
      [this.eventTypes.AUTH_FAILED]: 'medium',
      [this.eventTypes.PERMISSION_DENIED]: 'medium',
      [this.eventTypes.DATA_ACCESS]: 'low',
      [this.eventTypes.DATA_MODIFY]: 'medium',
      [this.eventTypes.SYSTEM_CHANGE]: 'high',
      [this.eventTypes.SECURITY_VIOLATION]: 'high'
    };
    
    return severityMap[eventType] || 'medium';
  }
  
  generateFingerprint(req) {
    const components = [
      req.ip,
      req.headers['user-agent'],
      req.headers['accept-language'],
      req.headers['accept-encoding']
    ].filter(Boolean);
    
    return crypto
      .createHash('sha256')
      .update(components.join('|'))
      .digest('hex')
      .substring(0, 16);
  }
  
  async writeLog(logEntry) {
    const logString = JSON.stringify(logEntry);
    
    // Console logging
    if (this.options.enableConsole) {
      console.log(`[AUDIT] ${logString}`);
    }
    
    // File logging
    if (this.options.enableFile) {
      const fs = await import('fs/promises');
      await fs.appendFile(this.options.logFile, logString + '\n');
    }
    
    // Database logging
    if (this.options.enableDatabase) {
      await this.writeToDatabase(logEntry);
    }
  }
  
  async writeToDatabase(logEntry) {
    // Implementation depends on your database
    // Example with Prisma:
    /*
    await prisma.auditLog.create({
      data: logEntry
    });
    */
  }
  
  async sendSecurityAlert(logEntry) {
    // Send alert to security team
    console.error(`[SECURITY ALERT] ${JSON.stringify(logEntry)}`);
    
    // In production, integrate with alerting system
    // - Email notifications
    // - Slack/Teams alerts
    // - PagerDuty incidents
  }
  
  // Search and analysis methods
  async searchLogs(filters = {}) {
    // Implementation for log search and analysis
    // This would typically query your log storage system
  }
  
  async generateReport(timeRange, eventTypes = []) {
    // Generate security reports
    // Aggregate suspicious activities
    // Track trends and patterns
  }
}
```

## 🎯 แบบฝึกหัด

### **🔐 แบบฝึกหัดที่ 1: Authentication System**
สร้าง comprehensive authentication:

```javascript
// TODO: สร้าง authentication system

// 1. OAuth integration
class OAuthManager {
  // Multiple providers
  // PKCE flow
  // State validation
}

// 2. Biometric authentication
class BiometricAuth {
  // WebAuthn API
  // Device registration
  // Fallback mechanisms
}

// 3. Risk-based authentication
class RiskAssessment {
  // Behavioral analysis
  // Device fingerprinting
  // Adaptive security
}
```

### **🛡️ แบบฝึกหัดที่ 2: Security Monitoring**
สร้าง security monitoring system:

```javascript
// TODO: สร้าง security monitoring

// 1. Threat detection
class ThreatDetector {
  // Anomaly detection
  // Pattern recognition
  // Real-time alerts
}

// 2. Vulnerability scanner
class VulnerabilityScanner {
  // Dependency scanning
  // Code analysis
  // Configuration checks
}

// 3. Incident response
class IncidentResponse {
  // Automated response
  // Forensic data collection
  // Recovery procedures
}
```

### **🔒 แบบฝึกหัดที่ 3: Data Protection**
สร้าง comprehensive data protection:

```javascript
// TODO: สร้าง data protection system

// 1. Data classification
class DataClassifier {
  // Sensitivity levels
  // Automatic tagging
  // Access policies
}

// 2. Encryption management
class EncryptionManager {
  // Key rotation
  // Multi-layer encryption
  // HSM integration
}

// 3. Privacy compliance
class PrivacyManager {
  // GDPR compliance
  // Data retention
  // Consent management
}
```

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 23: State Management](./23-state-management.md)

**บทถัดไป**: [บทที่ 25: Advanced Patterns](./25-advanced-patterns.md)

**กลับไปหน้าหลัก**: [README](../README.md)

---

## 📚 เอกสารอ้างอิง

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Web Security Fundamentals](https://developer.mozilla.org/en-US/docs/Web/Security)
- [JWT Best Practices](https://auth0.com/blog/a-look-at-the-latest-draft-for-jwt-bcp/)
- [Next.js Security](https://nextjs.org/docs/advanced-features/security-headers)

---

## 🎉 สรุป

ในบทนี้เราได้เรียนรู้:

- **Authentication & Authorization** - JWT, MFA, RBAC, และ policy-based access control
- **Security Middleware** - CSRF protection, input sanitization, และ rate limiting
- **Encryption & Hashing** - secure password handling, data encryption, และ key management
- **Security Monitoring** - audit logging, threat detection, และ incident response
- **Best Practices** - comprehensive security architecture และ defense in depth

Security เป็นหัวใจสำคัญของทุก application! 🚀
