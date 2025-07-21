# บทที่ 10: Authentication

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ Authentication patterns ใน Next.js
- ทำความเข้าใจ NextAuth.js integration
- เรียนรู้ Custom authentication implementation
- เข้าใจ JWT และ Session management

## 🔐 Authentication Overview

### **📊 Authentication Methods**

```javascript
// ✅ Authentication Methods Comparison
const authMethods = {
  session: {
    storage: 'Server-side (database/memory)',
    security: 'High',
    scalability: 'Moderate',
    stateful: true,
    useCase: 'Traditional web apps'
  },
  jwt: {
    storage: 'Client-side (localStorage/cookies)',
    security: 'Moderate-High',
    scalability: 'High',
    stateful: false,
    useCase: 'SPAs, mobile apps, microservices'
  },
  oauth: {
    storage: 'Third-party providers',
    security: 'High',
    scalability: 'High',
    stateful: false,
    useCase: 'Social logins, SSO'
  }
};
```

## 🔑 NextAuth.js Integration

### **📦 NextAuth.js Setup**

```bash
npm install next-auth
npm install @next-auth/prisma-adapter prisma @prisma/client
```

```javascript
// pages/api/auth/[...nextauth].js
import NextAuth from 'next-auth';
import GoogleProvider from 'next-auth/providers/google';
import GithubProvider from 'next-auth/providers/github';
import CredentialsProvider from 'next-auth/providers/credentials';
import { PrismaAdapter } from "@next-auth/prisma-adapter";
import { prisma } from '@/lib/prisma';
import bcrypt from 'bcryptjs';

export default NextAuth({
  // ✅ Database adapter
  adapter: PrismaAdapter(prisma),
  
  // ✅ Authentication providers
  providers: [
    // Google OAuth
    GoogleProvider({
      clientId: process.env.GOOGLE_CLIENT_ID,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET,
      authorization: {
        params: {
          prompt: "consent",
          access_type: "offline",
          response_type: "code"
        }
      }
    }),
    
    // GitHub OAuth
    GithubProvider({
      clientId: process.env.GITHUB_ID,
      clientSecret: process.env.GITHUB_SECRET,
    }),
    
    // Email/Password credentials
    CredentialsProvider({
      name: "credentials",
      credentials: {
        email: { 
          label: "Email", 
          type: "email", 
          placeholder: "your-email@example.com" 
        },
        password: { 
          label: "Password", 
          type: "password" 
        }
      },
      async authorize(credentials, req) {
        try {
          if (!credentials?.email || !credentials?.password) {
            throw new Error('Missing credentials');
          }

          // ✅ Find user in database
          const user = await prisma.user.findUnique({
            where: { email: credentials.email.toLowerCase() },
            include: { accounts: true }
          });

          if (!user) {
            throw new Error('No user found with this email');
          }

          // ✅ Check if user has a password (not OAuth-only)
          if (!user.password) {
            throw new Error('Please use OAuth to sign in');
          }

          // ✅ Verify password
          const isValidPassword = await bcrypt.compare(
            credentials.password, 
            user.password
          );

          if (!isValidPassword) {
            throw new Error('Invalid password');
          }

          // ✅ Check if email is verified
          if (!user.emailVerified) {
            throw new Error('Please verify your email before signing in');
          }

          // ✅ Check if account is active
          if (!user.isActive) {
            throw new Error('Your account has been deactivated');
          }

          // ✅ Update last login
          await prisma.user.update({
            where: { id: user.id },
            data: { lastLoginAt: new Date() }
          });

          return {
            id: user.id,
            email: user.email,
            name: user.name,
            image: user.avatar,
            role: user.role
          };
        } catch (error) {
          console.error('Auth error:', error);
          throw new Error(error.message);
        }
      }
    })
  ],

  // ✅ Pages configuration
  pages: {
    signIn: '/auth/signin',
    signUp: '/auth/signup',
    error: '/auth/error',
    verifyRequest: '/auth/verify-request',
    newUser: '/auth/new-user'
  },

  // ✅ Session configuration
  session: {
    strategy: "jwt",
    maxAge: 30 * 24 * 60 * 60, // 30 days
    updateAge: 24 * 60 * 60, // 24 hours
  },

  // ✅ JWT configuration
  jwt: {
    maxAge: 30 * 24 * 60 * 60, // 30 days
    secret: process.env.NEXTAUTH_SECRET,
  },

  // ✅ Callbacks
  callbacks: {
    async jwt({ token, user, account, profile }) {
      // ✅ First time JWT created (sign in)
      if (user) {
        token.role = user.role;
        token.id = user.id;
      }

      // ✅ Handle OAuth account linking
      if (account && profile) {
        // Update user profile from OAuth provider
        await prisma.user.update({
          where: { id: token.id },
          data: {
            name: profile.name || token.name,
            image: profile.picture || profile.avatar_url || token.picture,
            lastLoginAt: new Date()
          }
        });
      }

      return token;
    },

    async session({ session, token }) {
      // ✅ Add custom fields to session
      if (token) {
        session.user.id = token.id;
        session.user.role = token.role;
      }

      // ✅ Get fresh user data from database
      const user = await prisma.user.findUnique({
        where: { id: token.id },
        select: {
          id: true,
          name: true,
          email: true,
          image: true,
          role: true,
          isActive: true,
          preferences: true
        }
      });

      if (user) {
        session.user = { ...session.user, ...user };
      }

      return session;
    },

    async signIn({ user, account, profile, email, credentials }) {
      try {
        // ✅ Allow OAuth sign-ins
        if (account?.provider !== 'credentials') {
          return true;
        }

        // ✅ Credentials provider already handled in authorize
        return true;
      } catch (error) {
        console.error('SignIn callback error:', error);
        return false;
      }
    },

    async redirect({ url, baseUrl }) {
      // ✅ Allows relative callback URLs
      if (url.startsWith("/")) return `${baseUrl}${url}`;
      
      // ✅ Allows callback URLs on the same origin
      else if (new URL(url).origin === baseUrl) return url;
      
      return baseUrl;
    }
  },

  // ✅ Events
  events: {
    async signIn({ user, account, profile, isNewUser }) {
      console.log(`User ${user.email} signed in via ${account.provider}`);
      
      // ✅ Track sign-in analytics
      if (process.env.NODE_ENV === 'production') {
        // Send to analytics service
      }
    },
    
    async signOut({ session, token }) {
      console.log(`User ${session?.user?.email} signed out`);
    },
    
    async createUser({ user }) {
      console.log(`New user created: ${user.email}`);
      
      // ✅ Send welcome email
      // await sendWelcomeEmail(user.email, user.name);
    }
  },

  // ✅ Debug mode for development
  debug: process.env.NODE_ENV === 'development',

  // ✅ Secret for JWT encryption
  secret: process.env.NEXTAUTH_SECRET,
});
```

### **🎨 Custom Sign-In Page**

```jsx
// pages/auth/signin.js
import { getSession, getProviders, signIn, getCsrfToken } from 'next-auth/react';
import { useState } from 'react';
import { useRouter } from 'next/router';
import Link from 'next/link';
import Head from 'next/head';

export default function SignIn({ providers, csrfToken }) {
  const router = useRouter();
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState('');
  const [formData, setFormData] = useState({
    email: '',
    password: ''
  });

  const { error: urlError } = router.query;

  const handleSubmit = async (e) => {
    e.preventDefault();
    setLoading(true);
    setError('');

    try {
      const result = await signIn('credentials', {
        email: formData.email,
        password: formData.password,
        redirect: false,
        callbackUrl: router.query.callbackUrl || '/'
      });

      if (result?.error) {
        setError(result.error);
      } else if (result?.ok) {
        router.push(result.url);
      }
    } catch (error) {
      setError('An unexpected error occurred');
    } finally {
      setLoading(false);
    }
  };

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
  };

  const getErrorMessage = (error) => {
    const errorMessages = {
      'Signin': 'Try signing in with a different account.',
      'OAuthSignin': 'Try signing in with a different account.',
      'OAuthCallback': 'Try signing in with a different account.',
      'OAuthCreateAccount': 'Try signing in with a different account.',
      'EmailCreateAccount': 'Try signing in with a different account.',
      'Callback': 'Try signing in with a different account.',
      'OAuthAccountNotLinked': 'To confirm your identity, sign in with the same account you used originally.',
      'EmailSignin': 'Check your email address.',
      'CredentialsSignin': 'Sign in failed. Check the details you provided are correct.',
      'default': 'Unable to sign in.'
    };
    
    return errorMessages[error] || errorMessages.default;
  };

  return (
    <>
      <Head>
        <title>Sign In - My App</title>
        <meta name="description" content="Sign in to your account" />
      </Head>

      <div className="min-h-screen flex items-center justify-center bg-gray-50 py-12 px-4 sm:px-6 lg:px-8">
        <div className="max-w-md w-full space-y-8">
          <div>
            <h2 className="mt-6 text-center text-3xl font-extrabold text-gray-900">
              Sign in to your account
            </h2>
            <p className="mt-2 text-center text-sm text-gray-600">
              Or{' '}
              <Link href="/auth/signup" className="font-medium text-indigo-600 hover:text-indigo-500">
                create a new account
              </Link>
            </p>
          </div>

          {/* Error Display */}
          {(error || urlError) && (
            <div className="bg-red-50 border border-red-200 rounded-md p-4">
              <div className="text-red-800 text-sm">
                {error || getErrorMessage(urlError)}
              </div>
            </div>
          )}

          {/* OAuth Providers */}
          <div className="space-y-3">
            {Object.values(providers)
              .filter(provider => provider.id !== 'credentials')
              .map((provider) => (
                <button
                  key={provider.name}
                  onClick={() => signIn(provider.id, { callbackUrl: router.query.callbackUrl })}
                  className="w-full flex justify-center items-center px-4 py-2 border border-gray-300 rounded-md shadow-sm bg-white text-sm font-medium text-gray-700 hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
                >
                  {provider.id === 'google' && (
                    <svg className="w-5 h-5 mr-2" viewBox="0 0 24 24">
                      <path fill="#4285F4" d="M22.56 12.25c0-.78-.07-1.53-.2-2.25H12v4.26h5.92c-.26 1.37-1.04 2.53-2.21 3.31v2.77h3.57c2.08-1.92 3.28-4.74 3.28-8.09z"/>
                      <path fill="#34A853" d="M12 23c2.97 0 5.46-.98 7.28-2.66l-3.57-2.77c-.98.66-2.23 1.06-3.71 1.06-2.86 0-5.29-1.93-6.16-4.53H2.18v2.84C3.99 20.53 7.7 23 12 23z"/>
                      <path fill="#FBBC05" d="M5.84 14.09c-.22-.66-.35-1.36-.35-2.09s.13-1.43.35-2.09V7.07H2.18C1.43 8.55 1 10.22 1 12s.43 3.45 1.18 4.93l2.85-2.22.81-.62z"/>
                      <path fill="#EA4335" d="M12 5.38c1.62 0 3.06.56 4.21 1.64l3.15-3.15C17.45 2.09 14.97 1 12 1 7.7 1 3.99 3.47 2.18 7.07l3.66 2.84c.87-2.6 3.3-4.53 6.16-4.53z"/>
                    </svg>
                  )}
                  {provider.id === 'github' && (
                    <svg className="w-5 h-5 mr-2" fill="currentColor" viewBox="0 0 24 24">
                      <path d="M12 0C5.37 0 0 5.37 0 12c0 5.31 3.435 9.795 8.205 11.385.6.105.825-.255.825-.57 0-.285-.015-1.23-.015-2.235-3.015.555-3.795-.735-4.035-1.41-.135-.345-.72-1.41-1.23-1.695-.42-.225-1.02-.78-.015-.795.945-.015 1.62.87 1.845 1.23 1.08 1.815 2.805 1.305 3.495.99.105-.78.42-1.305.765-1.605-2.67-.3-5.46-1.335-5.46-5.925 0-1.305.465-2.385 1.23-3.225-.12-.3-.54-1.53.12-3.18 0 0 1.005-.315 3.3 1.23.96-.27 1.98-.405 3-.405s2.04.135 3 .405c2.295-1.56 3.3-1.23 3.3-1.23.66 1.65.24 2.88.12 3.18.765.84 1.23 1.905 1.23 3.225 0 4.605-2.805 5.625-5.475 5.925.435.375.81 1.095.81 2.22 0 1.605-.015 2.895-.015 3.3 0 .315.225.69.825.57A12.02 12.02 0 0024 12c0-6.63-5.37-12-12-12z"/>
                    </svg>
                  )}
                  Sign in with {provider.name}
                </button>
              ))}
          </div>

          <div className="relative">
            <div className="absolute inset-0 flex items-center">
              <div className="w-full border-t border-gray-300" />
            </div>
            <div className="relative flex justify-center text-sm">
              <span className="px-2 bg-gray-50 text-gray-500">Or continue with</span>
            </div>
          </div>

          {/* Credentials Form */}
          <form className="mt-8 space-y-6" onSubmit={handleSubmit}>
            <input name="csrfToken" type="hidden" defaultValue={csrfToken} />
            
            <div className="space-y-4">
              <div>
                <label htmlFor="email" className="block text-sm font-medium text-gray-700">
                  Email address
                </label>
                <input
                  id="email"
                  name="email"
                  type="email"
                  autoComplete="email"
                  required
                  value={formData.email}
                  onChange={handleChange}
                  className="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm placeholder-gray-400 focus:outline-none focus:ring-indigo-500 focus:border-indigo-500"
                  placeholder="Enter your email"
                />
              </div>
              
              <div>
                <label htmlFor="password" className="block text-sm font-medium text-gray-700">
                  Password
                </label>
                <input
                  id="password"
                  name="password"
                  type="password"
                  autoComplete="current-password"
                  required
                  value={formData.password}
                  onChange={handleChange}
                  className="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm placeholder-gray-400 focus:outline-none focus:ring-indigo-500 focus:border-indigo-500"
                  placeholder="Enter your password"
                />
              </div>
            </div>

            <div className="flex items-center justify-between">
              <div className="text-sm">
                <Link href="/auth/forgot-password" className="font-medium text-indigo-600 hover:text-indigo-500">
                  Forgot your password?
                </Link>
              </div>
            </div>

            <div>
              <button
                type="submit"
                disabled={loading}
                className="group relative w-full flex justify-center py-2 px-4 border border-transparent text-sm font-medium rounded-md text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 disabled:opacity-50 disabled:cursor-not-allowed"
              >
                {loading ? (
                  <div className="flex items-center">
                    <svg className="animate-spin -ml-1 mr-3 h-5 w-5 text-white" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                      <circle className="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" strokeWidth="4"></circle>
                      <path className="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                    </svg>
                    Signing in...
                  </div>
                ) : (
                  'Sign in'
                )}
              </button>
            </div>
          </form>
        </div>
      </div>
    </>
  );
}

export async function getServerSideProps(context) {
  const session = await getSession(context);

  // ✅ Redirect if already signed in
  if (session) {
    return {
      redirect: {
        destination: '/',
        permanent: false,
      },
    };
  }

  const providers = await getProviders();
  const csrfToken = await getCsrfToken(context);

  return {
    props: {
      providers: providers ?? [],
      csrfToken,
    },
  };
}
```

### **📱 useSession Hook Usage**

```jsx
// components/UserProfile.js
import { useSession, signIn, signOut } from 'next-auth/react';
import { useState } from 'react';
import Image from 'next/image';

export default function UserProfile() {
  const { data: session, status, update } = useSession();
  const [updating, setUpdating] = useState(false);

  if (status === 'loading') {
    return (
      <div className="flex items-center space-x-2">
        <div className="animate-spin rounded-full h-6 w-6 border-b-2 border-gray-900"></div>
        <span>Loading...</span>
      </div>
    );
  }

  if (status === 'unauthenticated') {
    return (
      <div className="text-center">
        <p className="mb-4">You are not signed in</p>
        <button
          onClick={() => signIn()}
          className="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
        >
          Sign In
        </button>
      </div>
    );
  }

  const updateProfile = async (newData) => {
    setUpdating(true);
    try {
      // ✅ Update user in database
      const response = await fetch('/api/user/profile', {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(newData)
      });

      if (response.ok) {
        // ✅ Update NextAuth session
        await update({
          ...session,
          user: {
            ...session.user,
            ...newData
          }
        });
      }
    } catch (error) {
      console.error('Profile update failed:', error);
    } finally {
      setUpdating(false);
    }
  };

  return (
    <div className="max-w-md mx-auto bg-white rounded-xl shadow-md overflow-hidden">
      <div className="p-6">
        <div className="flex items-center space-x-4">
          {session.user.image && (
            <Image
              src={session.user.image}
              alt={session.user.name}
              width={60}
              height={60}
              className="rounded-full"
            />
          )}
          <div>
            <h2 className="text-xl font-bold">{session.user.name}</h2>
            <p className="text-gray-600">{session.user.email}</p>
            <span className="inline-block px-2 py-1 text-xs bg-blue-100 text-blue-800 rounded-full">
              {session.user.role}
            </span>
          </div>
        </div>

        <div className="mt-6 space-y-4">
          <div>
            <label className="block text-sm font-medium text-gray-700">
              Display Name
            </label>
            <input
              type="text"
              defaultValue={session.user.name}
              className="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md"
              onBlur={(e) => {
                if (e.target.value !== session.user.name) {
                  updateProfile({ name: e.target.value });
                }
              }}
            />
          </div>

          <div className="flex space-x-4">
            <button
              onClick={() => updateProfile({ 
                preferences: { 
                  ...session.user.preferences, 
                  theme: session.user.preferences?.theme === 'dark' ? 'light' : 'dark' 
                } 
              })}
              disabled={updating}
              className="flex-1 bg-gray-500 hover:bg-gray-700 text-white font-bold py-2 px-4 rounded"
            >
              Toggle Theme
            </button>
            
            <button
              onClick={() => signOut()}
              className="flex-1 bg-red-500 hover:bg-red-700 text-white font-bold py-2 px-4 rounded"
            >
              Sign Out
            </button>
          </div>
        </div>
      </div>
    </div>
  );
}

// pages/_app.js
import { SessionProvider } from 'next-auth/react';

export default function App({
  Component,
  pageProps: { session, ...pageProps },
}) {
  return (
    <SessionProvider session={session}>
      <Component {...pageProps} />
    </SessionProvider>
  );
}
```

## 🛡️ Custom Authentication

### **🔐 JWT Authentication Implementation**

```javascript
// lib/auth.js
import jwt from 'jsonwebtoken';
import bcrypt from 'bcryptjs';
import { prisma } from '@/lib/prisma';

const JWT_SECRET = process.env.JWT_SECRET;
const JWT_REFRESH_SECRET = process.env.JWT_REFRESH_SECRET;

export class AuthService {
  // ✅ Generate JWT tokens
  static generateTokens(payload) {
    const accessToken = jwt.sign(payload, JWT_SECRET, {
      expiresIn: '15m'
    });

    const refreshToken = jwt.sign(
      { userId: payload.id },
      JWT_REFRESH_SECRET,
      { expiresIn: '30d' }
    );

    return { accessToken, refreshToken };
  }

  // ✅ Verify JWT token
  static verifyToken(token, secret = JWT_SECRET) {
    try {
      return jwt.verify(token, secret);
    } catch (error) {
      throw new Error('Invalid token');
    }
  }

  // ✅ Hash password
  static async hashPassword(password) {
    const salt = await bcrypt.genSalt(12);
    return bcrypt.hash(password, salt);
  }

  // ✅ Compare password
  static async comparePassword(password, hashedPassword) {
    return bcrypt.compare(password, hashedPassword);
  }

  // ✅ Register user
  static async register({ name, email, password }) {
    try {
      // Check if user exists
      const existingUser = await prisma.user.findUnique({
        where: { email: email.toLowerCase() }
      });

      if (existingUser) {
        throw new Error('User already exists');
      }

      // Hash password
      const hashedPassword = await this.hashPassword(password);

      // Create user
      const user = await prisma.user.create({
        data: {
          name,
          email: email.toLowerCase(),
          password: hashedPassword,
          emailVerified: false
        },
        select: {
          id: true,
          name: true,
          email: true,
          role: true,
          createdAt: true
        }
      });

      // Generate tokens
      const { accessToken, refreshToken } = this.generateTokens({
        id: user.id,
        email: user.email,
        role: user.role
      });

      // Store refresh token
      await this.storeRefreshToken(user.id, refreshToken);

      return { user, accessToken, refreshToken };
    } catch (error) {
      throw error;
    }
  }

  // ✅ Login user
  static async login({ email, password }) {
    try {
      // Find user
      const user = await prisma.user.findUnique({
        where: { 
          email: email.toLowerCase(),
          isActive: true
        },
        select: {
          id: true,
          name: true,
          email: true,
          password: true,
          role: true,
          emailVerified: true
        }
      });

      if (!user) {
        throw new Error('Invalid credentials');
      }

      // Verify password
      const isValidPassword = await this.comparePassword(password, user.password);
      if (!isValidPassword) {
        throw new Error('Invalid credentials');
      }

      // Check email verification
      if (!user.emailVerified) {
        throw new Error('Please verify your email');
      }

      // Generate tokens
      const { accessToken, refreshToken } = this.generateTokens({
        id: user.id,
        email: user.email,
        role: user.role
      });

      // Store refresh token
      await this.storeRefreshToken(user.id, refreshToken);

      // Update last login
      await prisma.user.update({
        where: { id: user.id },
        data: { lastLoginAt: new Date() }
      });

      const { password: _, ...userWithoutPassword } = user;
      return { user: userWithoutPassword, accessToken, refreshToken };
    } catch (error) {
      throw error;
    }
  }

  // ✅ Refresh token
  static async refreshToken(refreshToken) {
    try {
      // Verify refresh token
      const decoded = this.verifyToken(refreshToken, JWT_REFRESH_SECRET);

      // Check if refresh token exists in database
      const storedToken = await prisma.refreshToken.findFirst({
        where: {
          token: refreshToken,
          userId: decoded.userId,
          expiresAt: { gt: new Date() }
        },
        include: { user: true }
      });

      if (!storedToken) {
        throw new Error('Invalid refresh token');
      }

      // Generate new tokens
      const { accessToken, refreshToken: newRefreshToken } = this.generateTokens({
        id: storedToken.user.id,
        email: storedToken.user.email,
        role: storedToken.user.role
      });

      // Replace old refresh token
      await prisma.refreshToken.update({
        where: { id: storedToken.id },
        data: {
          token: newRefreshToken,
          expiresAt: new Date(Date.now() + 30 * 24 * 60 * 60 * 1000) // 30 days
        }
      });

      return { accessToken, refreshToken: newRefreshToken };
    } catch (error) {
      throw error;
    }
  }

  // ✅ Store refresh token
  static async storeRefreshToken(userId, refreshToken) {
    return prisma.refreshToken.create({
      data: {
        token: refreshToken,
        userId,
        expiresAt: new Date(Date.now() + 30 * 24 * 60 * 60 * 1000) // 30 days
      }
    });
  }

  // ✅ Revoke refresh token
  static async revokeRefreshToken(refreshToken) {
    return prisma.refreshToken.deleteMany({
      where: { token: refreshToken }
    });
  }

  // ✅ Logout (revoke all tokens for user)
  static async logout(userId) {
    return prisma.refreshToken.deleteMany({
      where: { userId }
    });
  }

  // ✅ Get user by token
  static async getUserByToken(token) {
    try {
      const decoded = this.verifyToken(token);
      
      const user = await prisma.user.findUnique({
        where: { 
          id: decoded.id,
          isActive: true
        },
        select: {
          id: true,
          name: true,
          email: true,
          role: true,
          avatar: true,
          preferences: true
        }
      });

      if (!user) {
        throw new Error('User not found');
      }

      return user;
    } catch (error) {
      throw error;
    }
  }
}

// middleware/authMiddleware.js
import { AuthService } from '@/lib/auth';

export function withAuth(roles = []) {
  return async (req, res, next) => {
    try {
      // Get token from header
      const authHeader = req.headers.authorization;
      if (!authHeader || !authHeader.startsWith('Bearer ')) {
        return res.status(401).json({ message: 'Access token required' });
      }

      const token = authHeader.substring(7);
      
      // Verify token and get user
      const user = await AuthService.getUserByToken(token);
      
      // Check role permissions
      if (roles.length > 0 && !roles.includes(user.role)) {
        return res.status(403).json({ message: 'Insufficient permissions' });
      }

      // Add user to request
      req.user = user;
      
      if (next) {
        return next();
      }
    } catch (error) {
      return res.status(401).json({ message: error.message });
    }
  };
}

// Higher-order function for API routes
export function requireAuth(handler, roles = []) {
  return async (req, res) => {
    const authMiddleware = withAuth(roles);
    
    return new Promise((resolve, reject) => {
      authMiddleware(req, res, (error) => {
        if (error) {
          return reject(error);
        }
        return handler(req, res).then(resolve).catch(reject);
      });
    });
  };
}
```

### **🔒 Protected Routes**

```javascript
// pages/api/auth/register.js
import { AuthService } from '@/lib/auth';
import { validateRegistration } from '@/lib/validation';

export default async function handler(req, res) {
  if (req.method !== 'POST') {
    return res.status(405).json({ message: 'Method not allowed' });
  }

  try {
    // Validate input
    const validation = validateRegistration(req.body);
    if (!validation.isValid) {
      return res.status(400).json({
        message: 'Validation failed',
        errors: validation.errors
      });
    }

    // Register user
    const { user, accessToken, refreshToken } = await AuthService.register(req.body);

    // Set HTTP-only cookies
    res.setHeader('Set-Cookie', [
      `accessToken=${accessToken}; HttpOnly; Secure; SameSite=Strict; Max-Age=900; Path=/`,
      `refreshToken=${refreshToken}; HttpOnly; Secure; SameSite=Strict; Max-Age=2592000; Path=/`
    ]);

    res.status(201).json({
      message: 'Registration successful',
      user,
      accessToken
    });
  } catch (error) {
    res.status(400).json({ message: error.message });
  }
}

// pages/api/auth/login.js
import { AuthService } from '@/lib/auth';
import { validateLogin } from '@/lib/validation';

export default async function handler(req, res) {
  if (req.method !== 'POST') {
    return res.status(405).json({ message: 'Method not allowed' });
  }

  try {
    // Validate input
    const validation = validateLogin(req.body);
    if (!validation.isValid) {
      return res.status(400).json({
        message: 'Validation failed',
        errors: validation.errors
      });
    }

    // Login user
    const { user, accessToken, refreshToken } = await AuthService.login(req.body);

    // Set HTTP-only cookies
    res.setHeader('Set-Cookie', [
      `accessToken=${accessToken}; HttpOnly; Secure; SameSite=Strict; Max-Age=900; Path=/`,
      `refreshToken=${refreshToken}; HttpOnly; Secure; SameSite=Strict; Max-Age=2592000; Path=/`
    ]);

    res.status(200).json({
      message: 'Login successful',
      user,
      accessToken
    });
  } catch (error) {
    res.status(401).json({ message: error.message });
  }
}

// pages/api/user/profile.js
import { requireAuth } from '@/middleware/authMiddleware';
import { prisma } from '@/lib/prisma';

async function handler(req, res) {
  switch (req.method) {
    case 'GET':
      return getProfile(req, res);
    case 'PUT':
      return updateProfile(req, res);
    default:
      return res.status(405).json({ message: 'Method not allowed' });
  }
}

async function getProfile(req, res) {
  try {
    const user = await prisma.user.findUnique({
      where: { id: req.user.id },
      select: {
        id: true,
        name: true,
        email: true,
        avatar: true,
        role: true,
        preferences: true,
        createdAt: true,
        lastLoginAt: true
      }
    });

    res.status(200).json({ user });
  } catch (error) {
    res.status(500).json({ message: 'Failed to fetch profile' });
  }
}

async function updateProfile(req, res) {
  try {
    const { name, avatar, preferences } = req.body;

    const user = await prisma.user.update({
      where: { id: req.user.id },
      data: {
        ...(name && { name }),
        ...(avatar && { avatar }),
        ...(preferences && { preferences }),
        updatedAt: new Date()
      },
      select: {
        id: true,
        name: true,
        email: true,
        avatar: true,
        role: true,
        preferences: true
      }
    });

    res.status(200).json({
      message: 'Profile updated successfully',
      user
    });
  } catch (error) {
    res.status(500).json({ message: 'Failed to update profile' });
  }
}

export default requireAuth(handler);
```

## 📝 แบบฝึกหัด

### **🎯 แบบฝึกหัดที่ 1: NextAuth.js Setup**
1. ติดตั้งและ configure NextAuth.js
2. เพิ่ม OAuth providers (Google, GitHub)
3. สร้าง custom sign-in page
4. เพิ่ม session management

### **🎯 แบบฝึกหัดที่ 2: Custom Authentication**
1. สร้าง JWT-based authentication system
2. เพิ่ม refresh token mechanism
3. สร้าง password reset functionality
4. เพิ่ม email verification

### **🎯 แบบฝึกหัดที่ 3: Role-based Access Control**
1. เพิ่ม user roles และ permissions
2. สร้าง protected routes
3. เพิ่ม admin dashboard
4. ทดสอบ authorization

### **🎯 แบบฝึกหัดที่ 4: Advanced Security**
1. เพิ่ม two-factor authentication
2. สร้าง rate limiting
3. เพิ่ม session monitoring
4. เพิ่ม security headers

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 9: Database Integration](./09-database-integration.md)

**บทต่อไป**: [บทที่ 11: Performance Optimization](./11-performance-optimization.md)

**กลับไปหน้าหลัก**: [README](./README.md)

---

## 📚 เอกสารอ้างอิง

- [NextAuth.js Documentation](https://next-auth.js.org/)
- [JWT.io](https://jwt.io/)
- [OAuth 2.0 Specification](https://oauth.net/2/)
- [Authentication Best Practices](https://auth0.com/blog/a-look-at-the-latest-draft-for-oauth-2-security-best-current-practice/)
