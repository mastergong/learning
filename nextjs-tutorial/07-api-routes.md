# บทที่ 7: API Routes

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้การสร้าง API Routes ใน Next.js
- ทำความเข้าใจ HTTP Methods (GET, POST, PUT, DELETE)
- เรียนรู้ Middleware และ Authentication
- เข้าใจ API Error Handling และ Validation

## 🛣️ API Routes Overview

### **📁 API Routes Structure**

```
pages/api/
├── hello.js                → /api/hello
├── users/
│   ├── index.js           → /api/users
│   ├── [id].js            → /api/users/123
│   └── [id]/
│       └── posts.js       → /api/users/123/posts
├── auth/
│   ├── login.js           → /api/auth/login
│   ├── register.js        → /api/auth/register
│   └── logout.js          → /api/auth/logout
└── admin/
    └── [...params].js     → /api/admin/any/path
```

### **⚡ Basic API Route**

```javascript
// pages/api/hello.js
export default function handler(req, res) {
  // ✅ Method check
  if (req.method !== 'GET') {
    return res.status(405).json({ 
      error: 'Method Not Allowed',
      message: 'Only GET method is allowed'
    });
  }

  // ✅ Query parameters
  const { name = 'World' } = req.query;

  // ✅ Response
  res.status(200).json({
    message: `Hello, ${name}!`,
    timestamp: new Date().toISOString(),
    method: req.method,
    userAgent: req.headers['user-agent']
  });
}
```

## 🔄 HTTP Methods

### **📊 CRUD Operations**

```javascript
// pages/api/users/index.js
import { connectToDatabase } from '@/lib/mongodb';
import { validateUser, sanitizeUser } from '@/lib/validation';

export default async function handler(req, res) {
  try {
    const { db } = await connectToDatabase();
    
    switch (req.method) {
      case 'GET':
        return await getUsers(req, res, db);
      case 'POST':
        return await createUser(req, res, db);
      default:
        return res.status(405).json({ 
          error: 'Method Not Allowed',
          allowedMethods: ['GET', 'POST']
        });
    }
  } catch (error) {
    console.error('API Error:', error);
    return res.status(500).json({
      error: 'Internal Server Error',
      message: 'Something went wrong'
    });
  }
}

// ✅ GET /api/users - List users
async function getUsers(req, res, db) {
  try {
    const { 
      page = 1, 
      limit = 10, 
      search = '', 
      role = '',
      sortBy = 'createdAt',
      sortOrder = 'desc'
    } = req.query;

    // ✅ Build query
    const query = {};
    if (search) {
      query.$or = [
        { name: { $regex: search, $options: 'i' } },
        { email: { $regex: search, $options: 'i' } }
      ];
    }
    if (role) {
      query.role = role;
    }

    // ✅ Pagination
    const skip = (parseInt(page) - 1) * parseInt(limit);
    const sortDirection = sortOrder === 'desc' ? -1 : 1;

    // ✅ Execute queries
    const [users, total] = await Promise.all([
      db.collection('users')
        .find(query)
        .sort({ [sortBy]: sortDirection })
        .skip(skip)
        .limit(parseInt(limit))
        .project({ password: 0 }) // ไม่ส่ง password
        .toArray(),
      db.collection('users').countDocuments(query)
    ]);

    // ✅ Response with metadata
    res.status(200).json({
      users: users.map(sanitizeUser),
      pagination: {
        page: parseInt(page),
        limit: parseInt(limit),
        total,
        pages: Math.ceil(total / parseInt(limit)),
        hasNext: skip + parseInt(limit) < total,
        hasPrev: parseInt(page) > 1
      },
      filters: { search, role, sortBy, sortOrder }
    });
  } catch (error) {
    console.error('Get users error:', error);
    res.status(500).json({
      error: 'Failed to fetch users',
      message: error.message
    });
  }
}

// ✅ POST /api/users - Create user
async function createUser(req, res, db) {
  try {
    // ✅ Validate request body
    const validation = validateUser(req.body);
    if (!validation.isValid) {
      return res.status(400).json({
        error: 'Validation Error',
        details: validation.errors
      });
    }

    const { name, email, password, role = 'user' } = req.body;

    // ✅ Check if user exists
    const existingUser = await db.collection('users').findOne({ email });
    if (existingUser) {
      return res.status(409).json({
        error: 'User already exists',
        message: 'A user with this email already exists'
      });
    }

    // ✅ Hash password
    const bcrypt = require('bcryptjs');
    const hashedPassword = await bcrypt.hash(password, 12);

    // ✅ Create user
    const newUser = {
      name,
      email: email.toLowerCase(),
      password: hashedPassword,
      role,
      createdAt: new Date(),
      updatedAt: new Date(),
      isActive: true,
      emailVerified: false
    };

    const result = await db.collection('users').insertOne(newUser);

    // ✅ Remove password from response
    const userResponse = sanitizeUser({
      _id: result.insertedId,
      ...newUser
    });

    res.status(201).json({
      message: 'User created successfully',
      user: userResponse
    });
  } catch (error) {
    console.error('Create user error:', error);
    res.status(500).json({
      error: 'Failed to create user',
      message: error.message
    });
  }
}
```

### **🎯 Dynamic API Routes**

```javascript
// pages/api/users/[id].js
import { connectToDatabase } from '@/lib/mongodb';
import { ObjectId } from 'mongodb';
import { validateUserUpdate, sanitizeUser } from '@/lib/validation';
import { requireAuth } from '@/middleware/auth';

export default async function handler(req, res) {
  try {
    // ✅ Authentication check
    const authResult = await requireAuth(req, res);
    if (!authResult.user) return; // Response already sent

    const { db } = await connectToDatabase();
    const { id } = req.query;

    // ✅ Validate ObjectId
    if (!ObjectId.isValid(id)) {
      return res.status(400).json({
        error: 'Invalid user ID',
        message: 'The provided user ID is not valid'
      });
    }

    switch (req.method) {
      case 'GET':
        return await getUser(req, res, db, id, authResult.user);
      case 'PUT':
        return await updateUser(req, res, db, id, authResult.user);
      case 'DELETE':
        return await deleteUser(req, res, db, id, authResult.user);
      default:
        return res.status(405).json({ 
          error: 'Method Not Allowed',
          allowedMethods: ['GET', 'PUT', 'DELETE']
        });
    }
  } catch (error) {
    console.error('API Error:', error);
    return res.status(500).json({
      error: 'Internal Server Error',
      message: 'Something went wrong'
    });
  }
}

// ✅ GET /api/users/[id] - Get single user
async function getUser(req, res, db, id, currentUser) {
  try {
    const user = await db.collection('users').findOne(
      { _id: new ObjectId(id) },
      { projection: { password: 0 } }
    );

    if (!user) {
      return res.status(404).json({
        error: 'User not found',
        message: 'The requested user does not exist'
      });
    }

    // ✅ Authorization check
    if (currentUser.role !== 'admin' && currentUser.id !== id) {
      return res.status(403).json({
        error: 'Forbidden',
        message: 'You can only access your own profile'
      });
    }

    res.status(200).json({
      user: sanitizeUser(user)
    });
  } catch (error) {
    console.error('Get user error:', error);
    res.status(500).json({
      error: 'Failed to fetch user',
      message: error.message
    });
  }
}

// ✅ PUT /api/users/[id] - Update user
async function updateUser(req, res, db, id, currentUser) {
  try {
    // ✅ Authorization check
    if (currentUser.role !== 'admin' && currentUser.id !== id) {
      return res.status(403).json({
        error: 'Forbidden',
        message: 'You can only update your own profile'
      });
    }

    // ✅ Validate update data
    const validation = validateUserUpdate(req.body);
    if (!validation.isValid) {
      return res.status(400).json({
        error: 'Validation Error',
        details: validation.errors
      });
    }

    const updateData = { ...req.body };
    
    // ✅ Handle password update
    if (updateData.password) {
      const bcrypt = require('bcryptjs');
      updateData.password = await bcrypt.hash(updateData.password, 12);
    }

    // ✅ Handle email update
    if (updateData.email) {
      updateData.email = updateData.email.toLowerCase();
      
      // Check if email is already taken
      const existingUser = await db.collection('users').findOne({
        email: updateData.email,
        _id: { $ne: new ObjectId(id) }
      });
      
      if (existingUser) {
        return res.status(409).json({
          error: 'Email already taken',
          message: 'This email is already in use'
        });
      }
    }

    updateData.updatedAt = new Date();

    // ✅ Update user
    const result = await db.collection('users').findOneAndUpdate(
      { _id: new ObjectId(id) },
      { $set: updateData },
      { 
        returnDocument: 'after',
        projection: { password: 0 }
      }
    );

    if (!result.value) {
      return res.status(404).json({
        error: 'User not found',
        message: 'The user to update does not exist'
      });
    }

    res.status(200).json({
      message: 'User updated successfully',
      user: sanitizeUser(result.value)
    });
  } catch (error) {
    console.error('Update user error:', error);
    res.status(500).json({
      error: 'Failed to update user',
      message: error.message
    });
  }
}

// ✅ DELETE /api/users/[id] - Delete user
async function deleteUser(req, res, db, id, currentUser) {
  try {
    // ✅ Authorization check (only admin can delete users)
    if (currentUser.role !== 'admin') {
      return res.status(403).json({
        error: 'Forbidden',
        message: 'Only administrators can delete users'
      });
    }

    // ✅ Prevent self-deletion
    if (currentUser.id === id) {
      return res.status(400).json({
        error: 'Bad Request',
        message: 'You cannot delete your own account'
      });
    }

    // ✅ Soft delete (mark as inactive)
    const result = await db.collection('users').findOneAndUpdate(
      { _id: new ObjectId(id) },
      { 
        $set: { 
          isActive: false,
          deletedAt: new Date(),
          deletedBy: currentUser.id
        } 
      },
      { returnDocument: 'after' }
    );

    if (!result.value) {
      return res.status(404).json({
        error: 'User not found',
        message: 'The user to delete does not exist'
      });
    }

    res.status(200).json({
      message: 'User deleted successfully',
      user: { id: result.value._id, isActive: false }
    });
  } catch (error) {
    console.error('Delete user error:', error);
    res.status(500).json({
      error: 'Failed to delete user',
      message: error.message
    });
  }
}
```

## 🔐 Authentication API

### **🚪 Login Endpoint**

```javascript
// pages/api/auth/login.js
import { connectToDatabase } from '@/lib/mongodb';
import bcrypt from 'bcryptjs';
import jwt from 'jsonwebtoken';
import { setCookie } from 'cookies-next';
import { validateLogin } from '@/lib/validation';

export default async function handler(req, res) {
  if (req.method !== 'POST') {
    return res.status(405).json({ 
      error: 'Method Not Allowed',
      message: 'Only POST method is allowed'
    });
  }

  try {
    // ✅ Validate request body
    const validation = validateLogin(req.body);
    if (!validation.isValid) {
      return res.status(400).json({
        error: 'Validation Error',
        details: validation.errors
      });
    }

    const { email, password, rememberMe = false } = req.body;
    const { db } = await connectToDatabase();

    // ✅ Find user
    const user = await db.collection('users').findOne({ 
      email: email.toLowerCase(),
      isActive: true
    });

    if (!user) {
      return res.status(401).json({
        error: 'Invalid credentials',
        message: 'Email or password is incorrect'
      });
    }

    // ✅ Verify password
    const isValidPassword = await bcrypt.compare(password, user.password);
    if (!isValidPassword) {
      return res.status(401).json({
        error: 'Invalid credentials',
        message: 'Email or password is incorrect'
      });
    }

    // ✅ Check if email is verified
    if (!user.emailVerified) {
      return res.status(403).json({
        error: 'Email not verified',
        message: 'Please verify your email before logging in'
      });
    }

    // ✅ Generate JWT token
    const tokenPayload = {
      id: user._id.toString(),
      email: user.email,
      role: user.role,
      name: user.name
    };

    const accessToken = jwt.sign(
      tokenPayload,
      process.env.JWT_SECRET,
      { expiresIn: '15m' }
    );

    const refreshToken = jwt.sign(
      { id: user._id.toString() },
      process.env.JWT_REFRESH_SECRET,
      { expiresIn: rememberMe ? '30d' : '7d' }
    );

    // ✅ Store refresh token in database
    await db.collection('refreshTokens').insertOne({
      userId: user._id,
      token: refreshToken,
      createdAt: new Date(),
      expiresAt: new Date(Date.now() + (rememberMe ? 30 : 7) * 24 * 60 * 60 * 1000),
      userAgent: req.headers['user-agent'],
      ipAddress: req.headers['x-forwarded-for'] || req.connection.remoteAddress
    });

    // ✅ Set HTTP-only cookies
    setCookie('accessToken', accessToken, {
      req,
      res,
      maxAge: 15 * 60, // 15 minutes
      httpOnly: true,
      secure: process.env.NODE_ENV === 'production',
      sameSite: 'strict'
    });

    setCookie('refreshToken', refreshToken, {
      req,
      res,
      maxAge: (rememberMe ? 30 : 7) * 24 * 60 * 60, // 7 or 30 days
      httpOnly: true,
      secure: process.env.NODE_ENV === 'production',
      sameSite: 'strict'
    });

    // ✅ Update last login
    await db.collection('users').updateOne(
      { _id: user._id },
      { 
        $set: { 
          lastLoginAt: new Date(),
          lastLoginIP: req.headers['x-forwarded-for'] || req.connection.remoteAddress
        } 
      }
    );

    // ✅ Return user data (without password)
    const { password: _, ...userResponse } = user;
    
    res.status(200).json({
      message: 'Login successful',
      user: userResponse,
      accessToken // Optional: also return in response body
    });
  } catch (error) {
    console.error('Login error:', error);
    res.status(500).json({
      error: 'Internal Server Error',
      message: 'Something went wrong during login'
    });
  }
}
```

### **📝 Register Endpoint**

```javascript
// pages/api/auth/register.js
import { connectToDatabase } from '@/lib/mongodb';
import bcrypt from 'bcryptjs';
import { validateRegister } from '@/lib/validation';
import { sendVerificationEmail } from '@/lib/email';
import { generateVerificationToken } from '@/lib/tokens';

export default async function handler(req, res) {
  if (req.method !== 'POST') {
    return res.status(405).json({ 
      error: 'Method Not Allowed',
      message: 'Only POST method is allowed'
    });
  }

  try {
    // ✅ Validate request body
    const validation = validateRegister(req.body);
    if (!validation.isValid) {
      return res.status(400).json({
        error: 'Validation Error',
        details: validation.errors
      });
    }

    const { name, email, password } = req.body;
    const { db } = await connectToDatabase();

    // ✅ Check if user already exists
    const existingUser = await db.collection('users').findOne({ 
      email: email.toLowerCase() 
    });

    if (existingUser) {
      return res.status(409).json({
        error: 'User already exists',
        message: 'A user with this email already exists'
      });
    }

    // ✅ Hash password
    const hashedPassword = await bcrypt.hash(password, 12);

    // ✅ Generate verification token
    const verificationToken = generateVerificationToken();

    // ✅ Create user
    const newUser = {
      name: name.trim(),
      email: email.toLowerCase().trim(),
      password: hashedPassword,
      role: 'user',
      isActive: true,
      emailVerified: false,
      verificationToken,
      verificationTokenExpires: new Date(Date.now() + 24 * 60 * 60 * 1000), // 24 hours
      createdAt: new Date(),
      updatedAt: new Date()
    };

    const result = await db.collection('users').insertOne(newUser);

    // ✅ Send verification email
    try {
      await sendVerificationEmail(email, name, verificationToken);
    } catch (emailError) {
      console.error('Failed to send verification email:', emailError);
      // Continue with registration even if email fails
    }

    // ✅ Remove sensitive data from response
    const { password: _, verificationToken: __, ...userResponse } = newUser;
    userResponse._id = result.insertedId;

    res.status(201).json({
      message: 'Registration successful',
      user: userResponse,
      notice: 'Please check your email to verify your account'
    });
  } catch (error) {
    console.error('Registration error:', error);
    res.status(500).json({
      error: 'Internal Server Error',
      message: 'Something went wrong during registration'
    });
  }
}
```

### **🔄 Token Refresh Endpoint**

```javascript
// pages/api/auth/refresh.js
import { connectToDatabase } from '@/lib/mongodb';
import jwt from 'jsonwebtoken';
import { getCookie, setCookie, deleteCookie } from 'cookies-next';
import { ObjectId } from 'mongodb';

export default async function handler(req, res) {
  if (req.method !== 'POST') {
    return res.status(405).json({ 
      error: 'Method Not Allowed',
      message: 'Only POST method is allowed'
    });
  }

  try {
    // ✅ Get refresh token from cookie
    const refreshToken = getCookie('refreshToken', { req, res });
    
    if (!refreshToken) {
      return res.status(401).json({
        error: 'No refresh token',
        message: 'Refresh token is required'
      });
    }

    // ✅ Verify refresh token
    let decoded;
    try {
      decoded = jwt.verify(refreshToken, process.env.JWT_REFRESH_SECRET);
    } catch (jwtError) {
      // Clear invalid cookies
      deleteCookie('accessToken', { req, res });
      deleteCookie('refreshToken', { req, res });
      
      return res.status(401).json({
        error: 'Invalid refresh token',
        message: 'Refresh token is invalid or expired'
      });
    }

    const { db } = await connectToDatabase();

    // ✅ Check if refresh token exists in database
    const tokenRecord = await db.collection('refreshTokens').findOne({
      token: refreshToken,
      userId: new ObjectId(decoded.id),
      expiresAt: { $gt: new Date() }
    });

    if (!tokenRecord) {
      // Clear cookies
      deleteCookie('accessToken', { req, res });
      deleteCookie('refreshToken', { req, res });
      
      return res.status(401).json({
        error: 'Invalid refresh token',
        message: 'Refresh token not found or expired'
      });
    }

    // ✅ Get user data
    const user = await db.collection('users').findOne({
      _id: new ObjectId(decoded.id),
      isActive: true
    });

    if (!user) {
      // Clean up invalid token
      await db.collection('refreshTokens').deleteOne({ _id: tokenRecord._id });
      deleteCookie('accessToken', { req, res });
      deleteCookie('refreshToken', { req, res });
      
      return res.status(401).json({
        error: 'User not found',
        message: 'User associated with token not found'
      });
    }

    // ✅ Generate new access token
    const tokenPayload = {
      id: user._id.toString(),
      email: user.email,
      role: user.role,
      name: user.name
    };

    const newAccessToken = jwt.sign(
      tokenPayload,
      process.env.JWT_SECRET,
      { expiresIn: '15m' }
    );

    // ✅ Set new access token cookie
    setCookie('accessToken', newAccessToken, {
      req,
      res,
      maxAge: 15 * 60, // 15 minutes
      httpOnly: true,
      secure: process.env.NODE_ENV === 'production',
      sameSite: 'strict'
    });

    // ✅ Update token last used
    await db.collection('refreshTokens').updateOne(
      { _id: tokenRecord._id },
      { $set: { lastUsedAt: new Date() } }
    );

    res.status(200).json({
      message: 'Token refreshed successfully',
      accessToken: newAccessToken,
      user: {
        id: user._id,
        name: user.name,
        email: user.email,
        role: user.role
      }
    });
  } catch (error) {
    console.error('Token refresh error:', error);
    res.status(500).json({
      error: 'Internal Server Error',
      message: 'Something went wrong during token refresh'
    });
  }
}
```

## 🛡️ Middleware

### **🔐 Authentication Middleware**

```javascript
// middleware/auth.js
import jwt from 'jsonwebtoken';
import { getCookie } from 'cookies-next';
import { connectToDatabase } from '@/lib/mongodb';
import { ObjectId } from 'mongodb';

export async function requireAuth(req, res, roles = []) {
  try {
    // ✅ Get token from cookie or header
    let token = getCookie('accessToken', { req, res });
    
    if (!token && req.headers.authorization) {
      token = req.headers.authorization.replace('Bearer ', '');
    }

    if (!token) {
      res.status(401).json({
        error: 'Authentication required',
        message: 'No access token provided'
      });
      return { user: null };
    }

    // ✅ Verify JWT token
    let decoded;
    try {
      decoded = jwt.verify(token, process.env.JWT_SECRET);
    } catch (jwtError) {
      res.status(401).json({
        error: 'Invalid token',
        message: 'Access token is invalid or expired'
      });
      return { user: null };
    }

    // ✅ Get user from database
    const { db } = await connectToDatabase();
    const user = await db.collection('users').findOne({
      _id: new ObjectId(decoded.id),
      isActive: true
    });

    if (!user) {
      res.status(401).json({
        error: 'User not found',
        message: 'User associated with token not found'
      });
      return { user: null };
    }

    // ✅ Check role permissions
    if (roles.length > 0 && !roles.includes(user.role)) {
      res.status(403).json({
        error: 'Insufficient permissions',
        message: 'You do not have permission to access this resource'
      });
      return { user: null };
    }

    return { 
      user: {
        id: user._id.toString(),
        name: user.name,
        email: user.email,
        role: user.role
      }
    };
  } catch (error) {
    console.error('Auth middleware error:', error);
    res.status(500).json({
      error: 'Authentication error',
      message: 'Something went wrong during authentication'
    });
    return { user: null };
  }
}

// ✅ Role-based middleware
export const requireAdmin = (handler) => {
  return async (req, res) => {
    const authResult = await requireAuth(req, res, ['admin']);
    if (!authResult.user) return; // Response already sent
    
    req.user = authResult.user;
    return handler(req, res);
  };
};

export const requireUser = (handler) => {
  return async (req, res) => {
    const authResult = await requireAuth(req, res);
    if (!authResult.user) return; // Response already sent
    
    req.user = authResult.user;
    return handler(req, res);
  };
};
```

### **📊 Request Logging Middleware**

```javascript
// middleware/logging.js
export function withLogging(handler) {
  return async (req, res) => {
    const startTime = Date.now();
    const { method, url, headers } = req;
    
    // ✅ Log request
    console.log(`[${new Date().toISOString()}] ${method} ${url} - Start`);
    
    // ✅ Override res.json to log response
    const originalJson = res.json;
    res.json = function(data) {
      const duration = Date.now() - startTime;
      const statusCode = res.statusCode;
      
      console.log(
        `[${new Date().toISOString()}] ${method} ${url} - ${statusCode} - ${duration}ms`
      );
      
      // ✅ Log errors
      if (statusCode >= 400) {
        console.error(`Error response:`, data);
      }
      
      return originalJson.call(this, data);
    };

    try {
      return await handler(req, res);
    } catch (error) {
      const duration = Date.now() - startTime;
      console.error(
        `[${new Date().toISOString()}] ${method} ${url} - ERROR - ${duration}ms`,
        error
      );
      
      if (!res.headersSent) {
        res.status(500).json({
          error: 'Internal Server Error',
          message: 'Something went wrong'
        });
      }
    }
  };
}
```

### **🚀 Rate Limiting Middleware**

```javascript
// middleware/rateLimit.js
const rateLimitStore = new Map();

export function withRateLimit(options = {}) {
  const {
    windowMs = 15 * 60 * 1000, // 15 minutes
    maxRequests = 100,
    message = 'Too many requests, please try again later',
    skipSuccessfulRequests = false,
    keyGenerator = (req) => req.headers['x-forwarded-for'] || req.connection.remoteAddress
  } = options;

  return function(handler) {
    return async (req, res) => {
      const key = keyGenerator(req);
      const now = Date.now();
      
      // ✅ Clean up old entries
      for (const [k, data] of rateLimitStore.entries()) {
        if (now - data.resetTime > windowMs) {
          rateLimitStore.delete(k);
        }
      }
      
      // ✅ Get or create rate limit data
      let rateLimitData = rateLimitStore.get(key);
      if (!rateLimitData || now - rateLimitData.resetTime > windowMs) {
        rateLimitData = {
          count: 0,
          resetTime: now
        };
        rateLimitStore.set(key, rateLimitData);
      }
      
      // ✅ Check rate limit
      if (rateLimitData.count >= maxRequests) {
        const timeUntilReset = Math.ceil((windowMs - (now - rateLimitData.resetTime)) / 1000);
        
        res.status(429).json({
          error: 'Rate limit exceeded',
          message,
          retryAfter: timeUntilReset
        });
        return;
      }
      
      // ✅ Increment counter
      rateLimitData.count++;
      
      // ✅ Add rate limit headers
      res.setHeader('X-RateLimit-Limit', maxRequests);
      res.setHeader('X-RateLimit-Remaining', Math.max(0, maxRequests - rateLimitData.count));
      res.setHeader('X-RateLimit-Reset', new Date(rateLimitData.resetTime + windowMs).toISOString());
      
      try {
        const result = await handler(req, res);
        
        // ✅ Optionally skip counting successful requests
        if (skipSuccessfulRequests && res.statusCode < 400) {
          rateLimitData.count--;
        }
        
        return result;
      } catch (error) {
        throw error;
      }
    };
  };
}

// ✅ Usage examples
export const strictRateLimit = withRateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  maxRequests: 5,
  message: 'Too many login attempts, please try again later'
});

export const apiRateLimit = withRateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  maxRequests: 100,
  skipSuccessfulRequests: true
});
```

## ✅ Input Validation

### **🔍 Validation Library**

```javascript
// lib/validation.js
import Joi from 'joi';

// ✅ User validation schemas
export const userValidationSchema = Joi.object({
  name: Joi.string()
    .trim()
    .min(2)
    .max(50)
    .required()
    .messages({
      'string.empty': 'Name is required',
      'string.min': 'Name must be at least 2 characters',
      'string.max': 'Name cannot exceed 50 characters'
    }),
  
  email: Joi.string()
    .email()
    .lowercase()
    .required()
    .messages({
      'string.email': 'Please provide a valid email address',
      'string.empty': 'Email is required'
    }),
  
  password: Joi.string()
    .min(8)
    .pattern(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]/)
    .required()
    .messages({
      'string.min': 'Password must be at least 8 characters',
      'string.pattern.base': 'Password must contain at least one uppercase letter, one lowercase letter, one number, and one special character',
      'string.empty': 'Password is required'
    }),
  
  role: Joi.string()
    .valid('user', 'admin', 'moderator')
    .default('user')
});

export const userUpdateSchema = userValidationSchema.fork(
  ['password'], 
  (schema) => schema.optional()
);

export const loginSchema = Joi.object({
  email: Joi.string()
    .email()
    .required()
    .messages({
      'string.email': 'Please provide a valid email address',
      'string.empty': 'Email is required'
    }),
  
  password: Joi.string()
    .required()
    .messages({
      'string.empty': 'Password is required'
    }),
  
  rememberMe: Joi.boolean()
    .default(false)
});

// ✅ Validation functions
export function validateUser(data) {
  const { error, value } = userValidationSchema.validate(data, {
    abortEarly: false,
    stripUnknown: true
  });
  
  return {
    isValid: !error,
    errors: error?.details.map(detail => ({
      field: detail.path.join('.'),
      message: detail.message
    })) || [],
    data: value
  };
}

export function validateUserUpdate(data) {
  const { error, value } = userUpdateSchema.validate(data, {
    abortEarly: false,
    stripUnknown: true
  });
  
  return {
    isValid: !error,
    errors: error?.details.map(detail => ({
      field: detail.path.join('.'),
      message: detail.message
    })) || [],
    data: value
  };
}

export function validateLogin(data) {
  const { error, value } = loginSchema.validate(data, {
    abortEarly: false,
    stripUnknown: true
  });
  
  return {
    isValid: !error,
    errors: error?.details.map(detail => ({
      field: detail.path.join('.'),
      message: detail.message
    })) || [],
    data: value
  };
}

// ✅ Sanitization functions
export function sanitizeUser(user) {
  const { password, verificationToken, ...sanitized } = user;
  return {
    ...sanitized,
    id: user._id?.toString() || user.id
  };
}
```

## 📝 แบบฝึกหัด

### **🎯 แบบฝึกหัดที่ 1: Basic API Routes**
1. สร้าง CRUD API สำหรับ products
2. เพิ่ม pagination และ filtering
3. เพิ่ม input validation
4. ทดสอบด้วย Postman หรือ Thunder Client

### **🎯 แบบฝึกหัดที่ 2: Authentication System**
1. สร้าง complete auth system (login, register, logout)
2. เพิ่ม JWT token management
3. สร้าง password reset functionality
4. เพิ่ม email verification

### **🎯 แบบฝึกหัดที่ 3: Advanced Features**
1. เพิ่ม role-based authorization
2. สร้าง rate limiting middleware
3. เพิ่ม request logging
4. สร้าง API documentation

### **🎯 แบบฝึกหัดที่ 4: File Upload API**
1. สร้าง file upload endpoint
2. เพิ่ม image resizing
3. เพิ่ม file type validation
4. เพิ่ม cloud storage integration

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 6: Data Fetching Basics](./06-data-fetching-basics.md)

**บทต่อไป**: [บทที่ 8: State Management](./08-state-management.md)

**กลับไปหน้าหลัก**: [README](./README.md)

---

## 📚 เอกสารอ้างอิง

- [Next.js API Routes](https://nextjs.org/docs/api-routes/introduction)
- [Next.js Authentication](https://nextjs.org/docs/authentication)
- [JWT.io](https://jwt.io/)
- [Joi Validation](https://joi.dev/)
