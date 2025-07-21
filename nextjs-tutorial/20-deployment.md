# บทที่ 20: Deployment and DevOps

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้การ Deploy Next.js applications
- ทำความเข้าใจ CI/CD pipelines
- เรียนรู้ Docker containerization
- เข้าใจ Production optimization และ monitoring

## 🚀 Vercel Deployment

### **⚡ Zero-Config Deployment**

```bash
# Install Vercel CLI
npm install -g vercel

# Deploy to Vercel
vercel

# Deploy with custom settings
vercel --prod --name my-nextjs-app
```

```javascript
// vercel.json - Vercel configuration
{
  "version": 2,
  "name": "my-nextjs-app",
  "alias": ["myapp.vercel.app", "myapp.com"],
  "env": {
    "CUSTOM_KEY": "value"
  },
  "build": {
    "env": {
      "NEXT_PUBLIC_API_URL": "https://api.myapp.com"
    }
  },
  "functions": {
    "pages/api/**/*.js": {
      "maxDuration": 30
    }
  },
  "routes": [
    {
      "src": "/health",
      "dest": "/api/health"
    },
    {
      "src": "/admin/(.*)",
      "dest": "/admin/$1",
      "headers": {
        "cache-control": "no-cache"
      }
    }
  ],
  "redirects": [
    {
      "source": "/old-blog/:slug",
      "destination": "/blog/:slug",
      "permanent": true
    }
  ],
  "rewrites": [
    {
      "source": "/api/proxy/:path*",
      "destination": "https://external-api.com/:path*"
    }
  ],
  "headers": [
    {
      "source": "/api/(.*)",
      "headers": [
        {
          "key": "Access-Control-Allow-Origin",
          "value": "*"
        },
        {
          "key": "Access-Control-Allow-Methods",
          "value": "GET, POST, PUT, DELETE, OPTIONS"
        }
      ]
    }
  ],
  "regions": ["sfo1", "bru1"],
  "github": {
    "enabled": true,
    "autoAlias": true
  }
}

// next.config.js - Production optimizations
/** @type {import('next').NextConfig} */
const nextConfig = {
  // ✅ Output configuration
  output: 'standalone', // For Docker builds
  
  // ✅ Performance optimizations
  compress: true,
  poweredByHeader: false,
  generateEtags: true,
  
  // ✅ Build optimizations
  swcMinify: true,
  compiler: {
    removeConsole: process.env.NODE_ENV === 'production'
  },
  
  // ✅ Image optimization
  images: {
    formats: ['image/webp', 'image/avif'],
    minimumCacheTTL: 60,
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],
    domains: ['example.com', 'cdn.myapp.com'],
    remotePatterns: [
      {
        protocol: 'https',
        hostname: '**.amazonaws.com',
        pathname: '/images/**',
      },
    ],
    dangerouslyAllowSVG: false,
    contentSecurityPolicy: "default-src 'self'; script-src 'none'; sandbox;"
  },
  
  // ✅ Security headers
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: [
          {
            key: 'X-Frame-Options',
            value: 'DENY'
          },
          {
            key: 'X-Content-Type-Options',
            value: 'nosniff'
          },
          {
            key: 'Referrer-Policy',
            value: 'strict-origin-when-cross-origin'
          },
          {
            key: 'Content-Security-Policy',
            value: "default-src 'self'; script-src 'self' 'unsafe-eval'; object-src 'none';"
          }
        ]
      }
    ];
  },
  
  // ✅ Redirects
  async redirects() {
    return [
      {
        source: '/old-home',
        destination: '/',
        permanent: true
      }
    ];
  },
  
  // ✅ Rewrites
  async rewrites() {
    return [
      {
        source: '/api/external/:path*',
        destination: 'https://external-api.com/:path*'
      }
    ];
  },
  
  // ✅ Environment variables
  env: {
    CUSTOM_KEY: process.env.CUSTOM_KEY,
  },
  
  // ✅ Experimental features
  experimental: {
    appDir: true,
    serverComponentsExternalPackages: ['@prisma/client'],
    optimizeCss: true,
    gzipSize: true,
  },
  
  // ✅ Webpack customization
  webpack: (config, { buildId, dev, isServer, defaultLoaders, webpack }) => {
    // Bundle analyzer
    if (process.env.ANALYZE === 'true') {
      const { BundleAnalyzerPlugin } = require('webpack-bundle-analyzer');
      config.plugins.push(
        new BundleAnalyzerPlugin({
          analyzerMode: 'static',
          openAnalyzer: false,
        })
      );
    }
    
    return config;
  },
};

module.exports = nextConfig;

// Environment configuration
// .env.local
NEXT_PUBLIC_APP_URL=http://localhost:3000
DATABASE_URL=postgresql://user:password@localhost:5432/myapp
NEXTAUTH_SECRET=your-secret-key
NEXTAUTH_URL=http://localhost:3000

// .env.production
NEXT_PUBLIC_APP_URL=https://myapp.vercel.app
DATABASE_URL=postgresql://user:password@prod-db:5432/myapp
NEXTAUTH_SECRET=production-secret-key
NEXTAUTH_URL=https://myapp.vercel.app
```

### **🔧 Advanced Vercel Features**

```javascript
// pages/api/edge/hello.js - Edge function
export const config = {
  runtime: 'edge',
};

export default function handler(req) {
  const { searchParams } = new URL(req.url);
  const name = searchParams.get('name') || 'World';
  
  return new Response(
    JSON.stringify({
      message: `Hello ${name}!`,
      timestamp: new Date().toISOString(),
      region: process.env.VERCEL_REGION || 'unknown'
    }),
    {
      status: 200,
      headers: {
        'Content-Type': 'application/json',
        'Cache-Control': 's-maxage=60'
      }
    }
  );
}

// middleware.js - Edge middleware
import { NextResponse } from 'next/server';

export function middleware(request) {
  // ✅ Geolocation-based routing
  const country = request.geo?.country || 'US';
  const region = request.geo?.region || 'unknown';
  
  // ✅ A/B testing
  const bucket = Math.random() < 0.5 ? 'A' : 'B';
  
  const response = NextResponse.next();
  
  // ✅ Add custom headers
  response.headers.set('x-country', country);
  response.headers.set('x-region', region);
  response.headers.set('x-ab-bucket', bucket);
  
  // ✅ Feature flags
  if (country === 'US' && bucket === 'B') {
    response.headers.set('x-feature-flag', 'new-checkout');
  }
  
  return response;
}

export const config = {
  matcher: [
    '/((?!api|_next/static|_next/image|favicon.ico).*)',
  ],
};

// lib/analytics/vercel.js - Vercel Analytics
import { Analytics } from '@vercel/analytics/react';
import { SpeedInsights } from '@vercel/speed-insights/next';

export function VercelAnalytics() {
  return (
    <>
      <Analytics />
      <SpeedInsights />
    </>
  );
}

// Add to app/layout.js
import { VercelAnalytics } from '@/lib/analytics/vercel';

export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body>
        {children}
        <VercelAnalytics />
      </body>
    </html>
  );
}
```

## 🐳 Docker Deployment

### **📦 Containerization**

```dockerfile
# Dockerfile
FROM node:18-alpine AS base

# Install dependencies only when needed
FROM base AS deps
# Check https://github.com/nodejs/docker-node/tree/b4117f9333da4138b03a546ec926ef50a31506c3#nodealpine to understand why libc6-compat might be needed.
RUN apk add --no-cache libc6-compat
WORKDIR /app

# Install dependencies based on the preferred package manager
COPY package.json yarn.lock* package-lock.json* pnpm-lock.yaml* ./
RUN \
  if [ -f yarn.lock ]; then yarn --frozen-lockfile; \
  elif [ -f package-lock.json ]; then npm ci; \
  elif [ -f pnpm-lock.yaml ]; then yarn global add pnpm && pnpm i --frozen-lockfile; \
  else echo "Lockfile not found." && exit 1; \
  fi


# Rebuild the source code only when needed
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

# Next.js collects completely anonymous telemetry data about general usage.
# Learn more here: https://nextjs.org/telemetry
# Uncomment the following line in case you want to disable telemetry during the build.
ENV NEXT_TELEMETRY_DISABLED 1

# Build the application
RUN yarn build

# If using npm comment out above and use below instead
# RUN npm run build

# Production image, copy all the files and run next
FROM base AS runner
WORKDIR /app

ENV NODE_ENV production
# Uncomment the following line in case you want to disable telemetry during runtime.
ENV NEXT_TELEMETRY_DISABLED 1

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public

# Set the correct permission for prerender cache
RUN mkdir .next
RUN chown nextjs:nodejs .next

# Automatically leverage output traces to reduce image size
# https://nextjs.org/docs/advanced-features/output-file-tracing
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV PORT 3000
# set hostname to localhost
ENV HOSTNAME "0.0.0.0"

# server.js is created by next build from the standalone output
# https://nextjs.org/docs/pages/api-reference/next-config-js/output
CMD ["node", "server.js"]

# docker-compose.yml - Development setup
version: '3.8'

services:
  # Next.js application
  nextjs:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://postgres:password@postgres:5432/myapp
      - NEXTAUTH_SECRET=development-secret
      - NEXTAUTH_URL=http://localhost:3000
    depends_on:
      - postgres
      - redis
    volumes:
      - ./uploads:/app/uploads
    networks:
      - app-network

  # PostgreSQL database
  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./scripts/init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - app-network

  # Redis for caching
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    networks:
      - app-network

  # Nginx reverse proxy
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
    depends_on:
      - nextjs
    networks:
      - app-network

volumes:
  postgres_data:
  redis_data:

networks:
  app-network:
    driver: bridge

# docker-compose.prod.yml - Production setup
version: '3.8'

services:
  nextjs:
    build:
      context: .
      dockerfile: Dockerfile
      target: runner
    restart: unless-stopped
    environment:
      - NODE_ENV=production
      - DATABASE_URL=${DATABASE_URL}
      - NEXTAUTH_SECRET=${NEXTAUTH_SECRET}
      - NEXTAUTH_URL=${NEXTAUTH_URL}
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.nextjs.rule=Host(`myapp.com`)"
      - "traefik.http.routers.nextjs.tls=true"
      - "traefik.http.routers.nextjs.tls.certresolver=letsencrypt"
    networks:
      - app-network

  traefik:
    image: traefik:v3.0
    command:
      - "--api.dashboard=true"
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
      - "--certificatesresolvers.letsencrypt.acme.tlschallenge=true"
      - "--certificatesresolvers.letsencrypt.acme.email=admin@myapp.com"
      - "--certificatesresolvers.letsencrypt.acme.storage=/letsencrypt/acme.json"
    ports:
      - "80:80"
      - "443:443"
      - "8080:8080"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
      - "./letsencrypt:/letsencrypt"
    networks:
      - app-network

networks:
  app-network:
    external: true
```

### **🛠️ Build Scripts**

```json
// package.json - Build scripts
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:e2e": "playwright test",
    "type-check": "tsc --noEmit",
    "docker:build": "docker build -t myapp:latest .",
    "docker:run": "docker run -p 3000:3000 myapp:latest",
    "docker:compose": "docker-compose up -d",
    "docker:compose:prod": "docker-compose -f docker-compose.prod.yml up -d",
    "analyze": "ANALYZE=true npm run build",
    "export": "next export",
    "deploy:vercel": "vercel --prod",
    "deploy:netlify": "netlify deploy --prod",
    "health-check": "curl -f http://localhost:3000/api/health || exit 1"
  }
}

# Makefile - Deployment automation
.PHONY: build test deploy clean

# Variables
IMAGE_NAME = myapp
VERSION = $(shell git rev-parse --short HEAD)
REGISTRY = myregistry.com

# Build application
build:
	@echo "Building Next.js application..."
	npm run build
	
# Build Docker image
docker-build:
	@echo "Building Docker image..."
	docker build -t $(IMAGE_NAME):$(VERSION) .
	docker tag $(IMAGE_NAME):$(VERSION) $(IMAGE_NAME):latest

# Run tests
test:
	@echo "Running tests..."
	npm run test
	npm run test:e2e

# Type checking
type-check:
	@echo "Running type check..."
	npm run type-check

# Linting
lint:
	@echo "Running linter..."
	npm run lint

# Security audit
audit:
	@echo "Running security audit..."
	npm audit
	
# Build and test
ci: type-check lint test docker-build
	@echo "CI pipeline completed successfully"

# Deploy to staging
deploy-staging:
	@echo "Deploying to staging..."
	docker-compose -f docker-compose.staging.yml up -d

# Deploy to production
deploy-prod:
	@echo "Deploying to production..."
	docker-compose -f docker-compose.prod.yml up -d

# Health check
health-check:
	@echo "Running health check..."
	./scripts/health-check.sh

# Clean up
clean:
	@echo "Cleaning up..."
	docker system prune -f
	rm -rf .next
	rm -rf node_modules/.cache

# scripts/health-check.sh
#!/bin/bash
set -e

echo "Starting health check..."

# Check if application is running
if curl -f http://localhost:3000/api/health > /dev/null 2>&1; then
    echo "✅ Application is healthy"
else
    echo "❌ Application health check failed"
    exit 1
fi

# Check database connection
if curl -f http://localhost:3000/api/health/db > /dev/null 2>&1; then
    echo "✅ Database connection is healthy"
else
    echo "❌ Database health check failed"
    exit 1
fi

# Check Redis connection
if curl -f http://localhost:3000/api/health/redis > /dev/null 2>&1; then
    echo "✅ Redis connection is healthy"
else
    echo "❌ Redis health check failed"
    exit 1
fi

echo "All health checks passed!"
```

## 🔄 CI/CD Pipelines

### **🐙 GitHub Actions**

```yaml
# .github/workflows/ci.yml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  NODE_VERSION: '18'
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # Code quality checks
  quality:
    name: Code Quality
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: ${{ env.NODE_VERSION }}
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Type checking
      run: npm run type-check

    - name: Linting
      run: npm run lint

    - name: Security audit
      run: npm audit --audit-level moderate

    - name: Check for vulnerabilities
      run: |
        npx audit-ci --moderate
        npx better-npm-audit audit

  # Unit and integration tests
  test:
    name: Run Tests
    runs-on: ubuntu-latest
    needs: quality
    
    strategy:
      matrix:
        node-version: [16, 18, 20]
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Setup Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v4
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Run unit tests
      run: npm run test -- --coverage --watchAll=false

    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage/lcov.info
        flags: unittests
        name: codecov-umbrella

  # E2E tests
  e2e:
    name: E2E Tests
    runs-on: ubuntu-latest
    needs: quality
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: ${{ env.NODE_VERSION }}
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Install Playwright browsers
      run: npx playwright install --with-deps

    - name: Build application
      run: npm run build

    - name: Start application
      run: npm start &
      
    - name: Wait for application
      run: npx wait-on http://localhost:3000

    - name: Run E2E tests
      run: npm run test:e2e

    - name: Upload test results
      uses: actions/upload-artifact@v3
      if: always()
      with:
        name: playwright-report
        path: playwright-report/
        retention-days: 30

  # Build and push Docker image
  build:
    name: Build Docker Image
    runs-on: ubuntu-latest
    needs: [test, e2e]
    if: github.ref == 'refs/heads/main'
    
    permissions:
      contents: read
      packages: write

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3

    - name: Log in to Container Registry
      uses: docker/login-action@v3
      with:
        registry: ${{ env.REGISTRY }}
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}

    - name: Extract metadata
      id: meta
      uses: docker/metadata-action@v5
      with:
        images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
        tags: |
          type=ref,event=branch
          type=ref,event=pr
          type=sha,prefix={{branch}}-
          type=raw,value=latest,enable={{is_default_branch}}

    - name: Build and push Docker image
      uses: docker/build-push-action@v5
      with:
        context: .
        platforms: linux/amd64,linux/arm64
        push: true
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}
        cache-from: type=gha
        cache-to: type=gha,mode=max

  # Deploy to staging
  deploy-staging:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/develop'
    environment: staging
    
    steps:
    - name: Deploy to staging
      run: |
        echo "Deploying to staging environment..."
        # Add your staging deployment commands here

  # Deploy to production
  deploy-production:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/main'
    environment: production
    
    steps:
    - name: Deploy to production
      run: |
        echo "Deploying to production environment..."
        # Add your production deployment commands here

# .github/workflows/deploy-vercel.yml
name: Deploy to Vercel

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '18'
        cache: 'npm'

    - name: Install Vercel CLI
      run: npm install --global vercel@latest

    - name: Pull Vercel Environment Information
      run: vercel pull --yes --environment=production --token=${{ secrets.VERCEL_TOKEN }}

    - name: Build Project Artifacts
      run: vercel build --prod --token=${{ secrets.VERCEL_TOKEN }}

    - name: Deploy Project Artifacts to Vercel
      run: vercel deploy --prebuilt --prod --token=${{ secrets.VERCEL_TOKEN }}

# .github/workflows/security.yml
name: Security Scanning

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]
  schedule:
    - cron: '0 2 * * 1' # Weekly on Mondays

jobs:
  dependency-check:
    name: Dependency Security Check
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Run Snyk to check for vulnerabilities
      uses: snyk/actions/node@master
      env:
        SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
      with:
        args: --severity-threshold=high

    - name: Upload result to GitHub Code Scanning
      uses: github/codeql-action/upload-sarif@v2
      with:
        sarif_file: snyk.sarif

  code-scanning:
    name: Code Security Analysis
    runs-on: ubuntu-latest
    permissions:
      actions: read
      contents: read
      security-events: write

    steps:
    - name: Checkout repository
      uses: actions/checkout@v4

    - name: Initialize CodeQL
      uses: github/codeql-action/init@v2
      with:
        languages: javascript

    - name: Autobuild
      uses: github/codeql-action/autobuild@v2

    - name: Perform CodeQL Analysis
      uses: github/codeql-action/analyze@v2
```

### **🌊 GitLab CI/CD**

```yaml
# .gitlab-ci.yml
stages:
  - quality
  - test
  - build
  - deploy

variables:
  NODE_VERSION: "18"
  DOCKER_DRIVER: overlay2
  DOCKER_TLS_CERTDIR: "/certs"

# Templates
.node_template: &node_template
  image: node:${NODE_VERSION}-alpine
  cache:
    key: ${CI_COMMIT_REF_SLUG}
    paths:
      - node_modules/
      - .npm/
  before_script:
    - npm ci --cache .npm --prefer-offline

# Quality checks
lint:
  <<: *node_template
  stage: quality
  script:
    - npm run lint
    - npm run type-check
  artifacts:
    reports:
      junit: junit.xml

audit:
  <<: *node_template
  stage: quality
  script:
    - npm audit --audit-level moderate
  allow_failure: false

# Tests
unit-test:
  <<: *node_template
  stage: test
  script:
    - npm run test -- --coverage --watchAll=false
  artifacts:
    reports:
      junit: junit.xml
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml
    paths:
      - coverage/

e2e-test:
  <<: *node_template
  stage: test
  services:
    - name: postgres:15-alpine
      alias: postgres
  variables:
    POSTGRES_DB: testdb
    POSTGRES_USER: test
    POSTGRES_PASSWORD: test
    DATABASE_URL: "postgresql://test:test@postgres:5432/testdb"
  script:
    - npm run build
    - npm start &
    - sleep 10
    - npm run test:e2e
  artifacts:
    when: always
    paths:
      - playwright-report/
    expire_in: 1 week

# Build
build:
  stage: build
  image: docker:24.0.5
  services:
    - docker:24.0.5-dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
    - docker tag $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA $CI_REGISTRY_IMAGE:latest
    - docker push $CI_REGISTRY_IMAGE:latest
  only:
    - main
    - develop

# Deploy to staging
deploy-staging:
  stage: deploy
  image: alpine:latest
  before_script:
    - apk add --no-cache curl
  script:
    - echo "Deploying to staging..."
    - curl -X POST "$STAGING_WEBHOOK_URL" -H "Authorization: Bearer $STAGING_TOKEN"
  environment:
    name: staging
    url: https://staging.myapp.com
  only:
    - develop

# Deploy to production
deploy-production:
  stage: deploy
  image: alpine:latest
  before_script:
    - apk add --no-cache curl
  script:
    - echo "Deploying to production..."
    - curl -X POST "$PRODUCTION_WEBHOOK_URL" -H "Authorization: Bearer $PRODUCTION_TOKEN"
  environment:
    name: production
    url: https://myapp.com
  when: manual
  only:
    - main
```

## 📊 Monitoring & Health Checks

### **🔍 Application Monitoring**

```javascript
// pages/api/health.js - Health check endpoint
export default async function handler(req, res) {
  const healthcheck = {
    uptime: process.uptime(),
    message: 'OK',
    timestamp: Date.now(),
    version: process.env.npm_package_version,
    environment: process.env.NODE_ENV,
  };

  try {
    // Check database connection
    await prisma.$queryRaw`SELECT 1`;
    healthcheck.database = 'connected';

    // Check Redis connection (if using Redis)
    if (redis) {
      await redis.ping();
      healthcheck.redis = 'connected';
    }

    // Check external services
    const externalChecks = await Promise.allSettled([
      fetch('https://api.external-service.com/health', { 
        timeout: 5000 
      }),
      // Add more external service checks
    ]);

    healthcheck.external = externalChecks.map((result, index) => ({
      service: `external-${index}`,
      status: result.status === 'fulfilled' ? 'healthy' : 'unhealthy'
    }));

    res.status(200).json(healthcheck);
  } catch (error) {
    healthcheck.message = error.message;
    healthcheck.database = 'disconnected';
    res.status(503).json(healthcheck);
  }
}

// lib/monitoring/metrics.js - Application metrics
import { register, Counter, Histogram, Gauge } from 'prom-client';

// HTTP request metrics
export const httpRequestsTotal = new Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'route', 'status_code'],
});

export const httpRequestDuration = new Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route'],
  buckets: [0.1, 0.5, 1, 2, 5],
});

// Database metrics
export const dbQueryDuration = new Histogram({
  name: 'db_query_duration_seconds',
  help: 'Duration of database queries in seconds',
  labelNames: ['operation'],
  buckets: [0.01, 0.05, 0.1, 0.5, 1],
});

export const dbConnectionsActive = new Gauge({
  name: 'db_connections_active',
  help: 'Number of active database connections',
});

// Business metrics
export const userRegistrations = new Counter({
  name: 'user_registrations_total',
  help: 'Total number of user registrations',
});

export const ordersTotal = new Counter({
  name: 'orders_total',
  help: 'Total number of orders',
  labelNames: ['status'],
});

// pages/api/metrics.js - Prometheus metrics endpoint
import { register } from 'prom-client';

export default async function handler(req, res) {
  if (req.method !== 'GET') {
    res.setHeader('Allow', ['GET']);
    return res.status(405).end(`Method ${req.method} Not Allowed`);
  }

  try {
    const metrics = await register.metrics();
    res.setHeader('Content-Type', register.contentType);
    res.status(200).send(metrics);
  } catch (error) {
    res.status(500).json({ error: 'Failed to generate metrics' });
  }
}

// middleware/monitoring.js - Request monitoring middleware
import { httpRequestsTotal, httpRequestDuration } from '@/lib/monitoring/metrics';

export function monitoringMiddleware(req, res, next) {
  const start = Date.now();
  
  // Track request
  const labels = {
    method: req.method,
    route: req.route?.path || req.url,
  };

  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;
    
    // Record metrics
    httpRequestsTotal.inc({
      ...labels,
      status_code: res.statusCode,
    });
    
    httpRequestDuration.observe(labels, duration);
  });

  next();
}

// lib/monitoring/logger.js - Structured logging
import winston from 'winston';

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  defaultMeta: {
    service: 'nextjs-app',
    version: process.env.npm_package_version,
    environment: process.env.NODE_ENV,
  },
  transports: [
    new winston.transports.File({ 
      filename: 'logs/error.log', 
      level: 'error' 
    }),
    new winston.transports.File({ 
      filename: 'logs/combined.log' 
    }),
  ],
});

if (process.env.NODE_ENV !== 'production') {
  logger.add(new winston.transports.Console({
    format: winston.format.simple()
  }));
}

// Add request ID tracking
logger.addRequestId = (req, res, next) => {
  req.requestId = req.headers['x-request-id'] || 
                  Math.random().toString(36).substr(2, 9);
  res.setHeader('x-request-id', req.requestId);
  
  // Add request ID to all subsequent logs
  req.logger = logger.child({ requestId: req.requestId });
  
  next();
};

export default logger;
```

## 📝 แบบฝึกหัด

### **🎯 แบบฝึกหัดที่ 1: Vercel Deployment**
1. Deploy Next.js app ไปยัง Vercel
2. ตั้งค่า custom domain
3. เพิ่ม environment variables
4. ทดสอบ edge functions

### **🎯 แบบฝึกหัดที่ 2: Docker Containerization**
1. สร้าง Dockerfile สำหรับ Next.js app
2. ตั้งค่า docker-compose
3. เพิ่ม reverse proxy ด้วย Nginx
4. ทดสอบ multi-stage builds

### **🎯 แบบฝึกหัดที่ 3: CI/CD Pipeline**
1. ตั้งค่า GitHub Actions workflow
2. เพิ่ม automated testing
3. สร้าง deployment stages
4. เพิ่ม security scanning

### **🎯 แบบฝึกหัดที่ 4: Monitoring Setup**
1. เพิ่ม health check endpoints
2. ตั้งค่า metrics collection
3. สร้าง monitoring dashboard
4. เพิ่ม alerting rules

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 19: Testing](./19-testing.md)

**บทต่อไป**: [บทที่ 21: Monitoring and Analytics](./21-monitoring-analytics.md)

**กลับไปหน้าหลัก**: [README](./README.md)

---

## 📚 เอกสารอ้างอิง

- [Vercel Deployment](https://vercel.com/docs/concepts/deployments/overview)
- [Next.js Docker](https://nextjs.org/docs/deployment#docker-image)
- [GitHub Actions](https://docs.github.com/en/actions)
- [Prometheus Monitoring](https://prometheus.io/docs/introduction/overview/)
