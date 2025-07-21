# บทที่ 14: Server Components

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ React Server Components
- ทำความเข้าใจ Server vs Client Components
- เรียนรู้ Data fetching ใน Server Components
- เข้าใจ Server Actions และ Forms

## ⚡ Server Components Overview

### **🖥️ Server vs Client Components**

```jsx
// ✅ Server Component (default in app directory)
// app/posts/page.js
import { PostCard } from '@/components/PostCard';
import { prisma } from '@/lib/prisma';

// ✅ This runs on the server
async function getPosts() {
  const posts = await prisma.post.findMany({
    include: {
      author: {
        select: { name: true, avatar: true }
      },
      _count: {
        select: { comments: true, likes: true }
      }
    },
    orderBy: { createdAt: 'desc' },
    take: 10
  });

  return posts;
}

// ✅ Server Component - can be async
export default async function PostsPage() {
  const posts = await getPosts();

  return (
    <div className="space-y-6">
      <h1 className="text-3xl font-bold">Latest Posts</h1>
      
      {/* ✅ Server-side rendering */}
      <div className="grid gap-6 md:grid-cols-2 lg:grid-cols-3">
        {posts.map((post) => (
          <PostCard key={post.id} post={post} />
        ))}
      </div>
    </div>
  );
}

// ✅ Client Component (interactive)
// components/InteractivePost.js
'use client';

import { useState, useEffect } from 'react';
import { useRouter } from 'next/navigation';
import { toast } from 'react-hot-toast';

export function InteractivePost({ post, initialLiked = false }) {
  const [liked, setLiked] = useState(initialLiked);
  const [likeCount, setLikeCount] = useState(post._count.likes);
  const [loading, setLoading] = useState(false);
  const router = useRouter();

  // ✅ Client-side interactivity
  const handleLike = async () => {
    if (loading) return;

    setLoading(true);
    const previousLiked = liked;
    const previousCount = likeCount;

    // ✅ Optimistic update
    setLiked(!liked);
    setLikeCount(prev => liked ? prev - 1 : prev + 1);

    try {
      const response = await fetch(`/api/posts/${post.id}/like`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ liked: !liked })
      });

      if (!response.ok) {
        throw new Error('Failed to update like');
      }

      const data = await response.json();
      setLikeCount(data.likeCount);
      
      // ✅ Revalidate the page data
      router.refresh();
    } catch (error) {
      // ✅ Revert on error
      setLiked(previousLiked);
      setLikeCount(previousCount);
      toast.error('Failed to update like');
    } finally {
      setLoading(false);
    }
  };

  const handleShare = () => {
    if (navigator.share) {
      navigator.share({
        title: post.title,
        text: post.excerpt,
        url: `/posts/${post.slug}`
      });
    } else {
      // ✅ Fallback for browsers without Web Share API
      navigator.clipboard.writeText(`${window.location.origin}/posts/${post.slug}`);
      toast.success('Link copied to clipboard!');
    }
  };

  return (
    <div className="bg-white rounded-lg shadow-md overflow-hidden">
      {/* Post content */}
      <div className="p-6">
        <h3 className="text-xl font-semibold mb-2">{post.title}</h3>
        <p className="text-gray-600 mb-4">{post.excerpt}</p>
        
        <div className="flex items-center justify-between">
          <div className="flex items-center space-x-4">
            {/* ✅ Like button */}
            <button
              onClick={handleLike}
              disabled={loading}
              className={`flex items-center space-x-1 px-3 py-1 rounded-full transition-colors ${
                liked 
                  ? 'bg-red-100 text-red-600' 
                  : 'bg-gray-100 text-gray-600 hover:bg-gray-200'
              } disabled:opacity-50`}
            >
              <svg 
                className={`w-4 h-4 ${liked ? 'fill-current' : ''}`} 
                fill="none" 
                stroke="currentColor" 
                viewBox="0 0 24 24"
              >
                <path 
                  strokeLinecap="round" 
                  strokeLinejoin="round" 
                  strokeWidth={2} 
                  d="M4.318 6.318a4.5 4.5 0 000 6.364L12 20.364l7.682-7.682a4.5 4.5 0 00-6.364-6.364L12 7.636l-1.318-1.318a4.5 4.5 0 00-6.364 0z" 
                />
              </svg>
              <span>{likeCount}</span>
            </button>

            {/* ✅ Comments */}
            <div className="flex items-center space-x-1 text-gray-600">
              <svg className="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M8 12h.01M12 12h.01M16 12h.01M21 12c0 4.418-4.03 8-9 8a9.863 9.863 0 01-4.255-.949L3 20l1.395-3.72C3.512 15.042 3 13.574 3 12c0-4.418 4.03-8 9-8s9 3.582 9 8z" />
              </svg>
              <span>{post._count.comments}</span>
            </div>
          </div>

          {/* ✅ Share button */}
          <button
            onClick={handleShare}
            className="p-2 text-gray-400 hover:text-gray-600 transition-colors"
          >
            <svg className="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M8.684 13.342C8.886 12.938 9 12.482 9 12c0-.482-.114-.938-.316-1.342m0 2.684a3 3 0 110-2.684m0 2.684l6.632 3.316m-6.632-6l6.632-3.316m0 0a3 3 0 105.367-2.684 3 3 0 00-5.367 2.684zm0 9.316a3 3 0 105.367 2.684 3 3 0 00-5.367-2.684z" />
            </svg>
          </button>
        </div>
      </div>
    </div>
  );
}

// ✅ Hybrid approach - Server Component with Client Component
// components/PostCard.js (Server Component)
import { InteractivePost } from './InteractivePost';
import { getServerSession } from 'next-auth';
import { prisma } from '@/lib/prisma';

export async function PostCard({ post }) {
  const session = await getServerSession();
  
  // ✅ Check if user has liked this post (server-side)
  let isLiked = false;
  if (session?.user) {
    const like = await prisma.like.findUnique({
      where: {
        userId_postId: {
          userId: session.user.id,
          postId: post.id
        }
      }
    });
    isLiked = !!like;
  }

  return (
    <InteractivePost 
      post={post} 
      initialLiked={isLiked} 
    />
  );
}
```

### **🔄 Data Fetching Patterns**

```jsx
// ✅ Sequential data fetching
// app/user/[id]/page.js
import { UserProfile } from '@/components/UserProfile';
import { UserPosts } from '@/components/UserPosts';
import { UserStats } from '@/components/UserStats';
import { notFound } from 'next/navigation';

async function getUser(id) {
  const user = await prisma.user.findUnique({
    where: { id },
    include: {
      profile: true,
      _count: {
        select: {
          posts: true,
          followers: true,
          following: true
        }
      }
    }
  });

  if (!user) {
    return null;
  }

  return user;
}

async function getUserPosts(id) {
  return prisma.post.findMany({
    where: {
      authorId: id,
      published: true
    },
    include: {
      categories: true,
      _count: {
        select: { comments: true, likes: true }
      }
    },
    orderBy: { publishedAt: 'desc' },
    take: 10
  });
}

async function getUserStats(id) {
  const [postsCount, likesCount, commentsCount] = await Promise.all([
    prisma.post.count({
      where: { authorId: id, published: true }
    }),
    prisma.like.count({
      where: { post: { authorId: id } }
    }),
    prisma.comment.count({
      where: { post: { authorId: id } }
    })
  ]);

  return { postsCount, likesCount, commentsCount };
}

export default async function UserPage({ params }) {
  // ✅ Sequential fetching (waterfall)
  const user = await getUser(params.id);
  
  if (!user) {
    notFound();
  }

  const posts = await getUserPosts(params.id);
  const stats = await getUserStats(params.id);

  return (
    <div className="max-w-4xl mx-auto px-4 py-8">
      <UserProfile user={user} />
      <UserStats stats={stats} />
      <UserPosts posts={posts} />
    </div>
  );
}

// ✅ Parallel data fetching
// app/dashboard/analytics/page.js
async function getAnalytics() {
  return prisma.analytics.findFirst({
    orderBy: { createdAt: 'desc' }
  });
}

async function getRevenueData() {
  return prisma.revenue.findMany({
    where: {
      createdAt: {
        gte: new Date(Date.now() - 30 * 24 * 60 * 60 * 1000) // Last 30 days
      }
    },
    orderBy: { createdAt: 'asc' }
  });
}

async function getTopProducts() {
  return prisma.product.findMany({
    include: {
      _count: {
        select: { orders: true }
      }
    },
    orderBy: {
      orders: {
        _count: 'desc'
      }
    },
    take: 5
  });
}

export default async function AnalyticsPage() {
  // ✅ Parallel fetching
  const [analytics, revenueData, topProducts] = await Promise.all([
    getAnalytics(),
    getRevenueData(),
    getTopProducts()
  ]);

  return (
    <div className="space-y-8">
      <h1 className="text-3xl font-bold">Analytics Dashboard</h1>
      
      <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
        <div className="lg:col-span-2">
          <RevenueChart data={revenueData} />
        </div>
        <div>
          <TopProducts products={topProducts} />
        </div>
      </div>
      
      <AnalyticsOverview analytics={analytics} />
    </div>
  );
}

// ✅ Streaming with Suspense
// app/products/page.js
import { Suspense } from 'react';
import { ProductGrid } from '@/components/ProductGrid';
import { ProductFilters } from '@/components/ProductFilters';
import { FeaturedProducts } from '@/components/FeaturedProducts';

// Loading components
function ProductGridSkeleton() {
  return (
    <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
      {[...Array(9)].map((_, i) => (
        <div key={i} className="bg-gray-200 h-64 rounded-lg animate-pulse" />
      ))}
    </div>
  );
}

function FeaturedProductsSkeleton() {
  return (
    <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
      {[...Array(3)].map((_, i) => (
        <div key={i} className="bg-gray-200 h-32 rounded-lg animate-pulse" />
      ))}
    </div>
  );
}

export default function ProductsPage({ searchParams }) {
  return (
    <div className="max-w-7xl mx-auto px-4 py-8">
      <h1 className="text-3xl font-bold mb-8">Products</h1>
      
      {/* ✅ Featured products with streaming */}
      <section className="mb-12">
        <h2 className="text-2xl font-semibold mb-6">Featured Products</h2>
        <Suspense fallback={<FeaturedProductsSkeleton />}>
          <FeaturedProducts />
        </Suspense>
      </section>

      <div className="flex flex-col lg:flex-row gap-8">
        {/* Filters */}
        <aside className="lg:w-64">
          <ProductFilters />
        </aside>

        {/* Product grid with streaming */}
        <main className="flex-1">
          <Suspense fallback={<ProductGridSkeleton />}>
            <ProductGrid searchParams={searchParams} />
          </Suspense>
        </main>
      </div>
    </div>
  );
}

// components/ProductGrid.js (Server Component)
async function getProducts(searchParams = {}) {
  const { category, price, sort, search, page = 1 } = searchParams;
  const limit = 12;
  const skip = (page - 1) * limit;

  const where = {
    ...(category && { categoryId: category }),
    ...(price && {
      price: {
        gte: price.min,
        lte: price.max
      }
    }),
    ...(search && {
      OR: [
        { name: { contains: search, mode: 'insensitive' } },
        { description: { contains: search, mode: 'insensitive' } }
      ]
    })
  };

  const orderBy = (() => {
    switch (sort) {
      case 'price-low':
        return { price: 'asc' };
      case 'price-high':
        return { price: 'desc' };
      case 'name':
        return { name: 'asc' };
      default:
        return { createdAt: 'desc' };
    }
  })();

  const [products, total] = await Promise.all([
    prisma.product.findMany({
      where,
      include: {
        category: true,
        images: true,
        _count: {
          select: { reviews: true }
        }
      },
      orderBy,
      skip,
      take: limit
    }),
    prisma.product.count({ where })
  ]);

  return { products, total, hasMore: skip + limit < total };
}

export async function ProductGrid({ searchParams }) {
  const { products, total, hasMore } = await getProducts(searchParams);

  if (products.length === 0) {
    return (
      <div className="text-center py-12">
        <h3 className="text-lg font-medium text-gray-900">No products found</h3>
        <p className="mt-2 text-gray-600">
          Try adjusting your search or filter criteria.
        </p>
      </div>
    );
  }

  return (
    <div className="space-y-6">
      <div className="flex justify-between items-center">
        <p className="text-gray-600">{total} products found</p>
      </div>

      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
        {products.map((product) => (
          <ProductCard key={product.id} product={product} />
        ))}
      </div>

      {hasMore && (
        <div className="text-center">
          <LoadMoreButton searchParams={searchParams} />
        </div>
      )}
    </div>
  );
}
```

## 🎬 Server Actions

### **📝 Server Actions in Forms**

```jsx
// ✅ Server Action for form submission
// app/contact/actions.js
'use server';

import { prisma } from '@/lib/prisma';
import { sendEmail } from '@/lib/email';
import { revalidatePath } from 'next/cache';
import { redirect } from 'next/navigation';
import * as yup from 'yup';

const contactSchema = yup.object({
  name: yup.string().required('Name is required').min(2),
  email: yup.string().required('Email is required').email(),
  subject: yup.string().required('Subject is required').min(5),
  message: yup.string().required('Message is required').min(10),
  category: yup.string().oneOf(['general', 'support', 'sales'])
});

export async function submitContactForm(prevState, formData) {
  try {
    // ✅ Extract form data
    const data = {
      name: formData.get('name'),
      email: formData.get('email'),
      subject: formData.get('subject'),
      message: formData.get('message'),
      category: formData.get('category')
    };

    // ✅ Validate data
    await contactSchema.validate(data, { abortEarly: false });

    // ✅ Rate limiting check
    const recentSubmissions = await prisma.contactSubmission.count({
      where: {
        email: data.email,
        createdAt: {
          gte: new Date(Date.now() - 60 * 60 * 1000) // Last hour
        }
      }
    });

    if (recentSubmissions >= 3) {
      return {
        success: false,
        message: 'Too many submissions. Please try again later.',
        errors: {}
      };
    }

    // ✅ Save submission
    const submission = await prisma.contactSubmission.create({
      data: {
        ...data,
        status: 'pending'
      }
    });

    // ✅ Send email notification
    await sendEmail({
      to: process.env.ADMIN_EMAIL,
      subject: `New contact form submission: ${data.subject}`,
      template: 'contact-notification',
      data: {
        ...data,
        submissionId: submission.id
      }
    });

    // ✅ Send confirmation email to user
    await sendEmail({
      to: data.email,
      subject: 'Thank you for contacting us',
      template: 'contact-confirmation',
      data: {
        name: data.name,
        submissionId: submission.id
      }
    });

    // ✅ Revalidate cache
    revalidatePath('/contact');

    return {
      success: true,
      message: 'Thank you for your message. We\'ll get back to you soon!',
      errors: {}
    };

  } catch (error) {
    if (error.name === 'ValidationError') {
      const errors = {};
      error.inner.forEach(err => {
        errors[err.path] = err.message;
      });

      return {
        success: false,
        message: 'Please fix the errors below.',
        errors
      };
    }

    console.error('Contact form error:', error);
    return {
      success: false,
      message: 'Something went wrong. Please try again.',
      errors: {}
    };
  }
}

// ✅ Server Action for post creation
export async function createPost(prevState, formData) {
  try {
    const session = await getServerSession(authOptions);
    
    if (!session) {
      return {
        success: false,
        message: 'You must be logged in to create a post.',
        errors: {}
      };
    }

    const data = {
      title: formData.get('title'),
      content: formData.get('content'),
      excerpt: formData.get('excerpt'),
      categoryId: formData.get('categoryId'),
      tags: formData.get('tags')?.split(',').map(tag => tag.trim()) || [],
      published: formData.get('published') === 'true'
    };

    // ✅ Validation
    if (!data.title || !data.content) {
      return {
        success: false,
        message: 'Title and content are required.',
        errors: {
          ...((!data.title) && { title: 'Title is required' }),
          ...((!data.content) && { content: 'Content is required' })
        }
      };
    }

    // ✅ Generate slug
    const slug = data.title
      .toLowerCase()
      .replace(/[^a-z0-9]+/g, '-')
      .replace(/(^-|-$)/g, '');

    // ✅ Check if slug exists
    const existingPost = await prisma.post.findUnique({
      where: { slug }
    });

    if (existingPost) {
      return {
        success: false,
        message: 'A post with this title already exists.',
        errors: { title: 'This title is already taken' }
      };
    }

    // ✅ Create post
    const post = await prisma.post.create({
      data: {
        title: data.title,
        slug,
        content: data.content,
        excerpt: data.excerpt || data.content.substring(0, 150) + '...',
        published: data.published,
        publishedAt: data.published ? new Date() : null,
        authorId: session.user.id,
        ...(data.categoryId && { categoryId: data.categoryId }),
        tags: {
          connectOrCreate: data.tags.map(tag => ({
            where: { name: tag },
            create: { 
              name: tag, 
              slug: tag.toLowerCase().replace(/\s+/g, '-') 
            }
          }))
        }
      }
    });

    // ✅ Revalidate related pages
    revalidatePath('/blog');
    revalidatePath('/dashboard/posts');
    
    if (data.published) {
      revalidatePath(`/blog/${slug}`);
    }

    // ✅ Redirect to post
    redirect(`/dashboard/posts/${post.id}`);

  } catch (error) {
    console.error('Create post error:', error);
    return {
      success: false,
      message: 'Failed to create post. Please try again.',
      errors: {}
    };
  }
}

// ✅ Server Action for like/unlike
export async function togglePostLike(postId) {
  try {
    const session = await getServerSession(authOptions);
    
    if (!session) {
      return { success: false, message: 'Login required' };
    }

    const existingLike = await prisma.like.findUnique({
      where: {
        userId_postId: {
          userId: session.user.id,
          postId
        }
      }
    });

    if (existingLike) {
      // ✅ Unlike
      await prisma.like.delete({
        where: { id: existingLike.id }
      });
    } else {
      // ✅ Like
      await prisma.like.create({
        data: {
          userId: session.user.id,
          postId
        }
      });
    }

    // ✅ Get updated like count
    const likeCount = await prisma.like.count({
      where: { postId }
    });

    // ✅ Revalidate post pages
    revalidatePath(`/blog`);
    revalidatePath(`/blog/[slug]`, 'page');

    return { 
      success: true, 
      liked: !existingLike,
      likeCount 
    };

  } catch (error) {
    console.error('Toggle like error:', error);
    return { success: false, message: 'Failed to update like' };
  }
}

// app/contact/page.js
import { ContactForm } from './ContactForm';

export const metadata = {
  title: 'Contact Us',
  description: 'Get in touch with our team'
};

export default function ContactPage() {
  return (
    <div className="max-w-2xl mx-auto px-4 py-16">
      <div className="text-center mb-8">
        <h1 className="text-4xl font-bold text-gray-900 mb-4">Contact Us</h1>
        <p className="text-xl text-gray-600">
          We'd love to hear from you. Send us a message and we'll respond as soon as possible.
        </p>
      </div>
      
      <ContactForm />
    </div>
  );
}

// app/contact/ContactForm.js
'use client';

import { useFormState, useFormStatus } from 'react-dom';
import { submitContactForm } from './actions';
import { useEffect } from 'react';
import { toast } from 'react-hot-toast';

function SubmitButton() {
  const { pending } = useFormStatus();

  return (
    <button
      type="submit"
      disabled={pending}
      className="w-full bg-blue-600 text-white py-3 px-4 rounded-lg hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 disabled:opacity-50 disabled:cursor-not-allowed"
    >
      {pending ? 'Sending...' : 'Send Message'}
    </button>
  );
}

export function ContactForm() {
  const [state, formAction] = useFormState(submitContactForm, {
    success: null,
    message: '',
    errors: {}
  });

  useEffect(() => {
    if (state.success === true) {
      toast.success(state.message);
    } else if (state.success === false && state.message) {
      toast.error(state.message);
    }
  }, [state]);

  return (
    <div className="bg-white shadow-lg rounded-lg p-8">
      <form action={formAction} className="space-y-6">
        {/* Name */}
        <div>
          <label htmlFor="name" className="block text-sm font-medium text-gray-700">
            Name *
          </label>
          <input
            type="text"
            id="name"
            name="name"
            required
            className={`mt-1 block w-full px-3 py-2 border rounded-md shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-500 ${
              state.errors.name 
                ? 'border-red-300 focus:border-red-500' 
                : 'border-gray-300 focus:border-blue-500'
            }`}
            placeholder="Your full name"
          />
          {state.errors.name && (
            <p className="mt-1 text-sm text-red-600">{state.errors.name}</p>
          )}
        </div>

        {/* Email */}
        <div>
          <label htmlFor="email" className="block text-sm font-medium text-gray-700">
            Email *
          </label>
          <input
            type="email"
            id="email"
            name="email"
            required
            className={`mt-1 block w-full px-3 py-2 border rounded-md shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-500 ${
              state.errors.email 
                ? 'border-red-300 focus:border-red-500' 
                : 'border-gray-300 focus:border-blue-500'
            }`}
            placeholder="your@email.com"
          />
          {state.errors.email && (
            <p className="mt-1 text-sm text-red-600">{state.errors.email}</p>
          )}
        </div>

        {/* Category */}
        <div>
          <label htmlFor="category" className="block text-sm font-medium text-gray-700">
            Category
          </label>
          <select
            id="category"
            name="category"
            className="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-500"
          >
            <option value="general">General Inquiry</option>
            <option value="support">Support</option>
            <option value="sales">Sales</option>
          </select>
        </div>

        {/* Subject */}
        <div>
          <label htmlFor="subject" className="block text-sm font-medium text-gray-700">
            Subject *
          </label>
          <input
            type="text"
            id="subject"
            name="subject"
            required
            className={`mt-1 block w-full px-3 py-2 border rounded-md shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-500 ${
              state.errors.subject 
                ? 'border-red-300 focus:border-red-500' 
                : 'border-gray-300 focus:border-blue-500'
            }`}
            placeholder="What is this about?"
          />
          {state.errors.subject && (
            <p className="mt-1 text-sm text-red-600">{state.errors.subject}</p>
          )}
        </div>

        {/* Message */}
        <div>
          <label htmlFor="message" className="block text-sm font-medium text-gray-700">
            Message *
          </label>
          <textarea
            id="message"
            name="message"
            rows={6}
            required
            className={`mt-1 block w-full px-3 py-2 border rounded-md shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-500 ${
              state.errors.message 
                ? 'border-red-300 focus:border-red-500' 
                : 'border-gray-300 focus:border-blue-500'
            }`}
            placeholder="Tell us more about your inquiry..."
          />
          {state.errors.message && (
            <p className="mt-1 text-sm text-red-600">{state.errors.message}</p>
          )}
        </div>

        {/* Submit button */}
        <SubmitButton />
      </form>
    </div>
  );
}
```

### **🔄 Optimistic Updates**

```jsx
// components/OptimisticLike.js
'use client';

import { useOptimistic, useTransition } from 'react';
import { togglePostLike } from '@/app/actions';

export function OptimisticLike({ postId, initialLiked, initialCount }) {
  const [isPending, startTransition] = useTransition();
  const [optimisticState, addOptimistic] = useOptimistic(
    { liked: initialLiked, count: initialCount },
    (state, newLiked) => ({
      liked: newLiked,
      count: state.count + (newLiked ? 1 : -1)
    })
  );

  const handleLike = () => {
    startTransition(async () => {
      // ✅ Optimistic update
      addOptimistic(!optimisticState.liked);
      
      // ✅ Server action
      await togglePostLike(postId);
    });
  };

  return (
    <button
      onClick={handleLike}
      disabled={isPending}
      className={`flex items-center space-x-2 px-4 py-2 rounded-full transition-colors ${
        optimisticState.liked
          ? 'bg-red-100 text-red-600'
          : 'bg-gray-100 text-gray-600 hover:bg-gray-200'
      } disabled:opacity-50`}
    >
      <svg 
        className={`w-5 h-5 ${optimisticState.liked ? 'fill-current' : ''}`}
        fill="none" 
        stroke="currentColor" 
        viewBox="0 0 24 24"
      >
        <path 
          strokeLinecap="round" 
          strokeLinejoin="round" 
          strokeWidth={2} 
          d="M4.318 6.318a4.5 4.5 0 000 6.364L12 20.364l7.682-7.682a4.5 4.5 0 00-6.364-6.364L12 7.636l-1.318-1.318a4.5 4.5 0 00-6.364 0z" 
        />
      </svg>
      <span>{optimisticState.count}</span>
    </button>
  );
}

// components/OptimisticComments.js
'use client';

import { useOptimistic, useTransition } from 'react';
import { addComment } from '@/app/actions';
import { useSession } from 'next-auth/react';

export function OptimisticComments({ postId, initialComments }) {
  const { data: session } = useSession();
  const [isPending, startTransition] = useTransition();
  const [optimisticComments, addOptimisticComment] = useOptimistic(
    initialComments,
    (state, newComment) => [...state, newComment]
  );

  const handleSubmit = async (formData) => {
    if (!session) return;

    const content = formData.get('content');
    if (!content?.trim()) return;

    const optimisticComment = {
      id: Date.now(), // Temporary ID
      content,
      createdAt: new Date().toISOString(),
      author: {
        name: session.user.name,
        avatar: session.user.image
      },
      isOptimistic: true
    };

    startTransition(async () => {
      // ✅ Add optimistic comment
      addOptimisticComment(optimisticComment);
      
      // ✅ Submit to server
      await addComment(postId, content);
    });
  };

  return (
    <div className="space-y-6">
      <h3 className="text-xl font-semibold">
        Comments ({optimisticComments.length})
      </h3>

      {/* Comment form */}
      {session && (
        <form action={handleSubmit} className="space-y-4">
          <textarea
            name="content"
            placeholder="Write a comment..."
            className="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
            rows={3}
            required
          />
          <button
            type="submit"
            disabled={isPending}
            className="bg-blue-600 text-white px-4 py-2 rounded-md hover:bg-blue-700 disabled:opacity-50"
          >
            {isPending ? 'Posting...' : 'Post Comment'}
          </button>
        </form>
      )}

      {/* Comments list */}
      <div className="space-y-4">
        {optimisticComments.map((comment) => (
          <div 
            key={comment.id} 
            className={`p-4 bg-gray-50 rounded-lg ${
              comment.isOptimistic ? 'opacity-50' : ''
            }`}
          >
            <div className="flex items-center space-x-3 mb-2">
              <img
                src={comment.author.avatar}
                alt={comment.author.name}
                className="w-8 h-8 rounded-full"
              />
              <div>
                <p className="font-medium text-gray-900">
                  {comment.author.name}
                </p>
                <p className="text-sm text-gray-500">
                  {new Date(comment.createdAt).toLocaleDateString()}
                </p>
              </div>
            </div>
            <p className="text-gray-700">{comment.content}</p>
          </div>
        ))}
      </div>
    </div>
  );
}
```

## 📝 แบบฝึกหัด

### **🎯 แบบฝึกหัดที่ 1: Server Components Setup**
1. สร้าง Server Components สำหรับ data fetching
2. เพิ่ม Client Components สำหรับ interactivity
3. ทดสอบ hybrid approach
4. วัด performance differences

### **🎯 แบบฝึกหัดที่ 2: Data Fetching Patterns**
1. เปรียบเทียบ sequential vs parallel fetching
2. เพิ่ม Suspense boundaries
3. สร้าง loading skeletons
4. ทดสอบ streaming

### **🎯 แบบฝึกหัดที่ 3: Server Actions**
1. สร้าง Server Actions สำหรับ forms
2. เพิ่ม validation และ error handling
3. ทดสอบ progressive enhancement
4. เพิ่ม revalidation

### **🎯 แบบฝึกหัดที่ 4: Optimistic Updates**
1. เพิ่ม useOptimistic hooks
2. สร้าง optimistic UI updates
3. จัดการ error states
4. ทดสอบ user experience

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 13: App Router](./13-app-router.md)

**บทต่อไป**: [บทที่ 15: Streaming and Suspense](./15-streaming-suspense.md)

**กลับไปหน้าหลัก**: [README](./README.md)

---

## 📚 เอกสารอ้างอิง

- [React Server Components](https://nextjs.org/docs/app/building-your-application/rendering/server-components)
- [Server Actions](https://nextjs.org/docs/app/building-your-application/data-fetching/forms-and-mutations)
- [Client Components](https://nextjs.org/docs/app/building-your-application/rendering/client-components)
- [Data Fetching](https://nextjs.org/docs/app/building-your-application/data-fetching)
