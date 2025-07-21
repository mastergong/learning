# บทที่ 9: Database Integration

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้การเชื่อมต่อ Database ใน Next.js
- ทำความเข้าใจ MongoDB และ Prisma integration
- เรียนรู้ Database connection patterns
- เข้าใจ Data modeling และ Migrations

## 🗃️ Database Options

### **📊 Database Comparison**

```javascript
// ✅ MongoDB - NoSQL Document Database
const mongoExample = {
  user: {
    _id: "64f8a1b2c3d4e5f6g7h8i9j0",
    name: "John Doe",
    email: "john@example.com",
    posts: [
      {
        title: "My First Post",
        content: "Hello world!",
        tags: ["intro", "hello"],
        createdAt: "2024-01-15T10:30:00Z"
      }
    ],
    preferences: {
      theme: "dark",
      notifications: true
    }
  }
};

// ✅ PostgreSQL/MySQL - Relational Database
const sqlExample = `
  -- Users Table
  CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
  );

  -- Posts Table
  CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    title VARCHAR(255) NOT NULL,
    content TEXT,
    created_at TIMESTAMP DEFAULT NOW()
  );
`;

// ✅ Supabase - Postgres with Real-time features
const supabaseExample = {
  features: [
    'Real-time subscriptions',
    'Built-in authentication',
    'Row Level Security',
    'RESTful API',
    'GraphQL support'
  ]
};
```

## 🍃 MongoDB Integration

### **📦 MongoDB Setup**

```bash
npm install mongodb mongoose
```

```javascript
// lib/mongodb.js
import { MongoClient } from 'mongodb';

const uri = process.env.MONGODB_URI;
const options = {
  useUnifiedTopology: true,
  useNewUrlParser: true,
};

let client;
let clientPromise;

if (process.env.NODE_ENV === 'development') {
  // ✅ In development mode, use a global variable
  // so that the value is preserved across module reloads
  if (!global._mongoClientPromise) {
    client = new MongoClient(uri, options);
    global._mongoClientPromise = client.connect();
  }
  clientPromise = global._mongoClientPromise;
} else {
  // ✅ In production mode, it's best to not use a global variable
  client = new MongoClient(uri, options);
  clientPromise = client.connect();
}

export default clientPromise;

// ✅ Database helper functions
export async function connectToDatabase() {
  try {
    const client = await clientPromise;
    const db = client.db(process.env.MONGODB_DB);
    return { client, db };
  } catch (error) {
    console.error('Database connection failed:', error);
    throw error;
  }
}

export async function disconnectFromDatabase() {
  try {
    const client = await clientPromise;
    await client.close();
  } catch (error) {
    console.error('Database disconnection failed:', error);
  }
}
```

### **🎯 MongoDB Models with Mongoose**

```javascript
// models/User.js
import mongoose from 'mongoose';
import bcrypt from 'bcryptjs';

const userSchema = new mongoose.Schema({
  name: {
    type: String,
    required: [true, 'Name is required'],
    trim: true,
    maxlength: [50, 'Name cannot exceed 50 characters']
  },
  email: {
    type: String,
    required: [true, 'Email is required'],
    unique: true,
    lowercase: true,
    match: [/^\w+([.-]?\w+)*@\w+([.-]?\w+)*(\.\w{2,3})+$/, 'Please enter a valid email']
  },
  password: {
    type: String,
    required: [true, 'Password is required'],
    minlength: [8, 'Password must be at least 8 characters'],
    select: false // Don't include password in queries by default
  },
  role: {
    type: String,
    enum: ['user', 'admin', 'moderator'],
    default: 'user'
  },
  avatar: {
    type: String,
    default: null
  },
  isActive: {
    type: Boolean,
    default: true
  },
  emailVerified: {
    type: Boolean,
    default: false
  },
  verificationToken: {
    type: String,
    select: false
  },
  verificationTokenExpires: {
    type: Date,
    select: false
  },
  resetPasswordToken: {
    type: String,
    select: false
  },
  resetPasswordExpires: {
    type: Date,
    select: false
  },
  lastLoginAt: {
    type: Date,
    default: null
  },
  preferences: {
    theme: {
      type: String,
      enum: ['light', 'dark'],
      default: 'light'
    },
    notifications: {
      email: { type: Boolean, default: true },
      push: { type: Boolean, default: true },
      sms: { type: Boolean, default: false }
    },
    language: {
      type: String,
      default: 'en'
    }
  }
}, {
  timestamps: true,
  collection: 'users'
});

// ✅ Indexes
userSchema.index({ email: 1 }, { unique: true });
userSchema.index({ createdAt: -1 });
userSchema.index({ isActive: 1, role: 1 });

// ✅ Virtual fields
userSchema.virtual('id').get(function() {
  return this._id.toHexString();
});

userSchema.virtual('posts', {
  ref: 'Post',
  localField: '_id',
  foreignField: 'author'
});

// ✅ Pre-save middleware
userSchema.pre('save', async function(next) {
  // Hash password if it's modified
  if (this.isModified('password')) {
    try {
      const salt = await bcrypt.genSalt(12);
      this.password = await bcrypt.hash(this.password, salt);
    } catch (error) {
      return next(error);
    }
  }
  next();
});

// ✅ Instance methods
userSchema.methods.comparePassword = async function(candidatePassword) {
  try {
    return await bcrypt.compare(candidatePassword, this.password);
  } catch (error) {
    throw error;
  }
};

userSchema.methods.toJSON = function() {
  const userObject = this.toObject();
  delete userObject.password;
  delete userObject.verificationToken;
  delete userObject.resetPasswordToken;
  delete userObject.__v;
  return userObject;
};

userSchema.methods.generateVerificationToken = function() {
  const crypto = require('crypto');
  const token = crypto.randomBytes(32).toString('hex');
  this.verificationToken = token;
  this.verificationTokenExpires = Date.now() + 24 * 60 * 60 * 1000; // 24 hours
  return token;
};

// ✅ Static methods
userSchema.statics.findByEmail = function(email) {
  return this.findOne({ email: email.toLowerCase(), isActive: true });
};

userSchema.statics.findActiveUsers = function(page = 1, limit = 10) {
  const skip = (page - 1) * limit;
  return this.find({ isActive: true })
    .sort({ createdAt: -1 })
    .skip(skip)
    .limit(limit);
};

export default mongoose.models.User || mongoose.model('User', userSchema);

// models/Post.js
import mongoose from 'mongoose';

const postSchema = new mongoose.Schema({
  title: {
    type: String,
    required: [true, 'Title is required'],
    trim: true,
    maxlength: [200, 'Title cannot exceed 200 characters']
  },
  slug: {
    type: String,
    required: true,
    unique: true,
    lowercase: true
  },
  content: {
    type: String,
    required: [true, 'Content is required']
  },
  excerpt: {
    type: String,
    maxlength: [500, 'Excerpt cannot exceed 500 characters']
  },
  author: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  category: {
    type: String,
    required: true,
    enum: ['technology', 'lifestyle', 'business', 'health', 'travel']
  },
  tags: [{
    type: String,
    trim: true,
    lowercase: true
  }],
  featuredImage: {
    url: String,
    alt: String,
    caption: String
  },
  status: {
    type: String,
    enum: ['draft', 'published', 'archived'],
    default: 'draft'
  },
  publishedAt: {
    type: Date,
    default: null
  },
  readTime: {
    type: Number, // in minutes
    default: 1
  },
  views: {
    type: Number,
    default: 0
  },
  likes: [{
    user: {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'User'
    },
    createdAt: {
      type: Date,
      default: Date.now
    }
  }],
  comments: [{
    user: {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'User',
      required: true
    },
    content: {
      type: String,
      required: true,
      maxlength: [1000, 'Comment cannot exceed 1000 characters']
    },
    createdAt: {
      type: Date,
      default: Date.now
    },
    isApproved: {
      type: Boolean,
      default: false
    }
  }],
  seo: {
    metaTitle: String,
    metaDescription: String,
    keywords: [String]
  }
}, {
  timestamps: true,
  collection: 'posts'
});

// ✅ Indexes
postSchema.index({ slug: 1 }, { unique: true });
postSchema.index({ author: 1, status: 1 });
postSchema.index({ category: 1, status: 1 });
postSchema.index({ tags: 1 });
postSchema.index({ publishedAt: -1 });
postSchema.index({ 'seo.keywords': 1 });

// ✅ Text search index
postSchema.index({
  title: 'text',
  content: 'text',
  excerpt: 'text',
  tags: 'text'
});

// ✅ Pre-save middleware
postSchema.pre('save', function(next) {
  // Generate slug if not provided
  if (!this.slug && this.title) {
    this.slug = this.title
      .toLowerCase()
      .replace(/[^\w\s-]/g, '')
      .replace(/[-\s]+/g, '-')
      .trim();
  }

  // Calculate read time based on content
  if (this.content) {
    const wordsPerMinute = 200;
    const wordCount = this.content.split(/\s+/).length;
    this.readTime = Math.ceil(wordCount / wordsPerMinute);
  }

  // Set publishedAt when status changes to published
  if (this.isModified('status') && this.status === 'published' && !this.publishedAt) {
    this.publishedAt = new Date();
  }

  next();
});

// ✅ Instance methods
postSchema.methods.incrementViews = function() {
  this.views += 1;
  return this.save();
};

postSchema.methods.addComment = function(userId, content) {
  this.comments.push({
    user: userId,
    content: content,
    createdAt: new Date()
  });
  return this.save();
};

postSchema.methods.toggleLike = function(userId) {
  const existingLike = this.likes.find(
    like => like.user.toString() === userId.toString()
  );

  if (existingLike) {
    this.likes = this.likes.filter(
      like => like.user.toString() !== userId.toString()
    );
  } else {
    this.likes.push({ user: userId });
  }

  return this.save();
};

// ✅ Static methods
postSchema.statics.findPublished = function(options = {}) {
  const { page = 1, limit = 10, category, tags } = options;
  const skip = (page - 1) * limit;
  
  const query = { status: 'published' };
  if (category) query.category = category;
  if (tags) query.tags = { $in: tags };

  return this.find(query)
    .populate('author', 'name avatar')
    .sort({ publishedAt: -1 })
    .skip(skip)
    .limit(limit);
};

postSchema.statics.search = function(searchTerm, options = {}) {
  const { page = 1, limit = 10 } = options;
  const skip = (page - 1) * limit;

  return this.find(
    {
      $text: { $search: searchTerm },
      status: 'published'
    },
    { score: { $meta: 'textScore' } }
  )
    .populate('author', 'name avatar')
    .sort({ score: { $meta: 'textScore' } })
    .skip(skip)
    .limit(limit);
};

export default mongoose.models.Post || mongoose.model('Post', postSchema);
```

### **🚀 MongoDB API Examples**

```javascript
// pages/api/posts/index.js
import { connectToDatabase } from '@/lib/mongodb';
import Post from '@/models/Post';
import User from '@/models/User';
import mongoose from 'mongoose';

export default async function handler(req, res) {
  await mongoose.connect(process.env.MONGODB_URI);

  switch (req.method) {
    case 'GET':
      return await getPosts(req, res);
    case 'POST':
      return await createPost(req, res);
    default:
      return res.status(405).json({ message: 'Method not allowed' });
  }
}

async function getPosts(req, res) {
  try {
    const {
      page = 1,
      limit = 10,
      category,
      tags,
      search,
      author,
      sortBy = 'publishedAt',
      sortOrder = 'desc'
    } = req.query;

    let query = { status: 'published' };
    let posts;

    // ✅ Build query filters
    if (category) {
      query.category = category;
    }

    if (tags) {
      const tagArray = Array.isArray(tags) ? tags : [tags];
      query.tags = { $in: tagArray };
    }

    if (author) {
      query.author = author;
    }

    // ✅ Handle search
    if (search) {
      posts = await Post.search(search, { page, limit });
    } else {
      const skip = (parseInt(page) - 1) * parseInt(limit);
      const sortDirection = sortOrder === 'desc' ? -1 : 1;

      posts = await Post.find(query)
        .populate('author', 'name avatar email')
        .sort({ [sortBy]: sortDirection })
        .skip(skip)
        .limit(parseInt(limit))
        .lean();
    }

    // ✅ Get total count for pagination
    const total = await Post.countDocuments(query);

    // ✅ Get categories and tags for filters
    const [categories, popularTags] = await Promise.all([
      Post.distinct('category', { status: 'published' }),
      Post.aggregate([
        { $match: { status: 'published' } },
        { $unwind: '$tags' },
        { $group: { _id: '$tags', count: { $sum: 1 } } },
        { $sort: { count: -1 } },
        { $limit: 20 }
      ])
    ]);

    res.status(200).json({
      posts,
      pagination: {
        page: parseInt(page),
        limit: parseInt(limit),
        total,
        pages: Math.ceil(total / parseInt(limit)),
        hasNext: skip + parseInt(limit) < total,
        hasPrev: parseInt(page) > 1
      },
      filters: {
        categories,
        popularTags: popularTags.map(tag => tag._id)
      }
    });
  } catch (error) {
    console.error('Get posts error:', error);
    res.status(500).json({
      message: 'Failed to fetch posts',
      error: error.message
    });
  }
}

async function createPost(req, res) {
  try {
    // ✅ Validate required fields
    const { title, content, author, category } = req.body;
    
    if (!title || !content || !author || !category) {
      return res.status(400).json({
        message: 'Missing required fields',
        required: ['title', 'content', 'author', 'category']
      });
    }

    // ✅ Verify author exists
    const authorExists = await User.findById(author);
    if (!authorExists) {
      return res.status(404).json({
        message: 'Author not found'
      });
    }

    // ✅ Create post
    const post = new Post(req.body);
    await post.save();

    // ✅ Populate author data
    await post.populate('author', 'name avatar email');

    res.status(201).json({
      message: 'Post created successfully',
      post
    });
  } catch (error) {
    console.error('Create post error:', error);
    
    if (error.code === 11000) {
      return res.status(409).json({
        message: 'Post with this slug already exists'
      });
    }

    if (error.name === 'ValidationError') {
      return res.status(400).json({
        message: 'Validation error',
        errors: Object.values(error.errors).map(err => err.message)
      });
    }

    res.status(500).json({
      message: 'Failed to create post',
      error: error.message
    });
  }
}

// pages/api/posts/[slug].js
import Post from '@/models/Post';
import mongoose from 'mongoose';

export default async function handler(req, res) {
  await mongoose.connect(process.env.MONGODB_URI);

  const { slug } = req.query;

  switch (req.method) {
    case 'GET':
      return await getPost(req, res, slug);
    case 'PUT':
      return await updatePost(req, res, slug);
    case 'DELETE':
      return await deletePost(req, res, slug);
    default:
      return res.status(405).json({ message: 'Method not allowed' });
  }
}

async function getPost(req, res, slug) {
  try {
    const post = await Post.findOne({ slug, status: 'published' })
      .populate('author', 'name avatar bio social')
      .populate('comments.user', 'name avatar')
      .lean();

    if (!post) {
      return res.status(404).json({
        message: 'Post not found'
      });
    }

    // ✅ Increment views (in background)
    Post.findByIdAndUpdate(
      post._id,
      { $inc: { views: 1 } },
      { new: false }
    ).exec().catch(console.error);

    // ✅ Get related posts
    const relatedPosts = await Post.find({
      _id: { $ne: post._id },
      $or: [
        { category: post.category },
        { tags: { $in: post.tags } }
      ],
      status: 'published'
    })
      .populate('author', 'name avatar')
      .sort({ publishedAt: -1 })
      .limit(3)
      .lean();

    res.status(200).json({
      post,
      relatedPosts
    });
  } catch (error) {
    console.error('Get post error:', error);
    res.status(500).json({
      message: 'Failed to fetch post',
      error: error.message
    });
  }
}
```

## 🔮 Prisma Integration

### **📦 Prisma Setup**

```bash
npm install prisma @prisma/client
npx prisma init
```

```javascript
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql" // or "mysql", "sqlite"
  url      = env("DATABASE_URL")
}

// ✅ User Model
model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String?
  avatar    String?
  role      Role     @default(USER)
  isActive  Boolean  @default(true)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  // Relations
  posts     Post[]
  comments  Comment[]
  likes     Like[]
  sessions  Session[]

  @@map("users")
}

// ✅ Post Model
model Post {
  id          String      @id @default(cuid())
  title       String
  slug        String      @unique
  content     String
  excerpt     String?
  status      PostStatus  @default(DRAFT)
  publishedAt DateTime?
  readTime    Int         @default(1)
  views       Int         @default(0)
  createdAt   DateTime    @default(now())
  updatedAt   DateTime    @updatedAt

  // Relations
  author      User        @relation(fields: [authorId], references: [id], onDelete: Cascade)
  authorId    String
  category    Category    @relation(fields: [categoryId], references: [id])
  categoryId  String
  tags        TagsOnPosts[]
  comments    Comment[]
  likes       Like[]

  @@map("posts")
}

// ✅ Category Model
model Category {
  id          String   @id @default(cuid())
  name        String   @unique
  slug        String   @unique
  description String?
  color       String?
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  // Relations
  posts       Post[]

  @@map("categories")
}

// ✅ Tag Model
model Tag {
  id        String   @id @default(cuid())
  name      String   @unique
  slug      String   @unique
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  // Relations
  posts     TagsOnPosts[]

  @@map("tags")
}

// ✅ Many-to-Many Junction Table
model TagsOnPosts {
  post   Post   @relation(fields: [postId], references: [id], onDelete: Cascade)
  postId String
  tag    Tag    @relation(fields: [tagId], references: [id], onDelete: Cascade)
  tagId  String

  @@id([postId, tagId])
  @@map("tags_on_posts")
}

// ✅ Comment Model
model Comment {
  id        String   @id @default(cuid())
  content   String
  isApproved Boolean @default(false)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  // Relations
  post      Post     @relation(fields: [postId], references: [id], onDelete: Cascade)
  postId    String
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  userId    String
  parent    Comment? @relation("CommentReplies", fields: [parentId], references: [id])
  parentId  String?
  replies   Comment[] @relation("CommentReplies")

  @@map("comments")
}

// ✅ Like Model
model Like {
  id        String   @id @default(cuid())
  createdAt DateTime @default(now())

  // Relations
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  userId    String
  post      Post     @relation(fields: [postId], references: [id], onDelete: Cascade)
  postId    String

  @@unique([userId, postId])
  @@map("likes")
}

// ✅ Session Model for Authentication
model Session {
  id        String   @id @default(cuid())
  token     String   @unique
  expiresAt DateTime
  createdAt DateTime @default(now())

  // Relations
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  userId    String

  @@map("sessions")
}

// ✅ Enums
enum Role {
  USER
  ADMIN
  MODERATOR
}

enum PostStatus {
  DRAFT
  PUBLISHED
  ARCHIVED
}
```

### **🎯 Prisma Client Setup**

```javascript
// lib/prisma.js
import { PrismaClient } from '@prisma/client';

const globalForPrisma = globalThis;

export const prisma = globalForPrisma.prisma || new PrismaClient({
  log: process.env.NODE_ENV === 'development' ? ['query', 'error', 'warn'] : ['error'],
});

if (process.env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = prisma;
}

// ✅ Prisma helper functions
export async function connectToPrisma() {
  try {
    await prisma.$connect();
    console.log('✅ Connected to database via Prisma');
  } catch (error) {
    console.error('❌ Failed to connect to database:', error);
    throw error;
  }
}

export async function disconnectFromPrisma() {
  try {
    await prisma.$disconnect();
    console.log('✅ Disconnected from database');
  } catch (error) {
    console.error('❌ Failed to disconnect from database:', error);
  }
}

// ✅ Database seeding helper
export async function seed() {
  try {
    // Create categories
    const categories = await Promise.all([
      prisma.category.upsert({
        where: { slug: 'technology' },
        update: {},
        create: {
          name: 'Technology',
          slug: 'technology',
          description: 'Latest tech news and tutorials',
          color: '#3B82F6'
        }
      }),
      prisma.category.upsert({
        where: { slug: 'lifestyle' },
        update: {},
        create: {
          name: 'Lifestyle',
          slug: 'lifestyle',
          description: 'Life tips and personal development',
          color: '#10B981'
        }
      })
    ]);

    // Create tags
    const tags = await Promise.all([
      prisma.tag.upsert({
        where: { slug: 'nextjs' },
        update: {},
        create: { name: 'Next.js', slug: 'nextjs' }
      }),
      prisma.tag.upsert({
        where: { slug: 'react' },
        update: {},
        create: { name: 'React', slug: 'react' }
      }),
      prisma.tag.upsert({
        where: { slug: 'javascript' },
        update: {},
        create: { name: 'JavaScript', slug: 'javascript' }
      })
    ]);

    console.log('✅ Database seeded successfully');
    return { categories, tags };
  } catch (error) {
    console.error('❌ Error seeding database:', error);
    throw error;
  }
}
```

### **🚀 Prisma API Examples**

```javascript
// pages/api/posts/index.js
import { prisma } from '@/lib/prisma';

export default async function handler(req, res) {
  switch (req.method) {
    case 'GET':
      return await getPosts(req, res);
    case 'POST':
      return await createPost(req, res);
    default:
      return res.status(405).json({ message: 'Method not allowed' });
  }
}

async function getPosts(req, res) {
  try {
    const {
      page = 1,
      limit = 10,
      category,
      tags,
      search,
      status = 'PUBLISHED'
    } = req.query;

    const skip = (parseInt(page) - 1) * parseInt(limit);
    const take = parseInt(limit);

    // ✅ Build where clause
    const where = {
      status: status as PostStatus
    };

    if (category) {
      where.category = {
        slug: category
      };
    }

    if (tags) {
      const tagArray = Array.isArray(tags) ? tags : [tags];
      where.tags = {
        some: {
          tag: {
            slug: {
              in: tagArray
            }
          }
        }
      };
    }

    if (search) {
      where.OR = [
        { title: { contains: search, mode: 'insensitive' } },
        { content: { contains: search, mode: 'insensitive' } },
        { excerpt: { contains: search, mode: 'insensitive' } }
      ];
    }

    // ✅ Execute queries in parallel
    const [posts, total, categories, popularTags] = await Promise.all([
      prisma.post.findMany({
        where,
        include: {
          author: {
            select: {
              id: true,
              name: true,
              email: true,
              avatar: true
            }
          },
          category: {
            select: {
              id: true,
              name: true,
              slug: true,
              color: true
            }
          },
          tags: {
            include: {
              tag: {
                select: {
                  id: true,
                  name: true,
                  slug: true
                }
              }
            }
          },
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
        skip,
        take
      }),
      
      prisma.post.count({ where }),
      
      prisma.category.findMany({
        select: {
          id: true,
          name: true,
          slug: true,
          _count: {
            select: {
              posts: true
            }
          }
        }
      }),
      
      prisma.tag.findMany({
        select: {
          id: true,
          name: true,
          slug: true,
          _count: {
            select: {
              posts: true
            }
          }
        },
        orderBy: {
          posts: {
            _count: 'desc'
          }
        },
        take: 20
      })
    ]);

    res.status(200).json({
      posts: posts.map(post => ({
        ...post,
        tags: post.tags.map(({ tag }) => tag)
      })),
      pagination: {
        page: parseInt(page),
        limit: take,
        total,
        pages: Math.ceil(total / take),
        hasNext: skip + take < total,
        hasPrev: parseInt(page) > 1
      },
      filters: {
        categories,
        popularTags
      }
    });
  } catch (error) {
    console.error('Get posts error:', error);
    res.status(500).json({
      message: 'Failed to fetch posts',
      error: error.message
    });
  }
}

async function createPost(req, res) {
  try {
    const { title, content, excerpt, categoryId, tagIds, authorId } = req.body;

    // ✅ Validate required fields
    if (!title || !content || !categoryId || !authorId) {
      return res.status(400).json({
        message: 'Missing required fields',
        required: ['title', 'content', 'categoryId', 'authorId']
      });
    }

    // ✅ Generate slug
    const slug = title
      .toLowerCase()
      .replace(/[^\w\s-]/g, '')
      .replace(/[-\s]+/g, '-')
      .trim();

    // ✅ Create post with relations
    const post = await prisma.post.create({
      data: {
        title,
        slug,
        content,
        excerpt,
        author: {
          connect: { id: authorId }
        },
        category: {
          connect: { id: categoryId }
        },
        ...(tagIds && tagIds.length > 0 && {
          tags: {
            create: tagIds.map((tagId: string) => ({
              tag: {
                connect: { id: tagId }
              }
            }))
          }
        })
      },
      include: {
        author: {
          select: {
            id: true,
            name: true,
            email: true,
            avatar: true
          }
        },
        category: true,
        tags: {
          include: {
            tag: true
          }
        }
      }
    });

    res.status(201).json({
      message: 'Post created successfully',
      post: {
        ...post,
        tags: post.tags.map(({ tag }) => tag)
      }
    });
  } catch (error) {
    console.error('Create post error:', error);
    
    if (error.code === 'P2002') {
      return res.status(409).json({
        message: 'Post with this slug already exists'
      });
    }

    res.status(500).json({
      message: 'Failed to create post',
      error: error.message
    });
  }
}
```

## 🔄 Database Migrations

### **📊 Prisma Migrations**

```bash
# Generate migration
npx prisma migrate dev --name add_user_preferences

# Apply migrations to production
npx prisma migrate deploy

# Generate Prisma client
npx prisma generate

# Reset database (development only)
npx prisma migrate reset

# View database in Prisma Studio
npx prisma studio
```

```javascript
// scripts/migrate.js
const { PrismaClient } = require('@prisma/client');

const prisma = new PrismaClient();

async function main() {
  try {
    console.log('🚀 Starting database migration...');
    
    // ✅ Add indexes for better performance
    await prisma.$executeRaw`
      CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_posts_published_at 
      ON posts(published_at DESC) 
      WHERE status = 'PUBLISHED';
    `;
    
    await prisma.$executeRaw`
      CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_posts_search 
      ON posts USING gin(to_tsvector('english', title || ' ' || content));
    `;
    
    // ✅ Update existing data
    const postsWithoutReadTime = await prisma.post.findMany({
      where: { readTime: 0 },
      select: { id: true, content: true }
    });
    
    for (const post of postsWithoutReadTime) {
      const wordCount = post.content.split(/\s+/).length;
      const readTime = Math.ceil(wordCount / 200); // 200 words per minute
      
      await prisma.post.update({
        where: { id: post.id },
        data: { readTime }
      });
    }
    
    console.log('✅ Migration completed successfully');
  } catch (error) {
    console.error('❌ Migration failed:', error);
    process.exit(1);
  } finally {
    await prisma.$disconnect();
  }
}

main();
```

### **🌱 Database Seeding**

```javascript
// prisma/seed.js
const { PrismaClient } = require('@prisma/client');
const bcrypt = require('bcryptjs');

const prisma = new PrismaClient();

async function main() {
  console.log('🌱 Starting database seeding...');

  // ✅ Create admin user
  const adminPassword = await bcrypt.hash('admin123!@#', 12);
  const admin = await prisma.user.upsert({
    where: { email: 'admin@example.com' },
    update: {},
    create: {
      email: 'admin@example.com',
      name: 'Admin User',
      role: 'ADMIN',
      avatar: 'https://images.unsplash.com/photo-1472099645785-5658abf4ff4e?w=150&h=150&fit=crop&crop=face'
    }
  });

  // ✅ Create categories
  const categories = await Promise.all([
    prisma.category.upsert({
      where: { slug: 'technology' },
      update: {},
      create: {
        name: 'Technology',
        slug: 'technology',
        description: 'Latest tech news and tutorials',
        color: '#3B82F6'
      }
    }),
    prisma.category.upsert({
      where: { slug: 'web-development' },
      update: {},
      create: {
        name: 'Web Development',
        slug: 'web-development',
        description: 'Frontend and backend development',
        color: '#10B981'
      }
    }),
    prisma.category.upsert({
      where: { slug: 'mobile' },
      update: {},
      create: {
        name: 'Mobile Development',
        slug: 'mobile',
        description: 'iOS and Android development',
        color: '#F59E0B'
      }
    })
  ]);

  // ✅ Create tags
  const tags = await Promise.all([
    prisma.tag.upsert({
      where: { slug: 'nextjs' },
      update: {},
      create: { name: 'Next.js', slug: 'nextjs' }
    }),
    prisma.tag.upsert({
      where: { slug: 'react' },
      update: {},
      create: { name: 'React', slug: 'react' }
    }),
    prisma.tag.upsert({
      where: { slug: 'typescript' },
      update: {},
      create: { name: 'TypeScript', slug: 'typescript' }
    }),
    prisma.tag.upsert({
      where: { slug: 'javascript' },
      update: {},
      create: { name: 'JavaScript', slug: 'javascript' }
    }),
    prisma.tag.upsert({
      where: { slug: 'css' },
      update: {},
      create: { name: 'CSS', slug: 'css' }
    })
  ]);

  // ✅ Create sample posts
  const samplePosts = [
    {
      title: 'Getting Started with Next.js 14',
      slug: 'getting-started-nextjs-14',
      content: `
        Next.js 14 introduces amazing new features that make React development even better.
        In this comprehensive guide, we'll explore the App Router, Server Components, and more.
        
        ## What's New in Next.js 14
        
        1. **Improved App Router**: Better performance and DX
        2. **Server Actions**: Simplified data mutations
        3. **Enhanced Image Optimization**: Faster loading times
        
        Let's dive deep into each of these features...
      `,
      excerpt: 'Learn about the latest features in Next.js 14 and how to get started with the App Router.',
      status: 'PUBLISHED',
      publishedAt: new Date(),
      categoryId: categories[1].id, // Web Development
      tagIds: [tags[0].id, tags[1].id, tags[2].id] // Next.js, React, TypeScript
    },
    {
      title: 'Building Responsive UIs with CSS Grid',
      slug: 'responsive-ui-css-grid',
      content: `
        CSS Grid is a powerful layout system that makes creating responsive designs easier than ever.
        In this tutorial, we'll build a complete responsive layout from scratch.
        
        ## CSS Grid Basics
        
        Grid container properties:
        - display: grid
        - grid-template-columns
        - grid-template-rows
        - gap
        
        Let's start building...
      `,
      excerpt: 'Master CSS Grid to create beautiful, responsive layouts with less code.',
      status: 'PUBLISHED',
      publishedAt: new Date(Date.now() - 86400000), // 1 day ago
      categoryId: categories[1].id, // Web Development
      tagIds: [tags[4].id] // CSS
    }
  ];

  for (const postData of samplePosts) {
    const { tagIds, ...postFields } = postData;
    
    await prisma.post.create({
      data: {
        ...postFields,
        author: {
          connect: { id: admin.id }
        },
        tags: {
          create: tagIds.map(tagId => ({
            tag: {
              connect: { id: tagId }
            }
          }))
        }
      }
    });
  }

  console.log('✅ Database seeded successfully');
}

main()
  .catch((e) => {
    console.error('❌ Seeding failed:', e);
    process.exit(1);
  })
  .finally(async () => {
    await prisma.$disconnect();
  });
```

## 📝 แบบฝึกหัด

### **🎯 แบบฝึกหัดที่ 1: MongoDB Setup**
1. ติดตั้งและเชื่อมต่อ MongoDB
2. สร้าง User และ Post models ด้วย Mongoose
3. เพิ่ม validation และ middleware
4. ทดสอบ CRUD operations

### **🎯 แบบฝึกหัดที่ 2: Prisma Integration**
1. Setup Prisma ด้วย PostgreSQL
2. สร้าง database schema
3. Run migrations และ generate client
4. สร้าง API endpoints

### **🎯 แบบฝึกหัดที่ 3: Advanced Queries**
1. เพิ่ม full-text search
2. สร้าง pagination และ filtering
3. เพิ่ม data aggregation
4. Optimize database performance

### **🎯 แบบฝึกหัดที่ 4: Database Migration**
1. สร้าง migration scripts
2. เพิ่ม database seeding
3. Setup automated backups
4. การ monitoring database performance

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 8: State Management](./08-state-management.md)

**บทต่อไป**: [บทที่ 10: Authentication](./10-authentication.md)

**กลับไปหน้าหลัก**: [README](./README.md)

---

## 📚 เอกสารอ้างอิง

- [MongoDB Documentation](https://docs.mongodb.com/)
- [Mongoose ODM](https://mongoosejs.com/)
- [Prisma Documentation](https://www.prisma.io/docs/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
