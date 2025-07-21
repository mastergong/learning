# บทที่ 5: Components and Styling

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้การสร้างและใช้งาน React Components ใน Next.js
- ทำความเข้าใจ CSS Modules และ CSS-in-JS
- เรียนรู้ Tailwind CSS integration
- เข้าใจ Global styles และ Component styles

## 🧩 React Components ใน Next.js

### **⚛️ Functional Components**

```jsx
// components/Button.js
import styles from './Button.module.css';

export default function Button({ 
  children, 
  variant = 'primary', 
  size = 'medium',
  disabled = false,
  onClick,
  ...props 
}) {
  const className = `
    ${styles.button} 
    ${styles[variant]} 
    ${styles[size]} 
    ${disabled ? styles.disabled : ''}
  `.trim();

  return (
    <button 
      className={className}
      disabled={disabled}
      onClick={onClick}
      {...props}
    >
      {children}
    </button>
  );
}

// การใช้งาน
export default function HomePage() {
  return (
    <div>
      <Button variant="primary" size="large">
        Get Started
      </Button>
      <Button variant="secondary" size="small">
        Learn More
      </Button>
      <Button variant="danger" disabled>
        Delete
      </Button>
    </div>
  );
}
```

### **🎨 Component Props และ Types**

```jsx
// components/Card.js
import PropTypes from 'prop-types';
import styles from './Card.module.css';

export default function Card({ 
  title, 
  description, 
  image, 
  tags = [], 
  author,
  publishedAt,
  children 
}) {
  return (
    <article className={styles.card}>
      {image && (
        <div className={styles.imageContainer}>
          <img src={image} alt={title} />
        </div>
      )}
      
      <div className={styles.content}>
        <h3 className={styles.title}>{title}</h3>
        <p className={styles.description}>{description}</p>
        
        {tags.length > 0 && (
          <div className={styles.tags}>
            {tags.map((tag) => (
              <span key={tag} className={styles.tag}>
                {tag}
              </span>
            ))}
          </div>
        )}
        
        {author && publishedAt && (
          <div className={styles.meta}>
            <span className={styles.author}>By {author}</span>
            <span className={styles.date}>
              {new Date(publishedAt).toLocaleDateString()}
            </span>
          </div>
        )}
        
        {children}
      </div>
    </article>
  );
}

// PropTypes validation
Card.propTypes = {
  title: PropTypes.string.isRequired,
  description: PropTypes.string.isRequired,
  image: PropTypes.string,
  tags: PropTypes.arrayOf(PropTypes.string),
  author: PropTypes.string,
  publishedAt: PropTypes.string,
  children: PropTypes.node
};
```

### **🔄 Higher-Order Components (HOC)**

```jsx
// hoc/withAuth.js
import { useEffect, useState } from 'react';
import { useRouter } from 'next/router';

export default function withAuth(WrappedComponent) {
  return function AuthenticatedComponent(props) {
    const [isAuthenticated, setIsAuthenticated] = useState(false);
    const [isLoading, setIsLoading] = useState(true);
    const router = useRouter();

    useEffect(() => {
      const checkAuth = async () => {
        try {
          const token = localStorage.getItem('token');
          if (!token) {
            router.push('/login');
            return;
          }

          const response = await fetch('/api/auth/verify', {
            headers: { Authorization: `Bearer ${token}` }
          });

          if (response.ok) {
            setIsAuthenticated(true);
          } else {
            router.push('/login');
          }
        } catch (error) {
          console.error('Auth check failed:', error);
          router.push('/login');
        } finally {
          setIsLoading(false);
        }
      };

      checkAuth();
    }, [router]);

    if (isLoading) {
      return <div>Loading...</div>;
    }

    if (!isAuthenticated) {
      return null;
    }

    return <WrappedComponent {...props} />;
  };
}

// การใช้งาน
import withAuth from '@/hoc/withAuth';

function Dashboard() {
  return (
    <div>
      <h1>Dashboard</h1>
      <p>This is a protected page</p>
    </div>
  );
}

export default withAuth(Dashboard);
```

## 🎨 CSS Modules

### **📁 CSS Modules Setup**

```css
/* components/Button.module.css */
.button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 0.375rem;
  font-weight: 500;
  text-decoration: none;
  cursor: pointer;
  transition: all 0.2s ease-in-out;
  font-family: inherit;
}

.button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

/* Variants */
.primary {
  background-color: #3b82f6;
  color: white;
}

.primary:hover:not(:disabled) {
  background-color: #2563eb;
}

.secondary {
  background-color: #f3f4f6;
  color: #374151;
  border: 1px solid #d1d5db;
}

.secondary:hover:not(:disabled) {
  background-color: #e5e7eb;
}

.danger {
  background-color: #ef4444;
  color: white;
}

.danger:hover:not(:disabled) {
  background-color: #dc2626;
}

/* Sizes */
.small {
  padding: 0.375rem 0.75rem;
  font-size: 0.875rem;
}

.medium {
  padding: 0.5rem 1rem;
  font-size: 1rem;
}

.large {
  padding: 0.75rem 1.5rem;
  font-size: 1.125rem;
}

.disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
```

### **🎯 Advanced CSS Modules**

```css
/* components/Card.module.css */
.card {
  background: white;
  border-radius: 0.5rem;
  box-shadow: 0 1px 3px 0 rgba(0, 0, 0, 0.1);
  overflow: hidden;
  transition: all 0.3s ease;
}

.card:hover {
  box-shadow: 0 10px 25px 0 rgba(0, 0, 0, 0.15);
  transform: translateY(-2px);
}

.imageContainer {
  position: relative;
  width: 100%;
  height: 200px;
  overflow: hidden;
}

.imageContainer img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.3s ease;
}

.card:hover .imageContainer img {
  transform: scale(1.05);
}

.content {
  padding: 1.5rem;
}

.title {
  font-size: 1.25rem;
  font-weight: 600;
  margin-bottom: 0.5rem;
  color: #1f2937;
}

.description {
  color: #6b7280;
  line-height: 1.6;
  margin-bottom: 1rem;
}

.tags {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
  margin-bottom: 1rem;
}

.tag {
  display: inline-block;
  padding: 0.25rem 0.5rem;
  background-color: #e5e7eb;
  color: #374151;
  border-radius: 0.25rem;
  font-size: 0.875rem;
  font-weight: 500;
}

.meta {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 0.875rem;
  color: #6b7280;
  border-top: 1px solid #e5e7eb;
  padding-top: 1rem;
}

.author {
  font-weight: 500;
}

.date {
  font-style: italic;
}

/* Responsive Design */
@media (max-width: 768px) {
  .content {
    padding: 1rem;
  }
  
  .title {
    font-size: 1.125rem;
  }
  
  .meta {
    flex-direction: column;
    align-items: flex-start;
    gap: 0.25rem;
  }
}
```

## 💅 CSS-in-JS with Styled Components

### **📦 Installation**

```bash
npm install styled-components
npm install --save-dev babel-plugin-styled-components
```

```javascript
// .babelrc
{
  "presets": ["next/babel"],
  "plugins": [["styled-components", { "ssr": true }]]
}

// pages/_document.js
import Document from 'next/document';
import { ServerStyleSheet } from 'styled-components';

export default class MyDocument extends Document {
  static async getInitialProps(ctx) {
    const sheet = new ServerStyleSheet();
    const originalRenderPage = ctx.renderPage;

    try {
      ctx.renderPage = () =>
        originalRenderPage({
          enhanceApp: (App) => (props) =>
            sheet.collectStyles(<App {...props} />),
        });

      const initialProps = await Document.getInitialProps(ctx);
      return {
        ...initialProps,
        styles: (
          <>
            {initialProps.styles}
            {sheet.getStyleElement()}
          </>
        ),
      };
    } finally {
      sheet.seal();
    }
  }
}
```

### **🎨 Styled Components Examples**

```jsx
// components/StyledButton.js
import styled, { css } from 'styled-components';

const StyledButton = styled.button`
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 0.375rem;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease-in-out;
  font-family: inherit;

  ${props => props.disabled && css`
    opacity: 0.5;
    cursor: not-allowed;
  `}

  ${props => {
    switch (props.variant) {
      case 'primary':
        return css`
          background-color: #3b82f6;
          color: white;
          &:hover:not(:disabled) {
            background-color: #2563eb;
          }
        `;
      case 'secondary':
        return css`
          background-color: #f3f4f6;
          color: #374151;
          border: 1px solid #d1d5db;
          &:hover:not(:disabled) {
            background-color: #e5e7eb;
          }
        `;
      case 'danger':
        return css`
          background-color: #ef4444;
          color: white;
          &:hover:not(:disabled) {
            background-color: #dc2626;
          }
        `;
      default:
        return css`
          background-color: #f3f4f6;
          color: #374151;
        `;
    }
  }}

  ${props => {
    switch (props.size) {
      case 'small':
        return css`
          padding: 0.375rem 0.75rem;
          font-size: 0.875rem;
        `;
      case 'large':
        return css`
          padding: 0.75rem 1.5rem;
          font-size: 1.125rem;
        `;
      default:
        return css`
          padding: 0.5rem 1rem;
          font-size: 1rem;
        `;
    }
  }}
`;

export default function Button({ children, ...props }) {
  return <StyledButton {...props}>{children}</StyledButton>;
}
```

### **🎭 Styled Components Theme**

```jsx
// styles/theme.js
export const lightTheme = {
  colors: {
    primary: '#3b82f6',
    secondary: '#6b7280',
    success: '#10b981',
    danger: '#ef4444',
    warning: '#f59e0b',
    info: '#06b6d4',
    light: '#f8fafc',
    dark: '#1e293b',
    text: {
      primary: '#1f2937',
      secondary: '#6b7280',
      inverse: '#ffffff'
    },
    background: {
      primary: '#ffffff',
      secondary: '#f8fafc',
      dark: '#1e293b'
    }
  },
  spacing: {
    xs: '0.25rem',
    sm: '0.5rem',
    md: '1rem',
    lg: '1.5rem',
    xl: '2rem',
    xxl: '3rem'
  },
  borderRadius: {
    sm: '0.25rem',
    md: '0.375rem',
    lg: '0.5rem',
    xl: '0.75rem',
    full: '9999px'
  },
  shadows: {
    sm: '0 1px 2px 0 rgba(0, 0, 0, 0.05)',
    md: '0 4px 6px -1px rgba(0, 0, 0, 0.1)',
    lg: '0 10px 15px -3px rgba(0, 0, 0, 0.1)',
    xl: '0 20px 25px -5px rgba(0, 0, 0, 0.1)'
  }
};

export const darkTheme = {
  ...lightTheme,
  colors: {
    ...lightTheme.colors,
    text: {
      primary: '#f8fafc',
      secondary: '#cbd5e1',
      inverse: '#1e293b'
    },
    background: {
      primary: '#1e293b',
      secondary: '#334155',
      dark: '#0f172a'
    }
  }
};

// pages/_app.js
import { ThemeProvider } from 'styled-components';
import { lightTheme, darkTheme } from '@/styles/theme';
import { useState } from 'react';

export default function App({ Component, pageProps }) {
  const [isDark, setIsDark] = useState(false);

  return (
    <ThemeProvider theme={isDark ? darkTheme : lightTheme}>
      <Component {...pageProps} />
    </ThemeProvider>
  );
}
```

## 🌈 Tailwind CSS

### **📦 Installation**

```bash
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

```javascript
// tailwind.config.js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    './pages/**/*.{js,ts,jsx,tsx,mdx}',
    './components/**/*.{js,ts,jsx,tsx,mdx}',
    './app/**/*.{js,ts,jsx,tsx,mdx}',
  ],
  theme: {
    extend: {
      colors: {
        primary: {
          50: '#eff6ff',
          100: '#dbeafe', 
          500: '#3b82f6',
          600: '#2563eb',
          700: '#1d4ed8',
          900: '#1e3a8a',
        },
        secondary: {
          50: '#f8fafc',
          100: '#f1f5f9',
          500: '#64748b',
          600: '#475569',
          700: '#334155',
          900: '#0f172a',
        }
      },
      fontFamily: {
        sans: ['Inter', 'sans-serif'],
        mono: ['Fira Code', 'monospace'],
      },
      spacing: {
        '18': '4.5rem',
        '88': '22rem',
      }
    },
  },
  plugins: [
    require('@tailwindcss/forms'),
    require('@tailwindcss/typography'),
  ],
}
```

```css
/* styles/globals.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  html {
    font-family: 'Inter', system-ui, sans-serif;
  }
  
  h1, h2, h3, h4, h5, h6 {
    @apply font-semibold text-gray-900;
  }
  
  a {
    @apply text-primary-600 hover:text-primary-700 transition-colors;
  }
}

@layer components {
  .btn {
    @apply inline-flex items-center justify-center px-4 py-2 border border-transparent text-sm font-medium rounded-md shadow-sm focus:outline-none focus:ring-2 focus:ring-offset-2 transition-all;
  }
  
  .btn-primary {
    @apply btn bg-primary-600 text-white hover:bg-primary-700 focus:ring-primary-500;
  }
  
  .btn-secondary {
    @apply btn bg-gray-200 text-gray-800 hover:bg-gray-300 focus:ring-gray-500;
  }
  
  .card {
    @apply bg-white rounded-lg shadow-md overflow-hidden hover:shadow-lg transition-shadow;
  }
  
  .input {
    @apply block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm placeholder-gray-400 focus:outline-none focus:ring-primary-500 focus:border-primary-500;
  }
}

@layer utilities {
  .text-balance {
    text-wrap: balance;
  }
  
  .scrollbar-hide {
    -ms-overflow-style: none;
    scrollbar-width: none;
  }
  
  .scrollbar-hide::-webkit-scrollbar {
    display: none;
  }
}
```

### **🎨 Tailwind Components**

```jsx
// components/TailwindButton.js
import { forwardRef } from 'react';

const Button = forwardRef(function Button(
  { 
    children, 
    variant = 'primary', 
    size = 'md',
    disabled = false,
    className = '',
    ...props 
  },
  ref
) {
  const baseClasses = `
    inline-flex items-center justify-center font-medium rounded-md 
    focus:outline-none focus:ring-2 focus:ring-offset-2 
    transition-all duration-200 disabled:opacity-50 disabled:cursor-not-allowed
  `;

  const variants = {
    primary: 'bg-primary-600 text-white hover:bg-primary-700 focus:ring-primary-500',
    secondary: 'bg-gray-200 text-gray-800 hover:bg-gray-300 focus:ring-gray-500',
    danger: 'bg-red-600 text-white hover:bg-red-700 focus:ring-red-500',
    outline: 'border-2 border-primary-600 text-primary-600 hover:bg-primary-50 focus:ring-primary-500',
    ghost: 'text-gray-600 hover:bg-gray-100 focus:ring-gray-500'
  };

  const sizes = {
    sm: 'px-3 py-1.5 text-sm',
    md: 'px-4 py-2 text-sm',
    lg: 'px-6 py-3 text-base',
    xl: 'px-8 py-4 text-lg'
  };

  const classes = `
    ${baseClasses} 
    ${variants[variant]} 
    ${sizes[size]} 
    ${className}
  `.trim();

  return (
    <button 
      ref={ref}
      className={classes}
      disabled={disabled}
      {...props}
    >
      {children}
    </button>
  );
});

export default Button;
```

### **🎯 Responsive Tailwind Components**

```jsx
// components/ResponsiveCard.js
export default function ResponsiveCard({ 
  title, 
  description, 
  image, 
  tags = [], 
  author,
  publishedAt 
}) {
  return (
    <article className="
      group bg-white rounded-xl shadow-sm hover:shadow-md 
      transition-all duration-300 overflow-hidden border border-gray-200
    ">
      {image && (
        <div className="aspect-w-16 aspect-h-9 lg:aspect-h-10 overflow-hidden">
          <img 
            src={image} 
            alt={title}
            className="
              w-full h-48 lg:h-56 object-cover 
              group-hover:scale-105 transition-transform duration-300
            "
          />
        </div>
      )}
      
      <div className="p-4 lg:p-6">
        <h3 className="
          text-lg lg:text-xl font-semibold text-gray-900 
          mb-2 group-hover:text-primary-600 transition-colors
        ">
          {title}
        </h3>
        
        <p className="text-gray-600 text-sm lg:text-base mb-4 line-clamp-3">
          {description}
        </p>
        
        {tags.length > 0 && (
          <div className="flex flex-wrap gap-2 mb-4">
            {tags.map((tag) => (
              <span 
                key={tag}
                className="
                  inline-block px-2 py-1 bg-primary-50 text-primary-700 
                  rounded-md text-xs font-medium
                "
              >
                {tag}
              </span>
            ))}
          </div>
        )}
        
        {(author || publishedAt) && (
          <div className="
            flex flex-col sm:flex-row sm:items-center sm:justify-between 
            pt-4 border-t border-gray-200 gap-2
          ">
            {author && (
              <span className="text-sm font-medium text-gray-700">
                By {author}
              </span>
            )}
            {publishedAt && (
              <span className="text-sm text-gray-500">
                {new Date(publishedAt).toLocaleDateString('th-TH')}
              </span>
            )}
          </div>
        )}
      </div>
    </article>
  );
}
```

## 🌍 Global Styles

### **📁 Global CSS Setup**

```css
/* styles/globals.css */
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@100;200;300;400;500;600;700;800;900&display=swap');

:root {
  /* Colors */
  --color-primary-50: #eff6ff;
  --color-primary-100: #dbeafe;
  --color-primary-500: #3b82f6;
  --color-primary-600: #2563eb;
  --color-primary-900: #1e3a8a;
  
  --color-gray-50: #f9fafb;
  --color-gray-100: #f3f4f6;
  --color-gray-500: #6b7280;
  --color-gray-600: #4b5563;
  --color-gray-900: #111827;
  
  /* Spacing */
  --spacing-xs: 0.25rem;
  --spacing-sm: 0.5rem;
  --spacing-md: 1rem;
  --spacing-lg: 1.5rem;
  --spacing-xl: 2rem;
  
  /* Typography */
  --font-size-xs: 0.75rem;
  --font-size-sm: 0.875rem;
  --font-size-base: 1rem;
  --font-size-lg: 1.125rem;
  --font-size-xl: 1.25rem;
  --font-size-2xl: 1.5rem;
  --font-size-3xl: 1.875rem;
  
  /* Shadows */
  --shadow-sm: 0 1px 2px 0 rgba(0, 0, 0, 0.05);
  --shadow-md: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
  --shadow-lg: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
  
  /* Border Radius */
  --radius-sm: 0.25rem;
  --radius-md: 0.375rem;
  --radius-lg: 0.5rem;
  --radius-xl: 0.75rem;
}

/* Dark mode variables */
[data-theme="dark"] {
  --color-gray-50: #111827;
  --color-gray-100: #1f2937;
  --color-gray-500: #9ca3af;
  --color-gray-600: #d1d5db;
  --color-gray-900: #f9fafb;
}

/* Reset และ Base Styles */
*,
*::before,
*::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

html {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  line-height: 1.6;
  scroll-behavior: smooth;
}

body {
  background-color: var(--color-gray-50);
  color: var(--color-gray-900);
  transition: background-color 0.3s ease, color 0.3s ease;
}

/* Typography */
h1, h2, h3, h4, h5, h6 {
  font-weight: 600;
  line-height: 1.3;
  margin-bottom: var(--spacing-md);
  color: var(--color-gray-900);
}

h1 { font-size: var(--font-size-3xl); }
h2 { font-size: var(--font-size-2xl); }
h3 { font-size: var(--font-size-xl); }
h4 { font-size: var(--font-size-lg); }
h5 { font-size: var(--font-size-base); }
h6 { font-size: var(--font-size-sm); }

p {
  margin-bottom: var(--spacing-md);
  color: var(--color-gray-600);
}

a {
  color: var(--color-primary-600);
  text-decoration: none;
  transition: color 0.2s ease;
}

a:hover {
  color: var(--color-primary-500);
  text-decoration: underline;
}

/* Utility Classes */
.container {
  width: 100%;
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 var(--spacing-md);
}

.text-center { text-align: center; }
.text-left { text-align: left; }
.text-right { text-align: right; }

.mb-0 { margin-bottom: 0; }
.mb-sm { margin-bottom: var(--spacing-sm); }
.mb-md { margin-bottom: var(--spacing-md); }
.mb-lg { margin-bottom: var(--spacing-lg); }
.mb-xl { margin-bottom: var(--spacing-xl); }

.mt-0 { margin-top: 0; }
.mt-sm { margin-top: var(--spacing-sm); }
.mt-md { margin-top: var(--spacing-md); }
.mt-lg { margin-top: var(--spacing-lg); }
.mt-xl { margin-top: var(--spacing-xl); }

/* Animation Classes */
.fade-in {
  animation: fadeIn 0.5s ease-in;
}

.slide-up {
  animation: slideUp 0.5s ease-out;
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes slideUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* Responsive Typography */
@media (max-width: 768px) {
  :root {
    --font-size-3xl: 1.5rem;
    --font-size-2xl: 1.25rem;
    --font-size-xl: 1.125rem;
  }
  
  .container {
    padding: 0 var(--spacing-sm);
  }
}

/* Print Styles */
@media print {
  * {
    box-shadow: none !important;
    text-shadow: none !important;
  }
  
  a {
    text-decoration: underline;
  }
  
  .no-print {
    display: none !important;
  }
}

/* Accessibility */
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}

.focus-visible {
  outline: 2px solid var(--color-primary-500);
  outline-offset: 2px;
}

/* Scrollbar Styling */
::-webkit-scrollbar {
  width: 8px;
}

::-webkit-scrollbar-track {
  background: var(--color-gray-100);
}

::-webkit-scrollbar-thumb {
  background: var(--color-gray-300);
  border-radius: var(--radius-md);
}

::-webkit-scrollbar-thumb:hover {
  background: var(--color-gray-400);
}
```

## 🎭 Component Library

### **📚 Component Index**

```jsx
// components/index.js
export { default as Button } from './Button';
export { default as Card } from './Card';
export { default as Modal } from './Modal';
export { default as Input } from './Input';
export { default as Select } from './Select';
export { default as Checkbox } from './Checkbox';
export { default as Radio } from './Radio';
export { default as Badge } from './Badge';
export { default as Alert } from './Alert';
export { default as Tooltip } from './Tooltip';
export { default as Spinner } from './Spinner';
export { default as Avatar } from './Avatar';
export { default as Breadcrumb } from './Breadcrumb';
export { default as Pagination } from './Pagination';
export { default as Tabs } from './Tabs';
export { default as Accordion } from './Accordion';
export { default as Dropdown } from './Dropdown';
export { default as Sidebar } from './Sidebar';
export { default as Navbar } from './Navbar';
export { default as Footer } from './Footer';
```

### **🔧 Compound Components**

```jsx
// components/Tabs/index.js
import { useState, createContext, useContext } from 'react';
import styles from './Tabs.module.css';

const TabsContext = createContext();

function Tabs({ children, defaultValue, value, onChange }) {
  const [activeTab, setActiveTab] = useState(defaultValue || value);

  const handleTabChange = (tabValue) => {
    if (onChange) {
      onChange(tabValue);
    } else {
      setActiveTab(tabValue);
    }
  };

  const currentTab = value !== undefined ? value : activeTab;

  return (
    <TabsContext.Provider value={{ activeTab: currentTab, onTabChange: handleTabChange }}>
      <div className={styles.tabs}>
        {children}
      </div>
    </TabsContext.Provider>
  );
}

function TabsList({ children }) {
  return (
    <div className={styles.tabsList} role="tablist">
      {children}
    </div>
  );
}

function TabsTrigger({ value, children, disabled = false }) {
  const { activeTab, onTabChange } = useContext(TabsContext);
  const isActive = activeTab === value;

  return (
    <button
      className={`${styles.tabsTrigger} ${isActive ? styles.active : ''} ${disabled ? styles.disabled : ''}`}
      onClick={() => !disabled && onTabChange(value)}
      disabled={disabled}
      role="tab"
      aria-selected={isActive}
      aria-controls={`tabpanel-${value}`}
    >
      {children}
    </button>
  );
}

function TabsContent({ value, children }) {
  const { activeTab } = useContext(TabsContext);
  const isActive = activeTab === value;

  if (!isActive) return null;

  return (
    <div
      className={styles.tabsContent}
      role="tabpanel"
      id={`tabpanel-${value}`}
      aria-labelledby={`tab-${value}`}
    >
      {children}
    </div>
  );
}

// Export compound component
Tabs.List = TabsList;
Tabs.Trigger = TabsTrigger;
Tabs.Content = TabsContent;

export default Tabs;

// การใช้งาน
export default function ExamplePage() {
  return (
    <Tabs defaultValue="tab1">
      <Tabs.List>
        <Tabs.Trigger value="tab1">Tab 1</Tabs.Trigger>
        <Tabs.Trigger value="tab2">Tab 2</Tabs.Trigger>
        <Tabs.Trigger value="tab3" disabled>Tab 3</Tabs.Trigger>
      </Tabs.List>
      
      <Tabs.Content value="tab1">
        <h3>Content for Tab 1</h3>
        <p>This is the content for the first tab.</p>
      </Tabs.Content>
      
      <Tabs.Content value="tab2">
        <h3>Content for Tab 2</h3>
        <p>This is the content for the second tab.</p>
      </Tabs.Content>
    </Tabs>
  );
}
```

## 📝 แบบฝึกหัด

### **🎯 แบบฝึกหัดที่ 1: Component Library**
1. สร้าง Button component ด้วย CSS Modules
2. เพิ่ม variant และ size props
3. เพิ่ม loading state และ icon support
4. ทดสอบ accessibility

### **🎯 แบบฝึกหัดที่ 2: Styled Components**
1. แปลง Button component เป็น Styled Components
2. สร้าง theme system
3. เพิ่ม dark mode support
4. สร้าง responsive components

### **🎯 แบบฝึกหัดที่ 3: Tailwind CSS**
1. Setup Tailwind CSS ในโปรเจค
2. สร้าง custom utility classes
3. สร้าง responsive card component
4. เพิ่ม custom colors และ fonts

### **🎯 แบบฝึกหัดที่ 4: Advanced Components**
1. สร้าง Modal component ด้วย Portal
2. เพิ่ม Form components (Input, Select, Checkbox)
3. สร้าง Dropdown menu
4. เพิ่ม Animation และ Transitions

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 4: Pages and Routing](./04-pages-routing.md)

**บทต่อไป**: [บทที่ 6: Data Fetching Basics](./06-data-fetching-basics.md)

**กลับไปหน้าหลัก**: [README](./README.md)

---

## 📚 เอกสารอ้างอิง

- [Next.js CSS Support](https://nextjs.org/docs/basic-features/built-in-css-support)
- [CSS Modules](https://nextjs.org/docs/basic-features/built-in-css-support#css-modules)
- [Styled Components](https://styled-components.com/)
- [Tailwind CSS](https://tailwindcss.com/)
