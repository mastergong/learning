# บทที่ 23: Real-world Projects

## 🎯 จุดประสงค์ของบทเรียน
- สร้าง real-world applications แบบสมบูรณ์
- ใช้ความรู้ทั้งหมดที่เรียนมาในโปรเจกต์จริง
- เรียนรู้ best practices ในการพัฒนา production-ready apps
- ทำความเข้าใจ project structure และ workflow ที่เหมาะสม

## 🛒 โปรเจกต์ที่ 1: E-commerce Platform

### **🏗️ Project Architecture**

```
ecommerce-platform/
├── prisma/
│   ├── schema.prisma
│   ├── migrations/
│   └── seed.ts
├── src/
│   ├── app/                    # App Router
│   │   ├── (auth)/
│   │   │   ├── login/
│   │   │   └── register/
│   │   ├── (dashboard)/
│   │   │   ├── admin/
│   │   │   └── seller/
│   │   ├── products/
│   │   ├── cart/
│   │   ├── checkout/
│   │   └── api/
│   ├── components/
│   │   ├── ui/
│   │   ├── forms/
│   │   ├── layout/
│   │   └── product/
│   ├── lib/
│   │   ├── auth/
│   │   ├── database/
│   │   ├── payment/
│   │   ├── email/
│   │   └── utils/
│   └── store/
├── public/
├── tests/
└── docs/
```

### **💾 Database Schema**

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id          String   @id @default(cuid())
  email       String   @unique
  name        String
  password    String
  role        UserRole @default(CUSTOMER)
  avatar      String?
  phone       String?
  isVerified  Boolean  @default(false)
  isActive    Boolean  @default(true)
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  // Relations
  addresses     Address[]
  orders        Order[]
  reviews       Review[]
  wishlist      WishlistItem[]
  cart          CartItem[]
  sellerProfile SellerProfile?

  @@map("users")
}

model SellerProfile {
  id          String  @id @default(cuid())
  userId      String  @unique
  user        User    @relation(fields: [userId], references: [id], onDelete: Cascade)
  storeName   String
  storeSlug   String  @unique
  description String?
  logo        String?
  banner      String?
  isVerified  Boolean @default(false)
  rating      Float   @default(0)
  totalSales  Int     @default(0)

  // Relations
  products Product[]

  @@map("seller_profiles")
}

model Category {
  id          String @id @default(cuid())
  name        String
  slug        String @unique
  description String?
  image       String?
  parentId    String?
  parent      Category? @relation("CategoryHierarchy", fields: [parentId], references: [id])
  children    Category[] @relation("CategoryHierarchy")
  isActive    Boolean @default(true)
  
  // Relations
  products Product[]

  @@map("categories")
}

model Product {
  id          String  @id @default(cuid())
  name        String
  slug        String  @unique
  description String
  shortDesc   String?
  sku         String  @unique
  price       Float
  comparePrice Float?
  cost        Float?
  images      String[]
  thumbnail   String
  status      ProductStatus @default(DRAFT)
  inventory   Int     @default(0)
  lowStock    Int     @default(10)
  weight      Float?
  dimensions  Json?
  seoTitle    String?
  seoDesc     String?
  tags        String[]
  
  // Relations
  categoryId String
  category   Category @relation(fields: [categoryId], references: [id])
  sellerId   String
  seller     SellerProfile @relation(fields: [sellerId], references: [id])
  
  variants      ProductVariant[]
  reviews       Review[]
  wishlistItems WishlistItem[]
  cartItems     CartItem[]
  orderItems    OrderItem[]
  
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@map("products")
}

model ProductVariant {
  id        String @id @default(cuid())
  productId String
  product   Product @relation(fields: [productId], references: [id], onDelete: Cascade)
  name      String
  sku       String @unique
  price     Float
  inventory Int
  options   Json // e.g., { "color": "red", "size": "M" }
  image     String?
  
  // Relations
  cartItems  CartItem[]
  orderItems OrderItem[]

  @@map("product_variants")
}

model Address {
  id         String  @id @default(cuid())
  userId     String
  user       User    @relation(fields: [userId], references: [id], onDelete: Cascade)
  type       AddressType @default(SHIPPING)
  firstName  String
  lastName   String
  company    String?
  address1   String
  address2   String?
  city       String
  state      String
  zipCode    String
  country    String
  phone      String?
  isDefault  Boolean @default(false)
  
  // Relations
  orders Order[]

  @@map("addresses")
}

model Cart {
  id     String     @id @default(cuid())
  userId String?
  user   User?      @relation(fields: [userId], references: [id], onDelete: Cascade)
  items  CartItem[]
  
  sessionId String? // For guest carts
  expiresAt DateTime?
  
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@map("carts")
}

model CartItem {
  id        String @id @default(cuid())
  cartId    String
  cart      Cart   @relation(fields: [cartId], references: [id], onDelete: Cascade)
  productId String
  product   Product @relation(fields: [productId], references: [id], onDelete: Cascade)
  variantId String?
  variant   ProductVariant? @relation(fields: [variantId], references: [id], onDelete: Cascade)
  quantity  Int
  price     Float // Price at time of adding to cart
  
  userId String?
  user   User?   @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@unique([cartId, productId, variantId])
  @@map("cart_items")
}

model Order {
  id            String      @id @default(cuid())
  orderNumber   String      @unique
  userId        String
  user          User        @relation(fields: [userId], references: [id])
  status        OrderStatus @default(PENDING)
  
  // Pricing
  subtotal      Float
  taxAmount     Float
  shippingCost  Float
  discountAmount Float @default(0)
  total         Float
  
  // Shipping
  shippingAddressId String
  shippingAddress   Address @relation(fields: [shippingAddressId], references: [id])
  shippingMethod    String
  trackingNumber    String?
  
  // Payment
  paymentStatus   PaymentStatus @default(PENDING)
  paymentMethod   String
  paymentId       String? // External payment ID
  
  // Fulfillment
  fulfillmentStatus FulfillmentStatus @default(PENDING)
  shippedAt         DateTime?
  deliveredAt       DateTime?
  
  // Relations
  items   OrderItem[]
  refunds Refund[]
  
  notes     String?
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@map("orders")
}

model OrderItem {
  id        String @id @default(cuid())
  orderId   String
  order     Order  @relation(fields: [orderId], references: [id], onDelete: Cascade)
  productId String
  product   Product @relation(fields: [productId], references: [id])
  variantId String?
  variant   ProductVariant? @relation(fields: [variantId], references: [id])
  
  quantity    Int
  price       Float // Price at time of order
  total       Float
  productName String // Snapshot of product name
  productSku  String // Snapshot of SKU
  
  @@map("order_items")
}

model Review {
  id        String @id @default(cuid())
  productId String
  product   Product @relation(fields: [productId], references: [id], onDelete: Cascade)
  userId    String
  user      User    @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  rating    Int     // 1-5 stars
  title     String?
  comment   String
  images    String[]
  isVerified Boolean @default(false) // Verified purchase
  
  helpfulCount Int @default(0)
  
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@unique([productId, userId])
  @@map("reviews")
}

model WishlistItem {
  id        String @id @default(cuid())
  userId    String
  user      User   @relation(fields: [userId], references: [id], onDelete: Cascade)
  productId String
  product   Product @relation(fields: [productId], references: [id], onDelete: Cascade)
  
  addedAt DateTime @default(now())

  @@unique([userId, productId])
  @@map("wishlist_items")
}

model Refund {
  id        String      @id @default(cuid())
  orderId   String
  order     Order       @relation(fields: [orderId], references: [id])
  amount    Float
  reason    String
  status    RefundStatus @default(PENDING)
  refundId  String? // External refund ID
  
  processedAt DateTime?
  createdAt   DateTime @default(now())

  @@map("refunds")
}

// Enums
enum UserRole {
  CUSTOMER
  SELLER
  ADMIN
  SUPER_ADMIN
}

enum ProductStatus {
  DRAFT
  ACTIVE
  INACTIVE
  ARCHIVED
}

enum AddressType {
  SHIPPING
  BILLING
}

enum OrderStatus {
  PENDING
  CONFIRMED
  PROCESSING
  SHIPPED
  DELIVERED
  CANCELLED
  REFUNDED
}

enum PaymentStatus {
  PENDING
  AUTHORIZED
  PAID
  FAILED
  CANCELLED
  REFUNDED
  PARTIALLY_REFUNDED
}

enum FulfillmentStatus {
  PENDING
  PROCESSING
  SHIPPED
  DELIVERED
  RETURNED
}

enum RefundStatus {
  PENDING
  APPROVED
  PROCESSED
  REJECTED
}
```

### **🛍️ Product Components**

```tsx
// src/components/product/ProductCard.tsx
import Image from 'next/image';
import Link from 'next/link';
import { Star, Heart, ShoppingCart } from 'lucide-react';
import { Button } from '@/components/ui/Button';
import { Badge } from '@/components/ui/Badge';
import { useCart } from '@/store/useCart';
import { useWishlist } from '@/store/useWishlist';
import { Product } from '@prisma/client';

interface ProductCardProps {
  product: Product & {
    seller: { storeName: string };
    reviews: { rating: number }[];
  };
  variant?: 'grid' | 'list';
}

export function ProductCard({ product, variant = 'grid' }: ProductCardProps) {
  const { addItem } = useCart();
  const { toggleItem, isInWishlist } = useWishlist();

  const averageRating = product.reviews.length > 0
    ? product.reviews.reduce((sum, review) => sum + review.rating, 0) / product.reviews.length
    : 0;

  const discount = product.comparePrice
    ? Math.round(((product.comparePrice - product.price) / product.comparePrice) * 100)
    : 0;

  const handleAddToCart = (e: React.MouseEvent) => {
    e.preventDefault();
    addItem(product, 1);
  };

  const handleWishlistToggle = (e: React.MouseEvent) => {
    e.preventDefault();
    toggleItem(product.id);
  };

  if (variant === 'list') {
    return (
      <div className="flex bg-white border rounded-lg hover:shadow-lg transition-shadow">
        <div className="relative w-48 h-48 flex-shrink-0">
          <Image
            src={product.thumbnail}
            alt={product.name}
            fill
            className="object-cover rounded-l-lg"
          />
          {discount > 0 && (
            <Badge className="absolute top-2 left-2 bg-red-500">
              -{discount}%
            </Badge>
          )}
        </div>
        
        <div className="flex-1 p-6 flex flex-col justify-between">
          <div>
            <div className="flex items-start justify-between mb-2">
              <Link href={`/products/${product.slug}`}>
                <h3 className="text-lg font-semibold text-gray-900 hover:text-blue-600 line-clamp-2">
                  {product.name}
                </h3>
              </Link>
              <Button
                variant="ghost"
                size="sm"
                onClick={handleWishlistToggle}
                className="text-gray-400 hover:text-red-500"
              >
                <Heart 
                  className={`w-5 h-5 ${isInWishlist(product.id) ? 'fill-red-500 text-red-500' : ''}`} 
                />
              </Button>
            </div>

            <p className="text-sm text-gray-600 mb-2">{product.seller.storeName}</p>
            
            <p className="text-gray-700 text-sm mb-4 line-clamp-3">
              {product.shortDesc || product.description}
            </p>

            <div className="flex items-center gap-2 mb-4">
              <div className="flex items-center">
                {[...Array(5)].map((_, i) => (
                  <Star
                    key={i}
                    className={`w-4 h-4 ${
                      i < Math.floor(averageRating)
                        ? 'text-yellow-400 fill-yellow-400'
                        : 'text-gray-300'
                    }`}
                  />
                ))}
              </div>
              <span className="text-sm text-gray-600">
                ({product.reviews.length})
              </span>
            </div>
          </div>

          <div className="flex items-center justify-between">
            <div className="flex items-center gap-2">
              <span className="text-2xl font-bold text-gray-900">
                ฿{product.price.toLocaleString()}
              </span>
              {product.comparePrice && (
                <span className="text-lg text-gray-500 line-through">
                  ฿{product.comparePrice.toLocaleString()}
                </span>
              )}
            </div>

            <Button onClick={handleAddToCart} className="flex items-center gap-2">
              <ShoppingCart className="w-4 h-4" />
              เพิ่มลงตะกร้า
            </Button>
          </div>
        </div>
      </div>
    );
  }

  return (
    <div className="group bg-white border rounded-lg hover:shadow-lg transition-all duration-300">
      <div className="relative aspect-square overflow-hidden rounded-t-lg">
        <Image
          src={product.thumbnail}
          alt={product.name}
          fill
          className="object-cover group-hover:scale-105 transition-transform duration-300"
        />
        
        {discount > 0 && (
          <Badge className="absolute top-2 left-2 bg-red-500">
            -{discount}%
          </Badge>
        )}

        <div className="absolute top-2 right-2 opacity-0 group-hover:opacity-100 transition-opacity">
          <Button
            variant="ghost"
            size="sm"
            onClick={handleWishlistToggle}
            className="bg-white/80 backdrop-blur-sm text-gray-700 hover:text-red-500"
          >
            <Heart 
              className={`w-4 h-4 ${isInWishlist(product.id) ? 'fill-red-500 text-red-500' : ''}`} 
            />
          </Button>
        </div>

        {product.inventory <= product.lowStock && product.inventory > 0 && (
          <Badge variant="outline" className="absolute bottom-2 left-2 bg-orange-500 text-white">
            เหลือน้อย
          </Badge>
        )}

        {product.inventory === 0 && (
          <div className="absolute inset-0 bg-black/50 flex items-center justify-center">
            <Badge variant="destructive">สินค้าหมด</Badge>
          </div>
        )}
      </div>

      <div className="p-4">
        <p className="text-xs text-gray-500 mb-1">{product.seller.storeName}</p>
        
        <Link href={`/products/${product.slug}`}>
          <h3 className="font-semibold text-gray-900 hover:text-blue-600 line-clamp-2 mb-2">
            {product.name}
          </h3>
        </Link>

        <div className="flex items-center gap-2 mb-3">
          <div className="flex items-center">
            {[...Array(5)].map((_, i) => (
              <Star
                key={i}
                className={`w-3 h-3 ${
                  i < Math.floor(averageRating)
                    ? 'text-yellow-400 fill-yellow-400'
                    : 'text-gray-300'
                }`}
              />
            ))}
          </div>
          <span className="text-xs text-gray-600">
            ({product.reviews.length})
          </span>
        </div>

        <div className="flex items-center justify-between mb-3">
          <div className="flex items-center gap-1">
            <span className="text-lg font-bold text-gray-900">
              ฿{product.price.toLocaleString()}
            </span>
            {product.comparePrice && (
              <span className="text-sm text-gray-500 line-through">
                ฿{product.comparePrice.toLocaleString()}
              </span>
            )}
          </div>
        </div>

        <Button 
          onClick={handleAddToCart} 
          className="w-full"
          disabled={product.inventory === 0}
        >
          <ShoppingCart className="w-4 h-4 mr-2" />
          {product.inventory === 0 ? 'สินค้าหมด' : 'เพิ่มลงตะกร้า'}
        </Button>
      </div>
    </div>
  );
}

// src/components/product/ProductDetails.tsx
import { useState } from 'react';
import Image from 'next/image';
import { Star, Heart, Share2, Truck, Shield, RotateCcw } from 'lucide-react';
import { Button } from '@/components/ui/Button';
import { Badge } from '@/components/ui/Badge';
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/Tabs';
import { ProductVariantSelector } from './ProductVariantSelector';
import { ProductReviews } from './ProductReviews';
import { QuantitySelector } from './QuantitySelector';

interface ProductDetailsProps {
  product: Product & {
    category: Category;
    seller: SellerProfile;
    variants: ProductVariant[];
    reviews: (Review & { user: { name: string } })[];
  };
}

export function ProductDetails({ product }: ProductDetailsProps) {
  const [selectedImage, setSelectedImage] = useState(0);
  const [selectedVariant, setSelectedVariant] = useState<ProductVariant | null>(null);
  const [quantity, setQuantity] = useState(1);

  const averageRating = product.reviews.length > 0
    ? product.reviews.reduce((sum, review) => sum + review.rating, 0) / product.reviews.length
    : 0;

  const currentPrice = selectedVariant?.price || product.price;
  const currentInventory = selectedVariant?.inventory || product.inventory;

  const handleAddToCart = () => {
    // Add to cart logic
    console.log('Adding to cart:', {
      productId: product.id,
      variantId: selectedVariant?.id,
      quantity
    });
  };

  return (
    <div className="max-w-7xl mx-auto px-4 py-8">
      {/* Breadcrumb */}
      <nav className="flex items-center space-x-2 text-sm text-gray-600 mb-6">
        <a href="/" className="hover:text-gray-900">หน้าแรก</a>
        <span>/</span>
        <a href={`/categories/${product.category.slug}`} className="hover:text-gray-900">
          {product.category.name}
        </a>
        <span>/</span>
        <span className="text-gray-900">{product.name}</span>
      </nav>

      <div className="grid grid-cols-1 lg:grid-cols-2 gap-8 mb-12">
        {/* Product Images */}
        <div className="space-y-4">
          <div className="aspect-square relative overflow-hidden rounded-lg border">
            <Image
              src={product.images[selectedImage] || product.thumbnail}
              alt={product.name}
              fill
              className="object-cover"
            />
          </div>
          
          {product.images.length > 1 && (
            <div className="grid grid-cols-4 gap-2">
              {product.images.map((image, index) => (
                <button
                  key={index}
                  onClick={() => setSelectedImage(index)}
                  className={`aspect-square relative overflow-hidden rounded border-2 transition-colors ${
                    selectedImage === index ? 'border-blue-500' : 'border-gray-200'
                  }`}
                >
                  <Image
                    src={image}
                    alt={`${product.name} ${index + 1}`}
                    fill
                    className="object-cover"
                  />
                </button>
              ))}
            </div>
          )}
        </div>

        {/* Product Info */}
        <div className="space-y-6">
          <div>
            <h1 className="text-3xl font-bold text-gray-900 mb-2">{product.name}</h1>
            <p className="text-gray-600">{product.seller.storeName}</p>
          </div>

          {/* Rating */}
          <div className="flex items-center gap-4">
            <div className="flex items-center gap-1">
              {[...Array(5)].map((_, i) => (
                <Star
                  key={i}
                  className={`w-5 h-5 ${
                    i < Math.floor(averageRating)
                      ? 'text-yellow-400 fill-yellow-400'
                      : 'text-gray-300'
                  }`}
                />
              ))}
            </div>
            <span className="text-sm text-gray-600">
              {averageRating.toFixed(1)} ({product.reviews.length} รีวิว)
            </span>
          </div>

          {/* Price */}
          <div className="space-y-2">
            <div className="flex items-center gap-3">
              <span className="text-3xl font-bold text-gray-900">
                ฿{currentPrice.toLocaleString()}
              </span>
              {product.comparePrice && (
                <span className="text-xl text-gray-500 line-through">
                  ฿{product.comparePrice.toLocaleString()}
                </span>
              )}
            </div>
            {product.comparePrice && (
              <p className="text-green-600 font-medium">
                ประหยัด ฿{(product.comparePrice - currentPrice).toLocaleString()}
              </p>
            )}
          </div>

          {/* Variants */}
          {product.variants.length > 0 && (
            <ProductVariantSelector
              variants={product.variants}
              selectedVariant={selectedVariant}
              onVariantChange={setSelectedVariant}
            />
          )}

          {/* Quantity and Add to Cart */}
          <div className="space-y-4">
            <div className="flex items-center gap-4">
              <span className="text-sm font-medium">จำนวน:</span>
              <QuantitySelector
                value={quantity}
                onChange={setQuantity}
                max={currentInventory}
              />
              <span className="text-sm text-gray-600">
                เหลือ {currentInventory} ชิ้น
              </span>
            </div>

            <div className="flex gap-3">
              <Button 
                onClick={handleAddToCart}
                className="flex-1"
                disabled={currentInventory === 0}
              >
                เพิ่มลงตะกร้า
              </Button>
              <Button variant="outline" size="icon">
                <Heart className="w-4 h-4" />
              </Button>
              <Button variant="outline" size="icon">
                <Share2 className="w-4 h-4" />
              </Button>
            </div>
          </div>

          {/* Features */}
          <div className="grid grid-cols-3 gap-4 p-4 bg-gray-50 rounded-lg">
            <div className="flex items-center gap-2">
              <Truck className="w-5 h-5 text-blue-600" />
              <span className="text-sm">จัดส่งฟรี</span>
            </div>
            <div className="flex items-center gap-2">
              <Shield className="w-5 h-5 text-green-600" />
              <span className="text-sm">รับประกัน</span>
            </div>
            <div className="flex items-center gap-2">
              <RotateCcw className="w-5 h-5 text-orange-600" />
              <span className="text-sm">คืนสินค้าได้</span>
            </div>
          </div>
        </div>
      </div>

      {/* Product Details Tabs */}
      <Tabs defaultValue="description" className="w-full">
        <TabsList className="grid w-full grid-cols-3">
          <TabsTrigger value="description">รายละเอียด</TabsTrigger>
          <TabsTrigger value="specifications">สเปค</TabsTrigger>
          <TabsTrigger value="reviews">รีวิว ({product.reviews.length})</TabsTrigger>
        </TabsList>
        
        <TabsContent value="description" className="mt-6">
          <div className="prose max-w-none">
            <div dangerouslySetInnerHTML={{ __html: product.description }} />
          </div>
        </TabsContent>
        
        <TabsContent value="specifications" className="mt-6">
          <div className="space-y-4">
            <div className="grid grid-cols-2 gap-4">
              <div>
                <span className="font-medium">SKU:</span>
                <span className="ml-2">{product.sku}</span>
              </div>
              <div>
                <span className="font-medium">น้ำหนัก:</span>
                <span className="ml-2">{product.weight} กรัม</span>
              </div>
              {product.dimensions && (
                <div>
                  <span className="font-medium">ขนาด:</span>
                  <span className="ml-2">
                    {product.dimensions.length} x {product.dimensions.width} x {product.dimensions.height} cm
                  </span>
                </div>
              )}
            </div>
          </div>
        </TabsContent>
        
        <TabsContent value="reviews" className="mt-6">
          <ProductReviews reviews={product.reviews} />
        </TabsContent>
      </Tabs>
    </div>
  );
}
```

### **🛒 Shopping Cart System**

```tsx
// src/store/useCart.ts
import { create } from 'zustand';
import { persist, devtools } from 'zustand/middleware';
import { immer } from 'zustand/middleware/immer';

interface CartItem {
  id: string;
  product: Product;
  variant?: ProductVariant;
  quantity: number;
  price: number;
}

interface CartState {
  items: CartItem[];
  total: number;
  itemCount: number;
  isOpen: boolean;
  
  // Actions
  addItem: (product: Product, quantity: number, variant?: ProductVariant) => void;
  removeItem: (itemId: string) => void;
  updateQuantity: (itemId: string, quantity: number) => void;
  clearCart: () => void;
  toggleCart: () => void;
  calculateTotal: () => void;
}

export const useCart = create<CartState>()(
  devtools(
    persist(
      immer((set, get) => ({
        items: [],
        total: 0,
        itemCount: 0,
        isOpen: false,

        addItem: (product, quantity, variant) => {
          set((state) => {
            const existingItemIndex = state.items.findIndex(
              item => item.product.id === product.id && 
                      item.variant?.id === variant?.id
            );

            const price = variant?.price || product.price;

            if (existingItemIndex >= 0) {
              state.items[existingItemIndex].quantity += quantity;
            } else {
              const newItem: CartItem = {
                id: `${product.id}-${variant?.id || 'default'}`,
                product,
                variant,
                quantity,
                price
              };
              state.items.push(newItem);
            }

            // Recalculate totals
            state.total = state.items.reduce(
              (sum, item) => sum + (item.price * item.quantity), 
              0
            );
            state.itemCount = state.items.reduce(
              (sum, item) => sum + item.quantity, 
              0
            );
          });
        },

        removeItem: (itemId) => {
          set((state) => {
            state.items = state.items.filter(item => item.id !== itemId);
            
            state.total = state.items.reduce(
              (sum, item) => sum + (item.price * item.quantity), 
              0
            );
            state.itemCount = state.items.reduce(
              (sum, item) => sum + item.quantity, 
              0
            );
          });
        },

        updateQuantity: (itemId, quantity) => {
          set((state) => {
            if (quantity <= 0) {
              state.items = state.items.filter(item => item.id !== itemId);
            } else {
              const item = state.items.find(item => item.id === itemId);
              if (item) {
                item.quantity = quantity;
              }
            }

            state.total = state.items.reduce(
              (sum, item) => sum + (item.price * item.quantity), 
              0
            );
            state.itemCount = state.items.reduce(
              (sum, item) => sum + item.quantity, 
              0
            );
          });
        },

        clearCart: () => {
          set((state) => {
            state.items = [];
            state.total = 0;
            state.itemCount = 0;
          });
        },

        toggleCart: () => {
          set((state) => {
            state.isOpen = !state.isOpen;
          });
        },

        calculateTotal: () => {
          set((state) => {
            state.total = state.items.reduce(
              (sum, item) => sum + (item.price * item.quantity), 
              0
            );
            state.itemCount = state.items.reduce(
              (sum, item) => sum + item.quantity, 
              0
            );
          });
        }
      })),
      {
        name: 'cart-storage',
        partialize: (state) => ({
          items: state.items,
          total: state.total,
          itemCount: state.itemCount
        })
      }
    ),
    { name: 'Cart Store' }
  )
);

// src/components/cart/CartDrawer.tsx
import { Fragment } from 'react';
import { Dialog, Transition } from '@headlessui/react';
import { X, Minus, Plus, ShoppingBag } from 'lucide-react';
import Image from 'next/image';
import Link from 'next/link';
import { Button } from '@/components/ui/Button';
import { useCart } from '@/store/useCart';

export function CartDrawer() {
  const { 
    items, 
    total, 
    itemCount, 
    isOpen, 
    toggleCart, 
    updateQuantity, 
    removeItem 
  } = useCart();

  return (
    <Transition.Root show={isOpen} as={Fragment}>
      <Dialog as="div" className="relative z-50" onClose={toggleCart}>
        <Transition.Child
          as={Fragment}
          enter="ease-in-out duration-500"
          enterFrom="opacity-0"
          enterTo="opacity-100"
          leave="ease-in-out duration-500"
          leaveFrom="opacity-100"
          leaveTo="opacity-0"
        >
          <div className="fixed inset-0 bg-gray-500 bg-opacity-75 transition-opacity" />
        </Transition.Child>

        <div className="fixed inset-0 overflow-hidden">
          <div className="absolute inset-0 overflow-hidden">
            <div className="pointer-events-none fixed inset-y-0 right-0 flex max-w-full pl-10">
              <Transition.Child
                as={Fragment}
                enter="transform transition ease-in-out duration-500 sm:duration-700"
                enterFrom="translate-x-full"
                enterTo="translate-x-0"
                leave="transform transition ease-in-out duration-500 sm:duration-700"
                leaveFrom="translate-x-0"
                leaveTo="translate-x-full"
              >
                <Dialog.Panel className="pointer-events-auto w-screen max-w-md">
                  <div className="flex h-full flex-col overflow-y-scroll bg-white shadow-xl">
                    <div className="flex-1 overflow-y-auto px-4 py-6 sm:px-6">
                      <div className="flex items-start justify-between">
                        <Dialog.Title className="text-lg font-medium text-gray-900">
                          ตะกร้าสินค้า ({itemCount})
                        </Dialog.Title>
                        <div className="ml-3 flex h-7 items-center">
                          <button
                            type="button"
                            className="-m-2 p-2 text-gray-400 hover:text-gray-500"
                            onClick={toggleCart}
                          >
                            <X className="h-6 w-6" />
                          </button>
                        </div>
                      </div>

                      <div className="mt-8">
                        <div className="flow-root">
                          {items.length === 0 ? (
                            <div className="text-center py-12">
                              <ShoppingBag className="mx-auto h-12 w-12 text-gray-400" />
                              <h3 className="mt-2 text-sm font-medium text-gray-900">
                                ตะกร้าว่าง
                              </h3>
                              <p className="mt-1 text-sm text-gray-500">
                                เริ่มซื้อสินค้าแล้วมาดูที่นี่
                              </p>
                            </div>
                          ) : (
                            <ul className="-my-6 divide-y divide-gray-200">
                              {items.map((item) => (
                                <li key={item.id} className="flex py-6">
                                  <div className="h-24 w-24 flex-shrink-0 overflow-hidden rounded-md border border-gray-200">
                                    <Image
                                      src={item.product.thumbnail}
                                      alt={item.product.name}
                                      width={96}
                                      height={96}
                                      className="h-full w-full object-cover object-center"
                                    />
                                  </div>

                                  <div className="ml-4 flex flex-1 flex-col">
                                    <div>
                                      <div className="flex justify-between text-base font-medium text-gray-900">
                                        <h3>
                                          <Link href={`/products/${item.product.slug}`}>
                                            {item.product.name}
                                          </Link>
                                        </h3>
                                        <p className="ml-4">
                                          ฿{(item.price * item.quantity).toLocaleString()}
                                        </p>
                                      </div>
                                      {item.variant && (
                                        <p className="mt-1 text-sm text-gray-500">
                                          {Object.entries(item.variant.options as Record<string, string>)
                                            .map(([key, value]) => `${key}: ${value}`)
                                            .join(', ')}
                                        </p>
                                      )}
                                    </div>
                                    <div className="flex flex-1 items-end justify-between text-sm">
                                      <div className="flex items-center space-x-2">
                                        <Button
                                          variant="outline"
                                          size="sm"
                                          onClick={() => updateQuantity(item.id, item.quantity - 1)}
                                          disabled={item.quantity <= 1}
                                        >
                                          <Minus className="h-3 w-3" />
                                        </Button>
                                        <span className="px-2">{item.quantity}</span>
                                        <Button
                                          variant="outline"
                                          size="sm"
                                          onClick={() => updateQuantity(item.id, item.quantity + 1)}
                                        >
                                          <Plus className="h-3 w-3" />
                                        </Button>
                                      </div>

                                      <div className="flex">
                                        <button
                                          type="button"
                                          className="font-medium text-red-600 hover:text-red-500"
                                          onClick={() => removeItem(item.id)}
                                        >
                                          ลบ
                                        </button>
                                      </div>
                                    </div>
                                  </div>
                                </li>
                              ))}
                            </ul>
                          )}
                        </div>
                      </div>
                    </div>

                    {items.length > 0 && (
                      <div className="border-t border-gray-200 px-4 py-6 sm:px-6">
                        <div className="flex justify-between text-base font-medium text-gray-900">
                          <p>รวมทั้งหมด</p>
                          <p>฿{total.toLocaleString()}</p>
                        </div>
                        <p className="mt-0.5 text-sm text-gray-500">
                          ราคายังไม่รวมค่าจัดส่งและภาษี
                        </p>
                        <div className="mt-6">
                          <Link
                            href="/checkout"
                            className="flex items-center justify-center rounded-md border border-transparent bg-blue-600 px-6 py-3 text-base font-medium text-white shadow-sm hover:bg-blue-700 w-full"
                            onClick={toggleCart}
                          >
                            ชำระเงิน
                          </Link>
                        </div>
                        <div className="mt-6 flex justify-center text-center text-sm text-gray-500">
                          <p>
                            หรือ{' '}
                            <button
                              type="button"
                              className="font-medium text-blue-600 hover:text-blue-500"
                              onClick={toggleCart}
                            >
                              เลือกซื้อต่อ
                            </button>
                          </p>
                        </div>
                      </div>
                    )}
                  </div>
                </Dialog.Panel>
              </Transition.Child>
            </div>
          </div>
        </div>
      </Dialog>
    </Transition.Root>
  );
}
```

### **💳 Checkout Process**

```tsx
// src/app/checkout/page.tsx
'use client';

import { useState } from 'react';
import { useRouter } from 'next/navigation';
import { useCart } from '@/store/useCart';
import { useAuth } from '@/store/useAuth';
import { CheckoutForm } from '@/components/checkout/CheckoutForm';
import { OrderSummary } from '@/components/checkout/OrderSummary';
import { PaymentMethod } from '@/components/checkout/PaymentMethod';
import { ShippingMethod } from '@/components/checkout/ShippingMethod';
import { AddressForm } from '@/components/checkout/AddressForm';
import { Button } from '@/components/ui/Button';
import { Card } from '@/components/ui/Card';
import { Stepper } from '@/components/ui/Stepper';

const steps = [
  { id: 'shipping', name: 'ที่อยู่จัดส่ง', description: 'กรอกที่อยู่สำหรับการจัดส่ง' },
  { id: 'delivery', name: 'วิธีจัดส่ง', description: 'เลือกวิธีการจัดส่งสินค้า' },
  { id: 'payment', name: 'การชำระเงิน', description: 'เลือกวิธีการชำระเงิน' },
  { id: 'review', name: 'ตรวจสอบคำสั่งซื้อ', description: 'ตรวจสอบข้อมูลก่อนสั่งซื้อ' }
];

export default function CheckoutPage() {
  const router = useRouter();
  const { user } = useAuth();
  const { items, total, clearCart } = useCart();
  const [currentStep, setCurrentStep] = useState(0);
  const [loading, setLoading] = useState(false);
  
  const [formData, setFormData] = useState({
    shippingAddress: null,
    shippingMethod: null,
    paymentMethod: null
  });

  // Redirect if cart is empty
  if (items.length === 0) {
    router.push('/');
    return null;
  }

  // Redirect if not authenticated
  if (!user) {
    router.push('/login?redirect=/checkout');
    return null;
  }

  const handleNext = () => {
    if (currentStep < steps.length - 1) {
      setCurrentStep(currentStep + 1);
    }
  };

  const handlePrevious = () => {
    if (currentStep > 0) {
      setCurrentStep(currentStep - 1);
    }
  };

  const handleSubmitOrder = async () => {
    setLoading(true);
    
    try {
      const orderData = {
        items: items.map(item => ({
          productId: item.product.id,
          variantId: item.variant?.id,
          quantity: item.quantity,
          price: item.price
        })),
        shippingAddress: formData.shippingAddress,
        shippingMethod: formData.shippingMethod,
        paymentMethod: formData.paymentMethod,
        subtotal: total,
        total: total + (formData.shippingMethod?.cost || 0)
      };

      const response = await fetch('/api/orders', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(orderData)
      });

      if (!response.ok) {
        throw new Error('Failed to create order');
      }

      const order = await response.json();
      
      // Clear cart
      clearCart();
      
      // Redirect to success page
      router.push(`/orders/${order.id}/success`);
      
    } catch (error) {
      console.error('Order submission failed:', error);
      alert('เกิดข้อผิดพลาดในการสั่งซื้อ กรุณาลองใหม่อีกครั้ง');
    } finally {
      setLoading(false);
    }
  };

  const isStepComplete = (stepIndex: number): boolean => {
    switch (stepIndex) {
      case 0: return !!formData.shippingAddress;
      case 1: return !!formData.shippingMethod;
      case 2: return !!formData.paymentMethod;
      default: return false;
    }
  };

  const canProceed = isStepComplete(currentStep);

  return (
    <div className="max-w-7xl mx-auto px-4 py-8">
      <h1 className="text-3xl font-bold text-gray-900 mb-8">ชำระเงิน</h1>

      <div className="grid grid-cols-1 lg:grid-cols-3 gap-8">
        {/* Main Content */}
        <div className="lg:col-span-2 space-y-6">
          {/* Progress Stepper */}
          <Stepper 
            steps={steps} 
            currentStep={currentStep}
            isStepComplete={isStepComplete}
          />

          {/* Step Content */}
          <Card className="p-6">
            {currentStep === 0 && (
              <AddressForm
                selectedAddress={formData.shippingAddress}
                onAddressSelect={(address) => 
                  setFormData(prev => ({ ...prev, shippingAddress: address }))
                }
              />
            )}

            {currentStep === 1 && (
              <ShippingMethod
                selectedMethod={formData.shippingMethod}
                onMethodSelect={(method) =>
                  setFormData(prev => ({ ...prev, shippingMethod: method }))
                }
              />
            )}

            {currentStep === 2 && (
              <PaymentMethod
                selectedMethod={formData.paymentMethod}
                onMethodSelect={(method) =>
                  setFormData(prev => ({ ...prev, paymentMethod: method }))
                }
              />
            )}

            {currentStep === 3 && (
              <div className="space-y-6">
                <h2 className="text-xl font-semibold">ตรวจสอบคำสั่งซื้อ</h2>
                
                {/* Order Review */}
                <div className="space-y-4">
                  <div className="border rounded-lg p-4">
                    <h3 className="font-medium mb-2">ที่อยู่จัดส่ง</h3>
                    <p className="text-sm text-gray-600">
                      {formData.shippingAddress?.firstName} {formData.shippingAddress?.lastName}<br />
                      {formData.shippingAddress?.address1}<br />
                      {formData.shippingAddress?.city}, {formData.shippingAddress?.state} {formData.shippingAddress?.zipCode}
                    </p>
                  </div>

                  <div className="border rounded-lg p-4">
                    <h3 className="font-medium mb-2">วิธีจัดส่ง</h3>
                    <p className="text-sm text-gray-600">
                      {formData.shippingMethod?.name} - ฿{formData.shippingMethod?.cost}
                    </p>
                  </div>

                  <div className="border rounded-lg p-4">
                    <h3 className="font-medium mb-2">การชำระเงิน</h3>
                    <p className="text-sm text-gray-600">
                      {formData.paymentMethod?.name}
                    </p>
                  </div>
                </div>
              </div>
            )}

            {/* Navigation Buttons */}
            <div className="flex justify-between mt-8">
              <Button
                variant="outline"
                onClick={handlePrevious}
                disabled={currentStep === 0}
              >
                ย้อนกลับ
              </Button>

              {currentStep < steps.length - 1 ? (
                <Button
                  onClick={handleNext}
                  disabled={!canProceed}
                >
                  ถัดไป
                </Button>
              ) : (
                <Button
                  onClick={handleSubmitOrder}
                  disabled={!canProceed || loading}
                  className="bg-green-600 hover:bg-green-700"
                >
                  {loading ? 'กำลังสั่งซื้อ...' : 'สั่งซื้อ'}
                </Button>
              )}
            </div>
          </Card>
        </div>

        {/* Order Summary */}
        <div className="lg:col-span-1">
          <OrderSummary 
            items={items}
            subtotal={total}
            shippingCost={formData.shippingMethod?.cost || 0}
            total={total + (formData.shippingMethod?.cost || 0)}
          />
        </div>
      </div>
    </div>
  );
}

// src/app/api/orders/route.ts - Order creation API
import { NextRequest, NextResponse } from 'next/server';
import { prisma } from '@/lib/prisma';
import { getServerSession } from 'next-auth';
import { authOptions } from '@/lib/auth';

export async function POST(request: NextRequest) {
  try {
    const session = await getServerSession(authOptions);
    
    if (!session?.user) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
    }

    const {
      items,
      shippingAddress,
      shippingMethod,
      paymentMethod,
      subtotal,
      total
    } = await request.json();

    // Create order in transaction
    const order = await prisma.$transaction(async (tx) => {
      // Generate order number
      const orderNumber = `ORD-${Date.now()}`;

      // Create order
      const newOrder = await tx.order.create({
        data: {
          orderNumber,
          userId: session.user.id,
          status: 'PENDING',
          subtotal,
          taxAmount: 0,
          shippingCost: shippingMethod.cost,
          total,
          shippingAddressId: shippingAddress.id,
          shippingMethod: shippingMethod.name,
          paymentMethod: paymentMethod.name,
          paymentStatus: 'PENDING',
          fulfillmentStatus: 'PENDING'
        }
      });

      // Create order items
      for (const item of items) {
        await tx.orderItem.create({
          data: {
            orderId: newOrder.id,
            productId: item.productId,
            variantId: item.variantId,
            quantity: item.quantity,
            price: item.price,
            total: item.price * item.quantity,
            productName: item.productName,
            productSku: item.productSku
          }
        });

        // Update inventory
        if (item.variantId) {
          await tx.productVariant.update({
            where: { id: item.variantId },
            data: { inventory: { decrement: item.quantity } }
          });
        } else {
          await tx.product.update({
            where: { id: item.productId },
            data: { inventory: { decrement: item.quantity } }
          });
        }
      }

      return newOrder;
    });

    // Process payment if required
    if (paymentMethod.type === 'card') {
      // Integrate with payment processor (Stripe, Omise, etc.)
      // This is a placeholder for payment processing
    }

    return NextResponse.json(order);
  } catch (error) {
    console.error('Order creation failed:', error);
    return NextResponse.json(
      { error: 'Failed to create order' },
      { status: 500 }
    );
  }
}
```

## 📝 แบบฝึกหัด

### **🎯 แบบฝึกหัดที่ 1: E-commerce Features**
1. สร้าง product catalog system
2. เพิ่ม shopping cart functionality
3. ตั้งค่า checkout process
4. เพิ่ม order management system

### **🎯 แบบฝึกหัดที่ 2: User Management**
1. สร้าง authentication system
2. เพิ่ม user profiles และ addresses
3. ตั้งค่า seller dashboard
4. เพิ่ม admin panel

### **🎯 แบบฝึกหัดที่ 3: Payment Integration**
1. เชื่อมต่อ payment gateway
2. เพิ่ม payment processing
3. ตั้งค่า refund system
4. ทดสอบ payment flows

### **🎯 แบบฝึกหัดที่ 4: Advanced Features**
1. เพิ่ม search และ filtering
2. สร้าง recommendation system
3. ตั้งค่า inventory management
4. เพิ่ม analytics และ reporting

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 22: Advanced Patterns and Architecture](./22-advanced-patterns.md)

**บทต่อไป**: [บทที่ 24: Performance Optimization](./24-performance-optimization.md)

**กลับไปหน้าหลัก**: [README](./README.md)

---

## 📚 เอกสารอ้างอิง

- [Next.js E-commerce Examples](https://nextjs.org/commerce)
- [Prisma E-commerce Schema](https://github.com/prisma/prisma-examples/tree/latest/typescript/ecommerce)
- [Stripe Integration](https://stripe.com/docs/payments/accept-a-payment)
- [Next.js App Router](https://nextjs.org/docs/app)
