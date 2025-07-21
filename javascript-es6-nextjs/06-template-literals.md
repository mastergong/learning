# บทที่ 6: Template Literals & String Methods

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ Template Literals และ String Interpolation
- ทำความเข้าใจ Multiline Strings และ Tagged Templates
- เรียนรู้ Modern String Methods ใน ES6+
- ประยุกต์ใช้ใน Next.js สำหรับ Dynamic Content

## 📝 Template Literals

### **🔧 Basic Template Literals**

```javascript
// ============= TRADITIONAL STRING CONCATENATION =============
const name = 'John';
const age = 30;
const city = 'Bangkok';

// ES5 way - ยุ่งยากและอ่านยาก
var message = 'Hello, my name is ' + name + '. I am ' + age + ' years old and I live in ' + city + '.';

// ============= ES6+ TEMPLATE LITERALS =============
const messageModern = `Hello, my name is ${name}. I am ${age} years old and I live in ${city}.`;

// ============= MULTILINE STRINGS =============
// ES5 way - ต้องใช้ \n และ concatenation
var htmlOld = '<div class="user-card">\n' +
              '  <h3>' + name + '</h3>\n' +
              '  <p>Age: ' + age + '</p>\n' +
              '  <p>City: ' + city + '</p>\n' +
              '</div>';

// ES6+ way - สะอาดและอ่านง่าย
const htmlModern = `
  <div class="user-card">
    <h3>${name}</h3>
    <p>Age: ${age}</p>
    <p>City: ${city}</p>
  </div>
`;

// ============= EXPRESSION EVALUATION =============
const price = 1000;
const taxRate = 0.07;
const discount = 100;

const invoice = `
  Subtotal: $${price}
  Tax (${(taxRate * 100).toFixed(1)}%): $${(price * taxRate).toFixed(2)}
  Discount: -$${discount}
  Total: $${(price + (price * taxRate) - discount).toFixed(2)}
`;

// ============= FUNCTION CALLS IN TEMPLATES =============
const users = ['John', 'Jane', 'Bob'];

const formatUserList = (users) => {
  return users.map(user => `• ${user}`).join('\n');
};

const userReport = `
  User Report (${new Date().toLocaleDateString()})
  Total Users: ${users.length}
  
  ${formatUserList(users)}
`;

// ============= CONDITIONAL EXPRESSIONS =============
const user = {
  name: 'John',
  premium: true,
  credits: 150
};

const welcomeMessage = `
  Welcome back, ${user.name}!
  
  ${user.premium ? '⭐ Premium Member' : '👤 Regular Member'}
  
  You have ${user.credits} credit${user.credits !== 1 ? 's' : ''} remaining.
  
  ${user.credits > 100 ? 'You have plenty of credits!' : 
    user.credits > 50 ? 'You have a good amount of credits.' : 
    'Consider purchasing more credits.'}
`;

// ============= NESTED TEMPLATES =============
const products = [
  { id: 1, name: 'Laptop', price: 25000, inStock: true },
  { id: 2, name: 'Mouse', price: 500, inStock: false },
  { id: 3, name: 'Keyboard', price: 1500, inStock: true }
];

const productList = `
  <div class="product-list">
    ${products.map(product => `
      <div class="product ${product.inStock ? 'in-stock' : 'out-of-stock'}">
        <h3>${product.name}</h3>
        <p class="price">$${product.price.toLocaleString()}</p>
        <span class="status">
          ${product.inStock ? '✅ In Stock' : '❌ Out of Stock'}
        </span>
      </div>
    `).join('')}
  </div>
`;
```

### **🏷️ Tagged Template Literals**

```javascript
// ============= BASIC TAGGED TEMPLATES =============
function highlight(strings, ...values) {
  return strings.reduce((result, string, index) => {
    const value = values[index] ? `<mark>${values[index]}</mark>` : '';
    return result + string + value;
  }, '');
}

const searchTerm = 'JavaScript';
const description = 'learning';

const highlightedText = highlight`I am ${description} ${searchTerm} programming.`;
// Result: "I am <mark>learning</mark> <mark>JavaScript</mark> programming."

// ============= ADVANCED TAGGED TEMPLATES =============
function sql(strings, ...values) {
  // ป้องกัน SQL injection โดย escape values
  const escapedValues = values.map(value => {
    if (typeof value === 'string') {
      return `'${value.replace(/'/g, "''")}'`;
    }
    if (typeof value === 'number') {
      return value.toString();
    }
    if (value === null || value === undefined) {
      return 'NULL';
    }
    return `'${String(value)}'`;
  });

  return strings.reduce((query, string, index) => {
    const value = escapedValues[index] || '';
    return query + string + value;
  }, '');
}

const userId = 123;
const userName = "John O'Connor";
const email = 'john@example.com';

const query = sql`
  SELECT * FROM users 
  WHERE id = ${userId} 
    AND name = ${userName} 
    AND email = ${email}
`;

// ============= INTERNATIONALIZATION (I18N) =============
function i18n(strings, ...values) {
  // Mock translation function
  const translations = {
    'Hello': 'สวัสดี',
    'world': 'โลก',
    'Welcome': 'ยินดีต้อนรับ'
  };

  return strings.reduce((result, string, index) => {
    const value = values[index] || '';
    const translatedString = translations[string.trim()] || string;
    return result + translatedString + value;
  }, '');
}

const greeting = i18n`Hello ${name}!`;
const welcome = i18n`Welcome ${'back'}!`;

// ============= STYLED COMPONENTS PATTERN =============
function css(strings, ...values) {
  return strings.reduce((styles, string, index) => {
    const value = values[index] || '';
    return styles + string + value;
  }, '');
}

const primaryColor = '#007bff';
const borderRadius = '4px';

const buttonStyles = css`
  background-color: ${primaryColor};
  border-radius: ${borderRadius};
  padding: 10px 20px;
  color: white;
  border: none;
  cursor: pointer;
  
  &:hover {
    background-color: ${primaryColor}cc;
  }
`;

// ============= VALIDATION TAGGED TEMPLATE =============
function validate(strings, ...values) {
  const errors = [];
  
  values.forEach((value, index) => {
    if (value === null || value === undefined || value === '') {
      errors.push(`Value at position ${index} is empty`);
    }
  });
  
  if (errors.length > 0) {
    throw new Error(`Validation failed: ${errors.join(', ')}`);
  }
  
  return strings.reduce((result, string, index) => {
    const value = values[index] || '';
    return result + string + value;
  }, '');
}

try {
  const validatedMessage = validate`Hello ${name}, welcome to ${city}!`;
  console.log(validatedMessage);
} catch (error) {
  console.error(error.message);
}
```

## 🔤 Modern String Methods

### **🔍 ES6+ String Methods**

```javascript
// ============= STRING SEARCHING =============
const text = 'JavaScript is awesome and JavaScript is powerful';

// includes() - check if string contains substring
console.log(text.includes('JavaScript')); // true
console.log(text.includes('Python')); // false
console.log(text.includes('javascript')); // false (case-sensitive)

// startsWith() - check if string starts with substring
console.log(text.startsWith('JavaScript')); // true
console.log(text.startsWith('Java')); // true
console.log(text.startsWith('Script')); // false

// endsWith() - check if string ends with substring
console.log(text.endsWith('powerful')); // true
console.log(text.endsWith('ful')); // true
console.log(text.endsWith('awesome')); // false

// Position-based searching
console.log(text.startsWith('is', 11)); // true (starts from position 11)
console.log(text.endsWith('JavaScript', 23)); // true (checks first 23 chars)

// ============= STRING REPETITION =============
const separator = '-'.repeat(50);
console.log(separator); // '--------------------------------------------------'

const loading = 'Loading' + '.'.repeat(3);
console.log(loading); // 'Loading...'

// Create indentation
const indent = (level) => ' '.repeat(level * 2);
const code = `
${indent(0)}function example() {
${indent(1)}if (true) {
${indent(2)}console.log('Hello');
${indent(1)}}
${indent(0)}}
`;

// ============= STRING PADDING =============
const numbers = [1, 22, 333, 4444];

// padStart() - pad at the beginning
const paddedNumbers = numbers.map(num => 
  num.toString().padStart(6, '0')
);
console.log(paddedNumbers); // ['000001', '000022', '000333', '004444']

// padEnd() - pad at the end
const labels = ['Name:', 'Email:', 'Phone:', 'Address:'];
const paddedLabels = labels.map(label => 
  label.padEnd(12, ' ')
);
console.log(paddedLabels);
// ['Name:       ', 'Email:      ', 'Phone:      ', 'Address:    ']

// Practical example: align table data
const tableData = [
  ['John Doe', 'john@example.com', '123-456-7890'],
  ['Jane Smith', 'jane@example.com', '098-765-4321'],
  ['Bob Johnson', 'bob@example.com', '555-123-4567']
];

const formatTable = (data) => {
  return data.map(([name, email, phone]) => 
    `${name.padEnd(15)} ${email.padEnd(20)} ${phone}`
  ).join('\n');
};

console.log(formatTable(tableData));

// ============= STRING TRIMMING =============
const userInput = '   hello world   ';

// Original trim() - removes whitespace from both ends
console.log(userInput.trim()); // 'hello world'

// ES2019+ methods
console.log(userInput.trimStart()); // 'hello world   '
console.log(userInput.trimEnd());   // '   hello world'

// Practical usage in form validation
const validateInput = (input) => {
  const trimmed = input.trim();
  
  if (!trimmed) {
    return { isValid: false, message: 'Input cannot be empty' };
  }
  
  if (trimmed.length < 3) {
    return { isValid: false, message: 'Input must be at least 3 characters' };
  }
  
  return { isValid: true, value: trimmed };
};
```

### **🔧 Advanced String Processing**

```javascript
// ============= STRING REPLACEMENT PATTERNS =============
const template = 'Hello {{name}}, welcome to {{city}}! Today is {{date}}.';

// Simple template replacement
const replaceTemplate = (template, data) => {
  return template.replace(/\{\{(\w+)\}\}/g, (match, key) => {
    return data[key] || match;
  });
};

const data = {
  name: 'John',
  city: 'Bangkok',
  date: new Date().toLocaleDateString()
};

const message = replaceTemplate(template, data);
console.log(message); // 'Hello John, welcome to Bangkok! Today is 1/1/2024.'

// ============= ADVANCED TEXT PROCESSING =============
class TextProcessor {
  static slugify(text) {
    return text
      .toLowerCase()
      .trim()
      .replace(/[^\w\s-]/g, '') // Remove special characters
      .replace(/[\s_-]+/g, '-') // Replace spaces and underscores with hyphens
      .replace(/^-+|-+$/g, ''); // Remove leading/trailing hyphens
  }

  static capitalize(text) {
    return text.charAt(0).toUpperCase() + text.slice(1).toLowerCase();
  }

  static titleCase(text) {
    return text
      .split(' ')
      .map(word => this.capitalize(word))
      .join(' ');
  }

  static truncate(text, maxLength, suffix = '...') {
    if (text.length <= maxLength) {
      return text;
    }
    
    return text.slice(0, maxLength - suffix.length).trim() + suffix;
  }

  static stripHtml(html) {
    return html.replace(/<[^>]*>/g, '');
  }

  static extractEmails(text) {
    const emailRegex = /\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b/g;
    return text.match(emailRegex) || [];
  }

  static formatPhoneNumber(phone) {
    // Remove all non-digit characters
    const digits = phone.replace(/\D/g, '');
    
    // Format as (XXX) XXX-XXXX
    if (digits.length === 10) {
      return `(${digits.slice(0, 3)}) ${digits.slice(3, 6)}-${digits.slice(6)}`;
    }
    
    return phone; // Return original if not 10 digits
  }

  static generateExcerpt(text, maxWords = 30) {
    const words = text.trim().split(/\s+/);
    
    if (words.length <= maxWords) {
      return text;
    }
    
    return words.slice(0, maxWords).join(' ') + '...';
  }
}

// Usage examples
console.log(TextProcessor.slugify('Hello World! This is a Test.')); 
// 'hello-world-this-is-a-test'

console.log(TextProcessor.titleCase('hello world from javascript'));
// 'Hello World From Javascript'

console.log(TextProcessor.truncate('This is a very long text that needs to be truncated', 20));
// 'This is a very lo...'

console.log(TextProcessor.extractEmails('Contact us at john@example.com or jane@test.org'));
// ['john@example.com', 'jane@test.org']

// ============= STRING COMPARISON =============
class StringUtils {
  static levenshteinDistance(str1, str2) {
    const matrix = [];
    
    for (let i = 0; i <= str2.length; i++) {
      matrix[i] = [i];
    }
    
    for (let j = 0; j <= str1.length; j++) {
      matrix[0][j] = j;
    }
    
    for (let i = 1; i <= str2.length; i++) {
      for (let j = 1; j <= str1.length; j++) {
        if (str2.charAt(i - 1) === str1.charAt(j - 1)) {
          matrix[i][j] = matrix[i - 1][j - 1];
        } else {
          matrix[i][j] = Math.min(
            matrix[i - 1][j - 1] + 1, // substitution
            matrix[i][j - 1] + 1,     // insertion
            matrix[i - 1][j] + 1      // deletion
          );
        }
      }
    }
    
    return matrix[str2.length][str1.length];
  }

  static similarity(str1, str2) {
    const distance = this.levenshteinDistance(str1, str2);
    const maxLength = Math.max(str1.length, str2.length);
    return maxLength === 0 ? 1 : (maxLength - distance) / maxLength;
  }

  static fuzzySearch(query, items, threshold = 0.6) {
    return items
      .map(item => ({
        item,
        score: this.similarity(query.toLowerCase(), item.toLowerCase())
      }))
      .filter(({ score }) => score >= threshold)
      .sort((a, b) => b.score - a.score)
      .map(({ item }) => item);
  }
}

// Usage
const searchItems = ['JavaScript', 'TypeScript', 'CoffeeScript', 'ActionScript'];
const results = StringUtils.fuzzySearch('script', searchItems);
console.log(results); // ['JavaScript', 'TypeScript', 'CoffeeScript', 'ActionScript']
```

## 🚀 Next.js Applications

### **🌐 Dynamic Content Generation**

```jsx
// components/ProductCard.jsx
import { useState } from 'react';

const ProductCard = ({ product }) => {
  const [quantity, setQuantity] = useState(1);
  
  const {
    id,
    name,
    price,
    originalPrice,
    description,
    images,
    rating,
    reviews,
    inStock,
    discount
  } = product;

  // Template literals สำหรับ dynamic classes
  const cardClasses = `
    product-card 
    ${inStock ? 'in-stock' : 'out-of-stock'}
    ${discount > 0 ? 'on-sale' : ''}
  `.trim().replace(/\s+/g, ' ');

  // Generate product URL
  const productUrl = `/products/${TextProcessor.slugify(name)}-${id}`;
  
  // Format price with currency
  const formatPrice = (price) => `฿${price.toLocaleString()}`;
  
  // Calculate savings
  const savings = originalPrice - price;
  const savingsPercentage = ((savings / originalPrice) * 100).toFixed(0);
  
  // Generate schema.org JSON-LD
  const productSchema = `{
    "@context": "https://schema.org/",
    "@type": "Product",
    "name": "${name}",
    "description": "${description}",
    "sku": "${id}",
    "image": "${images[0]}",
    "offers": {
      "@type": "Offer",
      "url": "${productUrl}",
      "priceCurrency": "THB",
      "price": "${price}",
      "availability": "${inStock ? 'https://schema.org/InStock' : 'https://schema.org/OutOfStock'}"
    },
    "aggregateRating": {
      "@type": "AggregateRating",
      "ratingValue": "${rating}",
      "reviewCount": "${reviews}"
    }
  }`;

  // Generate meta description
  const metaDescription = TextProcessor.generateExcerpt(
    `${name} - ${description} Price: ${formatPrice(price)}`, 
    25
  );

  return (
    <>
      <script 
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: productSchema }}
      />
      
      <div className={cardClasses}>
        <div className="product-image">
          <img 
            src={images[0]} 
            alt={`${name} - Product Image`}
            loading="lazy"
          />
          
          {discount > 0 && (
            <div className="discount-badge">
              -{savingsPercentage}%
            </div>
          )}
        </div>

        <div className="product-info">
          <h3 className="product-name">{name}</h3>
          
          <p className="product-description">
            {TextProcessor.truncate(description, 100)}
          </p>

          <div className="product-rating">
            <span className="stars">
              {'★'.repeat(Math.floor(rating))}
              {'☆'.repeat(5 - Math.floor(rating))}
            </span>
            <span className="rating-text">
              {rating} ({reviews} reviews)
            </span>
          </div>

          <div className="product-pricing">
            <span className="current-price">
              {formatPrice(price)}
            </span>
            
            {originalPrice > price && (
              <>
                <span className="original-price">
                  {formatPrice(originalPrice)}
                </span>
                <span className="savings">
                  Save {formatPrice(savings)}
                </span>
              </>
            )}
          </div>

          <div className="product-stock">
            {inStock ? (
              <span className="in-stock">✅ In Stock</span>
            ) : (
              <span className="out-of-stock">❌ Out of Stock</span>
            )}
          </div>

          {inStock && (
            <div className="product-actions">
              <div className="quantity-selector">
                <label>Quantity:</label>
                <select 
                  value={quantity}
                  onChange={(e) => setQuantity(Number(e.target.value))}
                >
                  {Array.from({ length: 10 }, (_, i) => (
                    <option key={i + 1} value={i + 1}>
                      {i + 1}
                    </option>
                  ))}
                </select>
              </div>

              <button 
                className="add-to-cart"
                onClick={() => addToCart(id, quantity)}
              >
                Add to Cart - {formatPrice(price * quantity)}
              </button>
            </div>
          )}
        </div>
      </div>
    </>
  );
};

// components/SearchResults.jsx
import { useState, useMemo } from 'react';

const SearchResults = ({ products, searchQuery }) => {
  const [sortBy, setSortBy] = useState('relevance');
  const [filterBy, setFilterBy] = useState('all');

  // Filter และ sort products
  const processedProducts = useMemo(() => {
    let filtered = products;

    // Filter by availability
    if (filterBy === 'in-stock') {
      filtered = filtered.filter(product => product.inStock);
    } else if (filterBy === 'on-sale') {
      filtered = filtered.filter(product => product.discount > 0);
    }

    // Sort products
    const sorted = [...filtered].sort((a, b) => {
      switch (sortBy) {
        case 'price-low':
          return a.price - b.price;
        case 'price-high':
          return b.price - a.price;
        case 'rating':
          return b.rating - a.rating;
        case 'name':
          return a.name.localeCompare(b.name);
        case 'relevance':
        default:
          // Simple relevance scoring based on search query
          const scoreA = StringUtils.similarity(searchQuery.toLowerCase(), a.name.toLowerCase());
          const scoreB = StringUtils.similarity(searchQuery.toLowerCase(), b.name.toLowerCase());
          return scoreB - scoreA;
      }
    });

    return sorted;
  }, [products, searchQuery, sortBy, filterBy]);

  // Generate search results summary
  const searchSummary = useMemo(() => {
    const total = processedProducts.length;
    const inStock = processedProducts.filter(p => p.inStock).length;
    const onSale = processedProducts.filter(p => p.discount > 0).length;

    return `
      Found ${total} result${total !== 1 ? 's' : ''} for "${searchQuery}"
      ${inStock > 0 ? `• ${inStock} in stock` : ''}
      ${onSale > 0 ? `• ${onSale} on sale` : ''}
    `.trim();
  }, [processedProducts, searchQuery]);

  // Generate page title
  const pageTitle = `Search Results for "${searchQuery}" - ${processedProducts.length} Products Found`;

  return (
    <div className="search-results">
      <div className="search-header">
        <h1>{pageTitle}</h1>
        <p className="search-summary">{searchSummary}</p>
      </div>

      <div className="search-controls">
        <div className="sort-controls">
          <label>Sort by:</label>
          <select value={sortBy} onChange={(e) => setSortBy(e.target.value)}>
            <option value="relevance">Relevance</option>
            <option value="price-low">Price: Low to High</option>
            <option value="price-high">Price: High to Low</option>
            <option value="rating">Customer Rating</option>
            <option value="name">Name A-Z</option>
          </select>
        </div>

        <div className="filter-controls">
          <label>Filter:</label>
          <select value={filterBy} onChange={(e) => setFilterBy(e.target.value)}>
            <option value="all">All Products</option>
            <option value="in-stock">In Stock Only</option>
            <option value="on-sale">On Sale Only</option>
          </select>
        </div>
      </div>

      <div className="products-grid">
        {processedProducts.length > 0 ? (
          processedProducts.map(product => (
            <ProductCard key={product.id} product={product} />
          ))
        ) : (
          <div className="no-results">
            <h3>No products found</h3>
            <p>Try adjusting your search criteria or filters.</p>
          </div>
        )}
      </div>
    </div>
  );
};

export default SearchResults;
```

### **📧 Email Template Generation**

```javascript
// utils/emailTemplates.js
class EmailTemplateGenerator {
  static generateWelcomeEmail({ name, email, verificationLink }) {
    const subject = `Welcome to our platform, ${name}!`;
    
    const html = `
      <!DOCTYPE html>
      <html lang="en">
      <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>${subject}</title>
        <style>
          body { 
            font-family: Arial, sans-serif; 
            line-height: 1.6; 
            color: #333; 
            max-width: 600px; 
            margin: 0 auto; 
            padding: 20px;
          }
          .header { 
            background: #007bff; 
            color: white; 
            padding: 20px; 
            text-align: center; 
            border-radius: 8px 8px 0 0;
          }
          .content { 
            background: #f8f9fa; 
            padding: 30px; 
            border-radius: 0 0 8px 8px;
          }
          .button { 
            background: #28a745; 
            color: white; 
            padding: 12px 24px; 
            text-decoration: none; 
            border-radius: 4px; 
            display: inline-block; 
            margin: 20px 0;
          }
          .footer { 
            text-align: center; 
            color: #666; 
            font-size: 12px; 
            margin-top: 30px;
          }
        </style>
      </head>
      <body>
        <div class="header">
          <h1>Welcome to Our Platform!</h1>
        </div>
        
        <div class="content">
          <h2>Hello ${name},</h2>
          
          <p>Thank you for joining our community! We're excited to have you on board.</p>
          
          <p>Your account has been created with the email address: <strong>${email}</strong></p>
          
          <p>To get started, please verify your email address by clicking the button below:</p>
          
          <a href="${verificationLink}" class="button">Verify Email Address</a>
          
          <p>If the button doesn't work, you can copy and paste this link into your browser:</p>
          <p><a href="${verificationLink}">${verificationLink}</a></p>
          
          <p>If you didn't create this account, please ignore this email.</p>
          
          <p>Best regards,<br>The Team</p>
        </div>
        
        <div class="footer">
          <p>This email was sent to ${email}. If you no longer wish to receive these emails, you can unsubscribe.</p>
        </div>
      </body>
      </html>
    `;

    const text = `
      Welcome to Our Platform!
      
      Hello ${name},
      
      Thank you for joining our community! We're excited to have you on board.
      
      Your account has been created with the email address: ${email}
      
      To get started, please verify your email address by visiting this link:
      ${verificationLink}
      
      If you didn't create this account, please ignore this email.
      
      Best regards,
      The Team
    `.trim();

    return { subject, html, text };
  }

  static generatePasswordResetEmail({ name, email, resetLink, expiresIn }) {
    const subject = 'Reset Your Password';
    
    const html = `
      <!DOCTYPE html>
      <html lang="en">
      <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>${subject}</title>
        <style>
          body { 
            font-family: Arial, sans-serif; 
            line-height: 1.6; 
            color: #333; 
            max-width: 600px; 
            margin: 0 auto; 
            padding: 20px;
          }
          .header { 
            background: #dc3545; 
            color: white; 
            padding: 20px; 
            text-align: center; 
            border-radius: 8px 8px 0 0;
          }
          .content { 
            background: #f8f9fa; 
            padding: 30px; 
            border-radius: 0 0 8px 8px;
          }
          .button { 
            background: #dc3545; 
            color: white; 
            padding: 12px 24px; 
            text-decoration: none; 
            border-radius: 4px; 
            display: inline-block; 
            margin: 20px 0;
          }
          .warning { 
            background: #fff3cd; 
            border: 1px solid #ffeaa7; 
            padding: 15px; 
            border-radius: 4px; 
            margin: 20px 0;
          }
        </style>
      </head>
      <body>
        <div class="header">
          <h1>🔒 Password Reset Request</h1>
        </div>
        
        <div class="content">
          <h2>Hello ${name},</h2>
          
          <p>We received a request to reset the password for your account (${email}).</p>
          
          <p>Click the button below to create a new password:</p>
          
          <a href="${resetLink}" class="button">Reset Password</a>
          
          <div class="warning">
            <strong>⚠️ Important:</strong> This link will expire in ${expiresIn}. 
            If you don't reset your password within this time, you'll need to request a new reset link.
          </div>
          
          <p>If you didn't request this password reset, please ignore this email. Your password will remain unchanged.</p>
          
          <p>For security reasons, this link can only be used once.</p>
          
          <p>Best regards,<br>The Security Team</p>
        </div>
      </body>
      </html>
    `;

    return { subject, html };
  }

  static generateOrderConfirmation({ 
    customerName, 
    orderNumber, 
    orderDate, 
    items, 
    total, 
    shippingAddress,
    estimatedDelivery 
  }) {
    const subject = `Order Confirmation #${orderNumber}`;
    
    const itemsList = items.map(item => `
      <tr>
        <td>${item.name}</td>
        <td>$${item.price.toFixed(2)}</td>
        <td>${item.quantity}</td>
        <td>$${(item.price * item.quantity).toFixed(2)}</td>
      </tr>
    `).join('');

    const html = `
      <!DOCTYPE html>
      <html lang="en">
      <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>${subject}</title>
        <style>
          body { 
            font-family: Arial, sans-serif; 
            line-height: 1.6; 
            color: #333; 
            max-width: 600px; 
            margin: 0 auto; 
            padding: 20px;
          }
          .header { 
            background: #28a745; 
            color: white; 
            padding: 20px; 
            text-align: center; 
            border-radius: 8px 8px 0 0;
          }
          .content { 
            background: #f8f9fa; 
            padding: 30px; 
            border-radius: 0 0 8px 8px;
          }
          .order-summary { 
            background: white; 
            padding: 20px; 
            border-radius: 4px; 
            margin: 20px 0;
          }
          table { 
            width: 100%; 
            border-collapse: collapse; 
            margin: 15px 0;
          }
          th, td { 
            padding: 10px; 
            text-align: left; 
            border-bottom: 1px solid #ddd;
          }
          th { 
            background: #f1f1f1; 
            font-weight: bold;
          }
          .total { 
            font-size: 18px; 
            font-weight: bold; 
            color: #28a745;
          }
        </style>
      </head>
      <body>
        <div class="header">
          <h1>✅ Order Confirmed!</h1>
          <p>Order #${orderNumber}</p>
        </div>
        
        <div class="content">
          <h2>Thank you for your order, ${customerName}!</h2>
          
          <p>Your order has been confirmed and is being processed. Here are the details:</p>
          
          <div class="order-summary">
            <h3>Order Summary</h3>
            <p><strong>Order Number:</strong> ${orderNumber}</p>
            <p><strong>Order Date:</strong> ${orderDate}</p>
            <p><strong>Estimated Delivery:</strong> ${estimatedDelivery}</p>
            
            <table>
              <thead>
                <tr>
                  <th>Item</th>
                  <th>Price</th>
                  <th>Quantity</th>
                  <th>Total</th>
                </tr>
              </thead>
              <tbody>
                ${itemsList}
              </tbody>
            </table>
            
            <p class="total">Total: $${total.toFixed(2)}</p>
          </div>
          
          <div class="order-summary">
            <h3>Shipping Address</h3>
            <p>
              ${shippingAddress.name}<br>
              ${shippingAddress.street}<br>
              ${shippingAddress.city}, ${shippingAddress.state} ${shippingAddress.zipCode}<br>
              ${shippingAddress.country}
            </p>
          </div>
          
          <p>We'll send you another email when your order ships with tracking information.</p>
          
          <p>Thank you for shopping with us!</p>
        </div>
      </body>
      </html>
    `;

    return { subject, html };
  }
}

// pages/api/send-email.js
import { EmailTemplateGenerator } from '../../utils/emailTemplates';

export default async function handler(req, res) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { type, data } = req.body;

  try {
    let emailContent;

    switch (type) {
      case 'welcome':
        emailContent = EmailTemplateGenerator.generateWelcomeEmail(data);
        break;
      case 'password-reset':
        emailContent = EmailTemplateGenerator.generatePasswordResetEmail(data);
        break;
      case 'order-confirmation':
        emailContent = EmailTemplateGenerator.generateOrderConfirmation(data);
        break;
      default:
        return res.status(400).json({ error: 'Invalid email type' });
    }

    // Send email using your preferred email service
    // await sendEmail(data.email, emailContent);

    res.status(200).json({ 
      message: 'Email sent successfully',
      preview: emailContent.html // For development/testing
    });

  } catch (error) {
    console.error('Email sending error:', error);
    res.status(500).json({ error: 'Failed to send email' });
  }
}
```

## 🎯 แบบฝึกหัด

### **🔧 แบบฝึกหัดที่ 1: Template Literals Practice**
สร้างฟังก์ชันที่ใช้ template literals:

```javascript
// TODO: สร้างฟังก์ชันเหล่านี้โดยใช้ template literals

// 1. สร้าง invoice template
function generateInvoice(invoice) {
  // invoice = {
  //   invoiceNumber: 'INV-001',
  //   date: '2024-01-01',
  //   customer: { name: 'John Doe', email: 'john@example.com' },
  //   items: [{ name: 'Product 1', price: 100, quantity: 2 }],
  //   tax: 0.07
  // }
  // Return formatted invoice string
}

// 2. สร้าง SQL query builder
function buildSelectQuery({ table, columns = ['*'], where = {}, orderBy, limit }) {
  // Return SQL SELECT statement
}

// 3. สร้าง HTML table from data
function generateTable(data, options = {}) {
  // data = [{ name: 'John', age: 30 }, { name: 'Jane', age: 25 }]
  // options = { className: 'table', headers: true }
  // Return HTML table string
}
```

### **🔤 แบบฝึกหัดที่ 2: String Methods**
ใช้ modern string methods:

```javascript
// TODO: สร้าง utility functions

// 1. Text search และ highlight
function highlightMatches(text, searchTerm) {
  // Highlight all occurrences of searchTerm in text
  // Return text with <mark> tags around matches
}

// 2. Format text สำหรับ display
function formatDisplayText(text, options = {}) {
  // options = { maxLength, truncate, capitalize, stripHtml }
  // Apply formatting based on options
}

// 3. Validate และ format inputs
function validateAndFormatInput(input, type) {
  // type = 'email', 'phone', 'url', 'creditCard'
  // Return { isValid, formatted, errors }
}

// 4. Generate URL-friendly slugs
function createSlug(title, options = {}) {
  // options = { maxLength, separator }
  // Convert title to URL-friendly slug
}
```

### **⚛️ แบบฝึกหัดที่ 3: Next.js Component**
สร้าง component ที่ใช้ template literals:

```jsx
// TODO: สร้าง BlogPost component ที่:
// 1. แสดง blog post content
// 2. Generate meta tags dynamically
// 3. Create table of contents จาก headings
// 4. Format publish date และ reading time
// 5. Generate social sharing links

const BlogPost = ({ post }) => {
  // post = {
  //   title: 'My Blog Post',
  //   content: 'Long content with headings...',
  //   author: { name: 'John Doe' },
  //   publishDate: '2024-01-01',
  //   tags: ['javascript', 'nextjs']
  // }
  
  // Implementation here
};
```

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 5: Spread & Rest Operators](./05-spread-rest.md)

**บทถัดไป**: [บทที่ 7: Modern Array Methods](./07-array-methods.md)

**กลับไปหน้าหลัก**: [README](../README.md)

---

## 📚 เอกสารอ้างอิง

- [MDN: Template Literals](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals)
- [MDN: String Methods](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)
- [JavaScript.info: Strings](https://javascript.info/string)

---

## 🎉 สรุป

ในบทนี้เราได้เรียนรู้:

- **Template Literals** สำหรับ string interpolation
- **Tagged Templates** สำหรับ advanced processing
- **Modern String Methods** ใน ES6+
- **Text Processing Utilities** สำหรับการจัดการข้อความ
- **การใช้งานใน Next.js** สำหรับ dynamic content

Template literals และ string methods ช่วยให้การจัดการข้อความใน JavaScript มีประสิทธิภาพและอ่านง่ายมากขึ้น! 🚀
