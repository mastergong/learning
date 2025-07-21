# บทที่ 13: App Router

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ Next.js 13+ App Router
- ทำความเข้าใจ File-based routing ใหม่
- เรียนรู้ Layout และ Template system
- เข้าใจ Route Groups และ Parallel Routes

## 🚀 App Router Overview

### **📁 App Directory Structure**

```
app/
├── layout.js          # Root layout
├── page.js            # Home page
├── loading.js         # Loading UI
├── error.js           # Error UI
├── not-found.js       # 404 page
├── global-error.js    # Global error boundary
├── favicon.ico        # Favicon
├── sitemap.xml        # Sitemap
├── robots.txt         # Robots.txt
├── manifest.json      # Web app manifest
├── (auth)/            # Route group
│   ├── login/
│   │   └── page.js
│   └── register/
│       └── page.js
├── blog/
│   ├── layout.js      # Blog layout
│   ├── page.js        # Blog index
│   ├── [slug]/
│   │   ├── page.js    # Dynamic blog post
│   │   └── loading.js
│   └── category/
│       └── [category]/
│           └── page.js
├── dashboard/
│   ├── layout.js
│   ├── page.js
│   ├── @analytics/    # Parallel route
│   │   ├── page.js
│   │   └── loading.js
│   ├── @charts/       # Parallel route
│   │   ├── page.js
│   │   └── loading.js
│   └── settings/
│       ├── page.js
│       └── profile/
│           └── page.js
└── api/               # API routes
    ├── auth/
    │   └── route.js
    └── users/
        ├── route.js
        └── [id]/
            └── route.js
```

### **🏠 Root Layout**

```jsx
// app/layout.js
import { Inter } from 'next/font/google';
import './globals.css';
import { Providers } from './providers';
import { Navigation } from '@/components/Navigation';
import { Footer } from '@/components/Footer';

const inter = Inter({ subsets: ['latin'] });

export const metadata = {
  title: {
    default: 'My Next.js App',
    template: '%s | My Next.js App'
  },
  description: 'A comprehensive Next.js application with App Router',
  keywords: ['Next.js', 'React', 'JavaScript', 'TypeScript'],
  authors: [{ name: 'Your Name', url: 'https://yourwebsite.com' }],
  creator: 'Your Name',
  publisher: 'Your Company',
  openGraph: {
    type: 'website',
    locale: 'en_US',
    url: 'https://yourwebsite.com',
    title: 'My Next.js App',
    description: 'A comprehensive Next.js application with App Router',
    siteName: 'My Next.js App',
    images: [
      {
        url: 'https://yourwebsite.com/og-image.jpg',
        width: 1200,
        height: 630,
        alt: 'My Next.js App'
      }
    ]
  },
  twitter: {
    card: 'summary_large_image',
    title: 'My Next.js App',
    description: 'A comprehensive Next.js application with App Router',
    creator: '@yourtwitterhandle',
    images: ['https://yourwebsite.com/twitter-image.jpg']
  },
  robots: {
    index: true,
    follow: true,
    googleBot: {
      index: true,
      follow: true,
      'max-video-preview': -1,
      'max-image-preview': 'large',
      'max-snippet': -1
    }
  },
  verification: {
    google: 'your-google-verification-code',
    yandex: 'your-yandex-verification-code'
  },
  alternates: {
    canonical: 'https://yourwebsite.com',
    languages: {
      'en-US': 'https://yourwebsite.com/en',
      'th-TH': 'https://yourwebsite.com/th'
    }
  }
};

export const viewport = {
  width: 'device-width',
  initialScale: 1,
  maximumScale: 1,
  userScalable: false,
  themeColor: [
    { media: '(prefers-color-scheme: light)', color: '#ffffff' },
    { media: '(prefers-color-scheme: dark)', color: '#000000' }
  ]
};

export default function RootLayout({ children }) {
  return (
    <html lang="en" suppressHydrationWarning>
      <body className={inter.className}>
        <Providers>
          <div className="min-h-screen flex flex-col">
            <Navigation />
            <main className="flex-1">
              {children}
            </main>
            <Footer />
          </div>
        </Providers>
      </body>
    </html>
  );
}

// app/providers.js
'use client';

import { SessionProvider } from 'next-auth/react';
import { ThemeProvider } from 'next-themes';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';
import { useState } from 'react';

export function Providers({ children }) {
  const [queryClient] = useState(() => new QueryClient({
    defaultOptions: {
      queries: {
        staleTime: 60 * 1000, // 1 minute
        cacheTime: 10 * 60 * 1000, // 10 minutes
        retry: 1,
        refetchOnWindowFocus: false
      }
    }
  }));

  return (
    <SessionProvider>
      <QueryClientProvider client={queryClient}>
        <ThemeProvider
          attribute="class"
          defaultTheme="system"
          enableSystem
          disableTransitionOnChange
        >
          {children}
          <ReactQueryDevtools initialIsOpen={false} />
        </ThemeProvider>
      </QueryClientProvider>
    </SessionProvider>
  );
}
```

## 📄 Pages และ Layouts

### **🎨 Nested Layouts**

```jsx
// app/blog/layout.js
import { BlogSidebar } from '@/components/BlogSidebar';
import { BlogHeader } from '@/components/BlogHeader';

export const metadata = {
  title: 'Blog',
  description: 'Read our latest articles and insights'
};

export default function BlogLayout({ children }) {
  return (
    <div className="bg-gray-50 min-h-screen">
      <BlogHeader />
      <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">
        <div className="lg:grid lg:grid-cols-12 lg:gap-8">
          {/* Sidebar */}
          <aside className="hidden lg:block lg:col-span-3">
            <BlogSidebar />
          </aside>
          
          {/* Main content */}
          <main className="lg:col-span-9">
            {children}
          </main>
        </div>
      </div>
    </div>
  );
}

// app/blog/page.js
import { BlogCard } from '@/components/BlogCard';
import { Pagination } from '@/components/Pagination';
import { prisma } from '@/lib/prisma';

export const metadata = {
  title: 'Blog Posts',
  description: 'Explore our collection of articles and insights'
};

async function getBlogPosts(page = 1, limit = 12) {
  const skip = (page - 1) * limit;
  
  const [posts, total] = await Promise.all([
    prisma.post.findMany({
      where: { published: true },
      include: {
        author: {
          select: { name: true, avatar: true }
        },
        categories: {
          select: { name: true, slug: true }
        },
        _count: {
          select: { comments: true, likes: true }
        }
      },
      orderBy: { publishedAt: 'desc' },
      skip,
      take: limit
    }),
    prisma.post.count({
      where: { published: true }
    })
  ]);

  return { posts, total, hasMore: skip + limit < total };
}

export default async function BlogPage({ searchParams }) {
  const page = Number(searchParams?.page) || 1;
  const { posts, total, hasMore } = await getBlogPosts(page);

  return (
    <div className="space-y-8">
      {/* Header */}
      <div className="text-center">
        <h1 className="text-4xl font-bold text-gray-900 sm:text-5xl">
          Latest Articles
        </h1>
        <p className="mt-4 text-xl text-gray-600">
          Discover insights, tutorials, and stories from our team
        </p>
      </div>

      {/* Featured Post */}
      {posts.length > 0 && page === 1 && (
        <div className="mb-12">
          <h2 className="text-2xl font-bold text-gray-900 mb-6">Featured Post</h2>
          <BlogCard post={posts[0]} featured />
        </div>
      )}

      {/* Posts Grid */}
      <div className="grid gap-8 md:grid-cols-2 lg:grid-cols-3">
        {posts.slice(page === 1 ? 1 : 0).map((post) => (
          <BlogCard key={post.id} post={post} />
        ))}
      </div>

      {/* Pagination */}
      {total > 12 && (
        <Pagination
          currentPage={page}
          totalPages={Math.ceil(total / 12)}
          hasMore={hasMore}
        />
      )}

      {/* Empty State */}
      {posts.length === 0 && (
        <div className="text-center py-12">
          <h3 className="text-lg font-medium text-gray-900">No posts found</h3>
          <p className="mt-2 text-gray-600">
            Check back later for new content.
          </p>
        </div>
      )}
    </div>
  );
}

// app/blog/[slug]/page.js
import { notFound } from 'next/navigation';
import { BlogContent } from '@/components/BlogContent';
import { BlogShare } from '@/components/BlogShare';
import { RelatedPosts } from '@/components/RelatedPosts';
import { CommentSection } from '@/components/CommentSection';
import { prisma } from '@/lib/prisma';

async function getPost(slug) {
  const post = await prisma.post.findUnique({
    where: {
      slug,
      published: true
    },
    include: {
      author: {
        select: {
          id: true,
          name: true,
          avatar: true,
          bio: true
        }
      },
      categories: {
        select: {
          name: true,
          slug: true
        }
      },
      tags: {
        select: {
          name: true,
          slug: true
        }
      },
      _count: {
        select: {
          comments: true,
          likes: true,
          views: true
        }
      }
    }
  });

  if (!post) {
    return null;
  }

  // Increment view count
  await prisma.post.update({
    where: { id: post.id },
    data: {
      views: {
        increment: 1
      }
    }
  });

  return post;
}

async function getRelatedPosts(postId, categoryIds) {
  return prisma.post.findMany({
    where: {
      AND: [
        { id: { not: postId } },
        { published: true },
        {
          categories: {
            some: {
              id: {
                in: categoryIds
              }
            }
          }
        }
      ]
    },
    include: {
      author: {
        select: { name: true, avatar: true }
      },
      categories: {
        select: { name: true, slug: true }
      }
    },
    take: 3,
    orderBy: { publishedAt: 'desc' }
  });
}

export async function generateMetadata({ params }) {
  const post = await getPost(params.slug);

  if (!post) {
    return {
      title: 'Post Not Found'
    };
  }

  return {
    title: post.title,
    description: post.excerpt,
    keywords: post.tags.map(tag => tag.name),
    authors: [{ name: post.author.name }],
    openGraph: {
      title: post.title,
      description: post.excerpt,
      type: 'article',
      publishedTime: post.publishedAt,
      modifiedTime: post.updatedAt,
      authors: [post.author.name],
      images: [
        {
          url: post.featuredImage,
          width: 1200,
          height: 630,
          alt: post.title
        }
      ]
    },
    twitter: {
      card: 'summary_large_image',
      title: post.title,
      description: post.excerpt,
      images: [post.featuredImage]
    }
  };
}

export default async function BlogPostPage({ params }) {
  const post = await getPost(params.slug);

  if (!post) {
    notFound();
  }

  const relatedPosts = await getRelatedPosts(
    post.id,
    post.categories.map(cat => cat.id)
  );

  const jsonLd = {
    '@context': 'https://schema.org',
    '@type': 'BlogPosting',
    headline: post.title,
    description: post.excerpt,
    image: post.featuredImage,
    datePublished: post.publishedAt,
    dateModified: post.updatedAt,
    author: {
      '@type': 'Person',
      name: post.author.name
    },
    publisher: {
      '@type': 'Organization',
      name: 'Your Blog Name',
      logo: {
        '@type': 'ImageObject',
        url: 'https://yourwebsite.com/logo.png'
      }
    }
  };

  return (
    <>
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(jsonLd) }}
      />
      
      <article className="max-w-4xl mx-auto">
        {/* Post Header */}
        <header className="mb-8">
          <div className="flex flex-wrap gap-2 mb-4">
            {post.categories.map((category) => (
              <span
                key={category.slug}
                className="px-3 py-1 text-sm bg-blue-100 text-blue-800 rounded-full"
              >
                {category.name}
              </span>
            ))}
          </div>
          
          <h1 className="text-4xl font-bold text-gray-900 mb-4">
            {post.title}
          </h1>
          
          <div className="flex items-center space-x-4 text-gray-600">
            <div className="flex items-center space-x-2">
              <img
                src={post.author.avatar}
                alt={post.author.name}
                className="w-8 h-8 rounded-full"
              />
              <span>{post.author.name}</span>
            </div>
            <time dateTime={post.publishedAt}>
              {new Date(post.publishedAt).toLocaleDateString()}
            </time>
            <span>{post._count.views} views</span>
          </div>
        </header>

        {/* Featured Image */}
        {post.featuredImage && (
          <div className="mb-8">
            <img
              src={post.featuredImage}
              alt={post.title}
              className="w-full h-64 md:h-96 object-cover rounded-lg"
            />
          </div>
        )}

        {/* Post Content */}
        <BlogContent content={post.content} />

        {/* Tags */}
        {post.tags.length > 0 && (
          <div className="mt-8 pt-8 border-t border-gray-200">
            <h3 className="text-lg font-medium text-gray-900 mb-3">Tags</h3>
            <div className="flex flex-wrap gap-2">
              {post.tags.map((tag) => (
                <span
                  key={tag.slug}
                  className="px-3 py-1 text-sm bg-gray-100 text-gray-700 rounded-full hover:bg-gray-200"
                >
                  #{tag.name}
                </span>
              ))}
            </div>
          </div>
        )}

        {/* Share */}
        <BlogShare post={post} />

        {/* Author Bio */}
        <div className="mt-8 pt-8 border-t border-gray-200">
          <div className="flex items-start space-x-4">
            <img
              src={post.author.avatar}
              alt={post.author.name}
              className="w-16 h-16 rounded-full"
            />
            <div>
              <h3 className="text-lg font-medium text-gray-900">
                {post.author.name}
              </h3>
              <p className="text-gray-600 mt-1">{post.author.bio}</p>
            </div>
          </div>
        </div>
      </article>

      {/* Related Posts */}
      {relatedPosts.length > 0 && (
        <RelatedPosts posts={relatedPosts} />
      )}

      {/* Comments */}
      <CommentSection postId={post.id} />
    </>
  );
}
```

## 🔀 Route Groups และ Parallel Routes

### **👥 Route Groups**

```jsx
// app/(auth)/layout.js - Authentication layout
export default function AuthLayout({ children }) {
  return (
    <div className="min-h-screen flex items-center justify-center bg-gray-50 py-12 px-4 sm:px-6 lg:px-8">
      <div className="max-w-md w-full space-y-8">
        <div className="text-center">
          <h2 className="mt-6 text-3xl font-extrabold text-gray-900">
            Welcome Back
          </h2>
        </div>
        {children}
      </div>
    </div>
  );
}

// app/(auth)/login/page.js
import { LoginForm } from '@/components/auth/LoginForm';

export const metadata = {
  title: 'Sign In',
  description: 'Sign in to your account'
};

export default function LoginPage() {
  return <LoginForm />;
}

// app/(auth)/register/page.js
import { RegisterForm } from '@/components/auth/RegisterForm';

export const metadata = {
  title: 'Create Account',
  description: 'Create a new account'
};

export default function RegisterPage() {
  return <RegisterForm />;
}

// app/(marketing)/layout.js - Marketing pages layout
import { MarketingHeader } from '@/components/marketing/MarketingHeader';
import { MarketingFooter } from '@/components/marketing/MarketingFooter';

export default function MarketingLayout({ children }) {
  return (
    <div className="min-h-screen">
      <MarketingHeader />
      <main>{children}</main>
      <MarketingFooter />
    </div>
  );
}

// app/(marketing)/about/page.js
export const metadata = {
  title: 'About Us',
  description: 'Learn more about our company and mission'
};

export default function AboutPage() {
  return (
    <div className="max-w-4xl mx-auto px-4 py-16">
      <h1 className="text-4xl font-bold text-center mb-8">About Us</h1>
      {/* About content */}
    </div>
  );
}
```

### **⚡ Parallel Routes**

```jsx
// app/dashboard/layout.js - Dashboard with parallel routes
export default function DashboardLayout({
  children,
  analytics,
  charts,
  recent
}) {
  return (
    <div className="min-h-screen bg-gray-50">
      <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">
        {/* Main content */}
        <div className="mb-8">
          {children}
        </div>

        {/* Parallel routes grid */}
        <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
          {/* Analytics slot */}
          <div className="lg:col-span-2">
            {analytics}
          </div>
          
          {/* Charts slot */}
          <div>
            {charts}
          </div>
        </div>

        {/* Recent activity */}
        <div className="mt-8">
          {recent}
        </div>
      </div>
    </div>
  );
}

// app/dashboard/@analytics/page.js
import { AnalyticsCard } from '@/components/dashboard/AnalyticsCard';
import { getAnalyticsData } from '@/lib/analytics';

export default async function AnalyticsSlot() {
  const analyticsData = await getAnalyticsData();

  return (
    <div className="space-y-6">
      <h2 className="text-2xl font-bold text-gray-900">Analytics Overview</h2>
      
      <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
        <AnalyticsCard
          title="Total Users"
          value={analyticsData.totalUsers}
          change={analyticsData.userGrowth}
          trend="up"
        />
        
        <AnalyticsCard
          title="Page Views"
          value={analyticsData.pageViews}
          change={analyticsData.viewGrowth}
          trend="up"
        />
        
        <AnalyticsCard
          title="Bounce Rate"
          value={`${analyticsData.bounceRate}%`}
          change={analyticsData.bounceChange}
          trend="down"
        />
        
        <AnalyticsCard
          title="Avg. Session"
          value={analyticsData.avgSession}
          change={analyticsData.sessionChange}
          trend="up"
        />
      </div>
    </div>
  );
}

// app/dashboard/@analytics/loading.js
export default function AnalyticsLoading() {
  return (
    <div className="space-y-6">
      <div className="h-8 bg-gray-200 rounded animate-pulse" />
      
      <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
        {[...Array(4)].map((_, i) => (
          <div key={i} className="bg-white p-6 rounded-lg border">
            <div className="h-4 bg-gray-200 rounded animate-pulse mb-4" />
            <div className="h-8 bg-gray-200 rounded animate-pulse mb-2" />
            <div className="h-3 bg-gray-200 rounded animate-pulse w-1/2" />
          </div>
        ))}
      </div>
    </div>
  );
}

// app/dashboard/@charts/page.js
import { ChartCard } from '@/components/dashboard/ChartCard';
import { getChartData } from '@/lib/charts';

export default async function ChartsSlot() {
  const chartData = await getChartData();

  return (
    <div className="space-y-6">
      <h2 className="text-xl font-bold text-gray-900">Performance Charts</h2>
      
      <ChartCard
        title="Revenue Trend"
        data={chartData.revenue}
        type="line"
      />
      
      <ChartCard
        title="Traffic Sources"
        data={chartData.sources}
        type="doughnut"
      />
      
      <ChartCard
        title="User Activity"
        data={chartData.activity}
        type="bar"
      />
    </div>
  );
}

// app/dashboard/@recent/page.js
import { RecentActivity } from '@/components/dashboard/RecentActivity';
import { getRecentActivity } from '@/lib/activity';

export default async function RecentSlot() {
  const activities = await getRecentActivity();

  return (
    <div className="bg-white rounded-lg border p-6">
      <h2 className="text-xl font-bold text-gray-900 mb-4">Recent Activity</h2>
      <RecentActivity activities={activities} />
    </div>
  );
}

// app/dashboard/page.js
export const metadata = {
  title: 'Dashboard',
  description: 'Your dashboard overview'
};

export default function DashboardPage() {
  return (
    <div>
      <h1 className="text-3xl font-bold text-gray-900 mb-4">Dashboard</h1>
      <p className="text-gray-600 mb-8">
        Welcome back! Here's what's happening with your account.
      </p>
    </div>
  );
}
```

## 🔗 Advanced Routing Features

### **🎯 Intercepting Routes**

```jsx
// app/@modal/(.)photo/[id]/page.js - Modal intercept
import { Modal } from '@/components/Modal';
import { PhotoViewer } from '@/components/PhotoViewer';
import { getPhoto } from '@/lib/photos';

export default async function PhotoModal({ params }) {
  const photo = await getPhoto(params.id);

  return (
    <Modal>
      <PhotoViewer photo={photo} />
    </Modal>
  );
}

// app/photo/[id]/page.js - Full page fallback
import { PhotoViewer } from '@/components/PhotoViewer';
import { getPhoto } from '@/lib/photos';
import { notFound } from 'next/navigation';

export default async function PhotoPage({ params }) {
  const photo = await getPhoto(params.id);

  if (!photo) {
    notFound();
  }

  return (
    <div className="max-w-4xl mx-auto px-4 py-8">
      <PhotoViewer photo={photo} fullPage />
    </div>
  );
}

// components/Modal.js
'use client';

import { useRouter } from 'next/navigation';
import { useEffect, useRef } from 'react';

export function Modal({ children }) {
  const router = useRouter();
  const dialogRef = useRef(null);

  useEffect(() => {
    const dialog = dialogRef.current;
    if (dialog) {
      dialog.showModal();
      
      const handleClose = () => {
        router.back();
      };

      dialog.addEventListener('close', handleClose);
      dialog.addEventListener('click', (e) => {
        if (e.target === dialog) {
          dialog.close();
        }
      });

      return () => {
        dialog.removeEventListener('close', handleClose);
      };
    }
  }, [router]);

  return (
    <dialog
      ref={dialogRef}
      className="backdrop:bg-black backdrop:bg-opacity-50 bg-transparent max-w-4xl max-h-screen p-4"
    >
      <div className="bg-white rounded-lg shadow-xl max-h-full overflow-auto">
        {children}
      </div>
    </dialog>
  );
}
```

### **🛠️ Route Handlers (API Routes)**

```javascript
// app/api/posts/route.js
import { NextRequest, NextResponse } from 'next/server';
import { prisma } from '@/lib/prisma';
import { getServerSession } from 'next-auth';
import { authOptions } from '@/lib/auth';

export async function GET(request) {
  try {
    const { searchParams } = new URL(request.url);
    const page = parseInt(searchParams.get('page') || '1');
    const limit = parseInt(searchParams.get('limit') || '10');
    const category = searchParams.get('category');
    const search = searchParams.get('search');

    const where = {
      published: true,
      ...(category && {
        categories: {
          some: {
            slug: category
          }
        }
      }),
      ...(search && {
        OR: [
          {
            title: {
              contains: search,
              mode: 'insensitive'
            }
          },
          {
            content: {
              contains: search,
              mode: 'insensitive'
            }
          }
        ]
      })
    };

    const [posts, total] = await Promise.all([
      prisma.post.findMany({
        where,
        include: {
          author: {
            select: {
              id: true,
              name: true,
              avatar: true
            }
          },
          categories: true,
          _count: {
            select: {
              comments: true,
              likes: true
            }
          }
        },
        orderBy: {
          publishedAt: 'desc'
        },
        skip: (page - 1) * limit,
        take: limit
      }),
      prisma.post.count({ where })
    ]);

    return NextResponse.json({
      posts,
      pagination: {
        page,
        limit,
        total,
        pages: Math.ceil(total / limit),
        hasNext: page * limit < total,
        hasPrev: page > 1
      }
    });
  } catch (error) {
    console.error('Posts API error:', error);
    return NextResponse.json(
      { error: 'Failed to fetch posts' },
      { status: 500 }
    );
  }
}

export async function POST(request) {
  try {
    const session = await getServerSession(authOptions);
    
    if (!session) {
      return NextResponse.json(
        { error: 'Unauthorized' },
        { status: 401 }
      );
    }

    const body = await request.json();
    const { title, content, excerpt, categoryIds, tags, featuredImage } = body;

    // Validation
    if (!title || !content) {
      return NextResponse.json(
        { error: 'Title and content are required' },
        { status: 400 }
      );
    }

    // Generate slug from title
    const slug = title
      .toLowerCase()
      .replace(/[^a-z0-9]+/g, '-')
      .replace(/(^-|-$)/g, '');

    // Check if slug exists
    const existingPost = await prisma.post.findUnique({
      where: { slug }
    });

    if (existingPost) {
      return NextResponse.json(
        { error: 'A post with this title already exists' },
        { status: 400 }
      );
    }

    // Create post
    const post = await prisma.post.create({
      data: {
        title,
        slug,
        content,
        excerpt,
        featuredImage,
        authorId: session.user.id,
        published: false,
        categories: {
          connect: categoryIds?.map(id => ({ id })) || []
        },
        tags: {
          connectOrCreate: tags?.map(tag => ({
            where: { name: tag },
            create: { name: tag, slug: tag.toLowerCase().replace(/\s+/g, '-') }
          })) || []
        }
      },
      include: {
        author: {
          select: {
            id: true,
            name: true,
            avatar: true
          }
        },
        categories: true,
        tags: true
      }
    });

    return NextResponse.json({ post }, { status: 201 });
  } catch (error) {
    console.error('Create post error:', error);
    return NextResponse.json(
      { error: 'Failed to create post' },
      { status: 500 }
    );
  }
}

// app/api/posts/[id]/route.js
export async function GET(request, { params }) {
  try {
    const post = await prisma.post.findUnique({
      where: {
        id: params.id
      },
      include: {
        author: {
          select: {
            id: true,
            name: true,
            avatar: true,
            bio: true
          }
        },
        categories: true,
        tags: true,
        comments: {
          include: {
            author: {
              select: {
                id: true,
                name: true,
                avatar: true
              }
            }
          },
          orderBy: {
            createdAt: 'desc'
          }
        },
        _count: {
          select: {
            comments: true,
            likes: true,
            views: true
          }
        }
      }
    });

    if (!post) {
      return NextResponse.json(
        { error: 'Post not found' },
        { status: 404 }
      );
    }

    return NextResponse.json({ post });
  } catch (error) {
    console.error('Get post error:', error);
    return NextResponse.json(
      { error: 'Failed to fetch post' },
      { status: 500 }
    );
  }
}

export async function PUT(request, { params }) {
  try {
    const session = await getServerSession(authOptions);
    
    if (!session) {
      return NextResponse.json(
        { error: 'Unauthorized' },
        { status: 401 }
      );
    }

    const body = await request.json();
    const { title, content, excerpt, categoryIds, tags, featuredImage, published } = body;

    // Check if user owns the post or is admin
    const existingPost = await prisma.post.findUnique({
      where: { id: params.id }
    });

    if (!existingPost) {
      return NextResponse.json(
        { error: 'Post not found' },
        { status: 404 }
      );
    }

    if (existingPost.authorId !== session.user.id && session.user.role !== 'ADMIN') {
      return NextResponse.json(
        { error: 'Forbidden' },
        { status: 403 }
      );
    }

    // Update post
    const post = await prisma.post.update({
      where: { id: params.id },
      data: {
        ...(title && { title }),
        ...(content && { content }),
        ...(excerpt && { excerpt }),
        ...(featuredImage && { featuredImage }),
        ...(published !== undefined && { 
          published,
          publishedAt: published ? new Date() : null
        }),
        ...(categoryIds && {
          categories: {
            set: [],
            connect: categoryIds.map(id => ({ id }))
          }
        }),
        ...(tags && {
          tags: {
            set: [],
            connectOrCreate: tags.map(tag => ({
              where: { name: tag },
              create: { name: tag, slug: tag.toLowerCase().replace(/\s+/g, '-') }
            }))
          }
        }),
        updatedAt: new Date()
      },
      include: {
        author: {
          select: {
            id: true,
            name: true,
            avatar: true
          }
        },
        categories: true,
        tags: true
      }
    });

    return NextResponse.json({ post });
  } catch (error) {
    console.error('Update post error:', error);
    return NextResponse.json(
      { error: 'Failed to update post' },
      { status: 500 }
    );
  }
}

export async function DELETE(request, { params }) {
  try {
    const session = await getServerSession(authOptions);
    
    if (!session) {
      return NextResponse.json(
        { error: 'Unauthorized' },
        { status: 401 }
      );
    }

    // Check if user owns the post or is admin
    const existingPost = await prisma.post.findUnique({
      where: { id: params.id }
    });

    if (!existingPost) {
      return NextResponse.json(
        { error: 'Post not found' },
        { status: 404 }
      );
    }

    if (existingPost.authorId !== session.user.id && session.user.role !== 'ADMIN') {
      return NextResponse.json(
        { error: 'Forbidden' },
        { status: 403 }
      );
    }

    await prisma.post.delete({
      where: { id: params.id }
    });

    return NextResponse.json({ message: 'Post deleted successfully' });
  } catch (error) {
    console.error('Delete post error:', error);
    return NextResponse.json(
      { error: 'Failed to delete post' },
      { status: 500 }
    );
  }
}
```

## 📝 แบบฝึกหัด

### **🎯 แบบฝึกหัดที่ 1: Basic App Router Setup**
1. สร้าง app directory structure
2. เพิ่ม root layout พร้อม metadata
3. สร้าง nested layouts
4. ทดสอบ routing

### **🎯 แบบฝึกหัดที่ 2: Route Groups**
1. สร้าง (auth) route group
2. เพิ่ม (marketing) route group
3. สร้าง different layouts สำหรับแต่ละ group
4. ทดสอบ layout isolation

### **🎯 แบบฝึกหัดที่ 3: Parallel Routes**
1. สร้าง dashboard พร้อม parallel routes
2. เพิ่ม @analytics, @charts slots
3. สร้าง loading states
4. ทดสอบ independent rendering

### **🎯 แบบฝึกหัดที่ 4: API Routes**
1. สร้าง RESTful API endpoints
2. เพิ่ม authentication middleware
3. เพิ่ม validation และ error handling
4. ทดสอบ API functionality

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 12: Forms and Validation](./12-forms-validation.md)

**บทต่อไป**: [บทที่ 14: Server Components](./14-server-components.md)

**กลับไปหน้าหลัก**: [README](./README.md)

---

## 📚 เอกสารอ้างอิง

- [Next.js App Router](https://nextjs.org/docs/app)
- [Routing Fundamentals](https://nextjs.org/docs/app/building-your-application/routing)
- [Layouts and Templates](https://nextjs.org/docs/app/building-your-application/routing/pages-and-layouts)
- [Route Groups](https://nextjs.org/docs/app/building-your-application/routing/route-groups)
