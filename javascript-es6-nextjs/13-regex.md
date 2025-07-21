# บทที่ 13: Regular Expressions (Regex)

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ Regular Expressions สำหรับ pattern matching
- ทำความเข้าใจ regex syntax และ flags
- เรียนรู้ advanced regex techniques
- ประยุกต์ใช้ regex ใน Next.js สำหรับ validation และ data processing

## 🔍 Regex Fundamentals

### **📝 Basic Syntax**

```javascript
// ============= REGEX CREATION =============
// Literal notation
const regex1 = /pattern/flags;
const regex2 = /hello/i; // Case-insensitive

// Constructor notation
const regex3 = new RegExp('pattern', 'flags');
const regex4 = new RegExp('hello', 'i');

// Dynamic patterns
const searchTerm = 'world';
const dynamicRegex = new RegExp(searchTerm, 'gi');

// ============= BASIC MATCHING =============
const text = "Hello World! This is a test.";

// test() - returns boolean
console.log(/hello/i.test(text)); // true
console.log(/goodbye/.test(text)); // false

// exec() - returns match details or null
const match = /world/i.exec(text);
console.log(match); 
// ['World', index: 6, input: 'Hello World! This is a test.', groups: undefined]

// String methods with regex
console.log(text.match(/world/i)); // ['World', index: 6, ...]
console.log(text.search(/world/i)); // 6
console.log(text.replace(/world/i, 'Universe')); // "Hello Universe! This is a test."
console.log(text.split(/\s+/)); // ['Hello', 'World!', 'This', 'is', 'a', 'test.']

// ============= FLAGS =============
const multilineText = `Line 1
Line 2
Line 3`;

// i - case insensitive
console.log(/HELLO/i.test("hello")); // true

// g - global (find all matches)
console.log("hello hello hello".match(/hello/)); // ['hello'] (first match only)
console.log("hello hello hello".match(/hello/g)); // ['hello', 'hello', 'hello']

// m - multiline (^ and $ match line boundaries)
console.log(/^Line/m.test(multilineText)); // true
console.log(/^Line/gm.exec(multilineText)); // matches each line

// s - dotall (. matches newlines)
console.log(/Line.*Line/s.test(multilineText)); // true

// u - unicode
console.log(/\u{1F600}/u.test("😀")); // true

// y - sticky (matches at lastIndex)
const stickyRegex = /\d+/y;
const numbers = "123 456 789";
stickyRegex.lastIndex = 0;
console.log(stickyRegex.exec(numbers)); // ['123']
console.log(stickyRegex.exec(numbers)); // null (no match at position 3)

// ============= CHARACTER CLASSES =============
const testString = "Hello123World!@#";

// Basic character classes
console.log(/\d/.test(testString)); // true - digits
console.log(/\w/.test(testString)); // true - word characters
console.log(/\s/.test("hello world")); // true - whitespace

// Negated character classes
console.log(/\D/.test("123")); // false - non-digits
console.log(/\W/.test("abc")); // false - non-word characters
console.log(/\S/.test(" ")); // false - non-whitespace

// Custom character classes
console.log(/[aeiou]/.test("hello")); // true - vowels
console.log(/[0-9]/.test("test123")); // true - digits
console.log(/[a-zA-Z]/.test("Hello")); // true - letters
console.log(/[^aeiou]/.test("hello")); // true - not vowels

// ============= QUANTIFIERS =============
const patterns = [
  { pattern: /a*/, text: "aaa", description: "0 or more" },
  { pattern: /a+/, text: "aaa", description: "1 or more" },
  { pattern: /a?/, text: "a", description: "0 or 1" },
  { pattern: /a{3}/, text: "aaa", description: "exactly 3" },
  { pattern: /a{2,4}/, text: "aaa", description: "2 to 4" },
  { pattern: /a{2,}/, text: "aaa", description: "2 or more" }
];

patterns.forEach(({ pattern, text, description }) => {
  const match = pattern.exec(text);
  console.log(`${description}: ${pattern} matches "${text}" = ${match ? match[0] : 'none'}`);
});

// Greedy vs lazy quantifiers
const greedyTest = "aaabbbccc";
console.log(/a+/.exec(greedyTest)); // ['aaa'] - greedy
console.log(/a+?/.exec(greedyTest)); // ['a'] - lazy

// ============= ANCHORS AND BOUNDARIES =============
const lines = ["hello", "world", "hello world"];

// ^ - start of string/line
lines.forEach(line => {
  console.log(`"${line}" starts with hello: ${/^hello/.test(line)}`);
});

// $ - end of string/line
lines.forEach(line => {
  console.log(`"${line}" ends with world: ${/world$/.test(line)}`);
});

// \b - word boundary
const boundaryText = "cat catch dog";
console.log(boundaryText.match(/\bcat\b/)); // ['cat'] - matches "cat" but not "catch"
console.log(boundaryText.match(/cat/g)); // ['cat', 'cat'] - matches both

// \B - non-word boundary
console.log(boundaryText.match(/\Bcat/)); // ['cat'] from "catch"
```

### **🔧 Advanced Patterns**

```javascript
// ============= GROUPS AND CAPTURING =============
const dateText = "Today is 2023-12-25";

// Capturing groups
const datePattern = /(\d{4})-(\d{2})-(\d{2})/;
const dateMatch = datePattern.exec(dateText);
console.log(dateMatch[0]); // "2023-12-25" - full match
console.log(dateMatch[1]); // "2023" - year
console.log(dateMatch[2]); // "12" - month
console.log(dateMatch[3]); // "25" - day

// Named capturing groups
const namedDatePattern = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/;
const namedMatch = namedDatePattern.exec(dateText);
console.log(namedMatch.groups.year); // "2023"
console.log(namedMatch.groups.month); // "12"
console.log(namedMatch.groups.day); // "25"

// Non-capturing groups
const nonCapturingPattern = /(?:Mr\.|Mrs\.|Ms\.)\s+(\w+)/;
const nameMatch = nonCapturingPattern.exec("Mr. Smith");
console.log(nameMatch[1]); // "Smith" - only the name is captured

// ============= LOOKAHEAD AND LOOKBEHIND =============
// Positive lookahead (?=...)
const passwordText = "password123";
const hasDigitLookahead = /\w+(?=\d)/; // word characters followed by digit
console.log(hasDigitLookahead.test(passwordText)); // true

// Negative lookahead (?!...)
const noDigitLookahead = /\w+(?!\d)/; // word characters not followed by digit
console.log(noDigitLookahead.test("hello")); // true
console.log(noDigitLookahead.test("hello1")); // false

// Positive lookbehind (?<=...)
const currencyText = "$100 €50 £75";
const afterDollar = /(?<=\$)\d+/; // digits after dollar sign
console.log(currencyText.match(afterDollar)); // ['100']

// Negative lookbehind (?<!...)
const notAfterDollar = /(?<!\$)\d+/g; // digits not after dollar sign
console.log(currencyText.match(notAfterDollar)); // ['50', '75']

// ============= COMPLEX VALIDATION PATTERNS =============
class RegexValidator {
  static patterns = {
    email: /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/,
    phone: /^[\+]?[1-9][\d]{0,15}$/,
    url: /^https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{1,256}\.[a-zA-Z0-9()]{1,6}\b([-a-zA-Z0-9()@:%_\+.~#?&//=]*)$/,
    creditCard: /^(?:4[0-9]{12}(?:[0-9]{3})?|5[1-5][0-9]{14}|3[47][0-9]{13}|3[0-9]{13}|6(?:011|5[0-9]{2})[0-9]{12})$/,
    ipv4: /^(?:(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$/,
    ipv6: /^([0-9a-fA-F]{1,4}:){7}[0-9a-fA-F]{1,4}$/,
    hexColor: /^#([0-9a-fA-F]{3}|[0-9a-fA-F]{6})$/,
    strongPassword: /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$/,
    username: /^[a-zA-Z0-9_]{3,20}$/,
    slug: /^[a-z0-9]+(?:-[a-z0-9]+)*$/
  };
  
  static validate(type, value) {
    const pattern = this.patterns[type];
    if (!pattern) {
      throw new Error(`Unknown validation type: ${type}`);
    }
    return pattern.test(value);
  }
  
  static getMatches(type, text) {
    const pattern = new RegExp(this.patterns[type].source, 'g');
    return text.match(pattern) || [];
  }
  
  static extract(type, text) {
    const pattern = this.patterns[type];
    const match = pattern.exec(text);
    return match ? match[0] : null;
  }
  
  static replaceMatches(type, text, replacement) {
    const pattern = new RegExp(this.patterns[type].source, 'g');
    return text.replace(pattern, replacement);
  }
  
  // Custom validation with detailed results
  static validateDetailed(type, value) {
    const pattern = this.patterns[type];
    if (!pattern) {
      return { valid: false, error: `Unknown validation type: ${type}` };
    }
    
    const isValid = pattern.test(value);
    const result = { valid: isValid, value, type };
    
    if (isValid) {
      const match = pattern.exec(value);
      result.match = match;
      result.groups = match.groups || {};
    } else {
      result.error = `Invalid ${type} format`;
    }
    
    return result;
  }
}

// Usage examples
console.log(RegexValidator.validate('email', 'user@example.com')); // true
console.log(RegexValidator.validate('phone', '+1234567890')); // true
console.log(RegexValidator.validate('strongPassword', 'MyPass123!')); // true

const emailText = "Contact us at admin@company.com or support@help.org";
console.log(RegexValidator.getMatches('email', emailText)); 
// ['admin@company.com', 'support@help.org']

// ============= TEXT PROCESSING UTILITIES =============
class TextProcessor {
  // Extract and format phone numbers
  static formatPhoneNumbers(text) {
    const phonePattern = /(\+?1)?[-.\s]?\(?([0-9]{3})\)?[-.\s]?([0-9]{3})[-.\s]?([0-9]{4})/g;
    
    return text.replace(phonePattern, (match, country, area, exchange, number) => {
      const formatted = `(${area}) ${exchange}-${number}`;
      return country ? `+1 ${formatted}` : formatted;
    });
  }
  
  // Clean and normalize whitespace
  static normalizeWhitespace(text) {
    return text
      .replace(/\s+/g, ' ') // Multiple spaces to single space
      .replace(/^\s+|\s+$/g, '') // Trim start and end
      .replace(/\s+([,.!?;:])/g, '$1') // Remove space before punctuation
      .replace(/([.!?])\s*([A-Z])/g, '$1 $2'); // Ensure space after sentence end
  }
  
  // Extract URLs and convert to links
  static linkifyUrls(text) {
    const urlPattern = /(https?:\/\/[^\s]+)/g;
    return text.replace(urlPattern, '<a href="$1" target="_blank">$1</a>');
  }
  
  // Highlight search terms
  static highlightTerms(text, searchTerms, className = 'highlight') {
    const escapedTerms = searchTerms.map(term => 
      term.replace(/[.*+?^${}()|[\]\\]/g, '\\$&')
    );
    const pattern = new RegExp(`(${escapedTerms.join('|')})`, 'gi');
    
    return text.replace(pattern, `<span class="${className}">$1</span>`);
  }
  
  // Extract hashtags and mentions
  static extractSocialTokens(text) {
    const hashtags = text.match(/#\w+/g) || [];
    const mentions = text.match(/@\w+/g) || [];
    
    return { hashtags, mentions };
  }
  
  // Remove HTML tags
  static stripHtml(html) {
    return html.replace(/<[^>]*>/g, '');
  }
  
  // Convert markdown-like syntax to HTML
  static simpleMarkdownToHtml(text) {
    return text
      .replace(/\*\*(.*?)\*\*/g, '<strong>$1</strong>') // Bold
      .replace(/\*(.*?)\*/g, '<em>$1</em>') // Italic
      .replace(/`(.*?)`/g, '<code>$1</code>') // Code
      .replace(/\[(.*?)\]\((.*?)\)/g, '<a href="$2">$1</a>'); // Links
  }
  
  // Extract code blocks
  static extractCodeBlocks(text) {
    const codeBlockPattern = /```(\w+)?\n([\s\S]*?)```/g;
    const inlineCodePattern = /`([^`]+)`/g;
    
    const codeBlocks = [];
    const inlineCodes = [];
    
    let match;
    while ((match = codeBlockPattern.exec(text)) !== null) {
      codeBlocks.push({
        language: match[1] || 'text',
        code: match[2].trim(),
        fullMatch: match[0]
      });
    }
    
    while ((match = inlineCodePattern.exec(text)) !== null) {
      inlineCodes.push({
        code: match[1],
        fullMatch: match[0]
      });
    }
    
    return { codeBlocks, inlineCodes };
  }
}

// Usage examples
const phoneText = "Call me at 555-123-4567 or +1 (555) 987-6543";
console.log(TextProcessor.formatPhoneNumbers(phoneText));

const messyText = "Hello    world!   This  is    a test.";
console.log(TextProcessor.normalizeWhitespace(messyText));

const urlText = "Visit https://example.com for more info";
console.log(TextProcessor.linkifyUrls(urlText));

const searchText = "The quick brown fox jumps";
console.log(TextProcessor.highlightTerms(searchText, ['quick', 'fox']));

// ============= ADVANCED REGEX TECHNIQUES =============
class AdvancedRegex {
  // Recursive patterns for nested structures
  static matchNestedParentheses(text) {
    // This is a simplified version - true recursive regex is complex
    const pattern = /\([^()]*(?:\([^()]*\)[^()]*)*\)/g;
    return text.match(pattern) || [];
  }
  
  // Balance checker
  static isBalanced(text, open = '(', close = ')') {
    const escapedOpen = open.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
    const escapedClose = close.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
    
    let count = 0;
    const pattern = new RegExp(`[${escapedOpen}${escapedClose}]`, 'g');
    
    let match;
    while ((match = pattern.exec(text)) !== null) {
      if (match[0] === open) {
        count++;
      } else {
        count--;
        if (count < 0) return false;
      }
    }
    
    return count === 0;
  }
  
  // CSV parser with regex
  static parseCSV(csvText) {
    const lines = csvText.trim().split('\n');
    const result = [];
    
    // Pattern for CSV fields (handles quoted fields with commas)
    const csvPattern = /("([^"]|"")*"|[^",]*)(,|$)/g;
    
    for (const line of lines) {
      const fields = [];
      let match;
      
      while ((match = csvPattern.exec(line)) !== null) {
        let field = match[1];
        
        // Remove surrounding quotes and handle escaped quotes
        if (field.startsWith('"') && field.endsWith('"')) {
          field = field.slice(1, -1).replace(/""/g, '"');
        }
        
        fields.push(field);
        
        if (match[3] !== ',') break; // End of line
      }
      
      result.push(fields);
      csvPattern.lastIndex = 0; // Reset for next line
    }
    
    return result;
  }
  
  // Template literal parser
  static parseTemplate(template, variables) {
    const pattern = /\$\{([^}]+)\}/g;
    
    return template.replace(pattern, (match, expression) => {
      try {
        // Simple variable replacement (be careful with eval in production!)
        const value = new Function('vars', `with(vars) { return ${expression}; }`)(variables);
        return value !== undefined ? String(value) : match;
      } catch (error) {
        return match; // Return original if evaluation fails
      }
    });
  }
  
  // Log parser
  static parseLogEntry(logLine) {
    const pattern = /^\[(?<timestamp>[^\]]+)\]\s+(?<level>\w+)\s+(?<message>.*?)(?:\s+\((?<context>[^)]+)\))?$/;
    const match = pattern.exec(logLine);
    
    if (!match) return null;
    
    return {
      timestamp: new Date(match.groups.timestamp),
      level: match.groups.level,
      message: match.groups.message,
      context: match.groups.context || null
    };
  }
  
  // SQL query parser (simplified)
  static parseSimpleSQL(query) {
    const selectPattern = /^SELECT\s+(.+?)\s+FROM\s+(\w+)(?:\s+WHERE\s+(.+?))?(?:\s+ORDER\s+BY\s+(.+?))?(?:\s+LIMIT\s+(\d+))?$/i;
    const match = selectPattern.exec(query.trim());
    
    if (!match) return null;
    
    return {
      type: 'SELECT',
      fields: match[1].split(',').map(field => field.trim()),
      table: match[2],
      where: match[3] || null,
      orderBy: match[4] || null,
      limit: match[5] ? parseInt(match[5]) : null
    };
  }
}

// Usage examples
console.log(AdvancedRegex.matchNestedParentheses("Hello (world (nested) test) end"));
console.log(AdvancedRegex.isBalanced("((()))")); // true
console.log(AdvancedRegex.isBalanced("((())")); // false

const csvData = `name,age,city
"John Doe",30,"New York"
Jane,25,Boston`;
console.log(AdvancedRegex.parseCSV(csvData));

const template = "Hello ${name}, you are ${age} years old!";
const vars = { name: "John", age: 30 };
console.log(AdvancedRegex.parseTemplate(template, vars));

const logLine = "[2023-12-25 10:30:00] INFO User logged in (user_id: 123)";
console.log(AdvancedRegex.parseLogEntry(logLine));
```

## 🏗️ Next.js Applications

### **✅ Form Validation System**

```jsx
// ============= VALIDATION LIBRARY =============
// lib/validation.js
export class ValidationRule {
  constructor(pattern, message, options = {}) {
    this.pattern = pattern instanceof RegExp ? pattern : new RegExp(pattern);
    this.message = message;
    this.required = options.required || false;
    this.transform = options.transform || (value => value);
  }
  
  validate(value) {
    // Transform value first
    const transformedValue = this.transform(value);
    
    // Check if empty and required
    if (!transformedValue && this.required) {
      return { valid: false, message: this.message };
    }
    
    // Skip validation for empty optional fields
    if (!transformedValue && !this.required) {
      return { valid: true, value: transformedValue };
    }
    
    // Test pattern
    const isValid = this.pattern.test(transformedValue);
    
    return {
      valid: isValid,
      value: transformedValue,
      message: isValid ? null : this.message
    };
  }
}

export class ValidationSchema {
  constructor() {
    this.rules = new Map();
  }
  
  addRule(fieldName, rule) {
    if (!this.rules.has(fieldName)) {
      this.rules.set(fieldName, []);
    }
    this.rules.get(fieldName).push(rule);
    return this;
  }
  
  field(fieldName) {
    return new FieldBuilder(this, fieldName);
  }
  
  validate(data) {
    const results = {};
    const errors = {};
    let isValid = true;
    
    for (const [fieldName, rules] of this.rules) {
      const value = data[fieldName];
      let transformedValue = value;
      
      for (const rule of rules) {
        const result = rule.validate(transformedValue);
        
        if (!result.valid) {
          errors[fieldName] = result.message;
          isValid = false;
          break;
        }
        
        transformedValue = result.value;
      }
      
      if (!errors[fieldName]) {
        results[fieldName] = transformedValue;
      }
    }
    
    return {
      valid: isValid,
      data: results,
      errors
    };
  }
}

class FieldBuilder {
  constructor(schema, fieldName) {
    this.schema = schema;
    this.fieldName = fieldName;
  }
  
  required(message = 'This field is required') {
    this.schema.addRule(this.fieldName, new ValidationRule(
      /.+/,
      message,
      { required: true }
    ));
    return this;
  }
  
  email(message = 'Invalid email format') {
    this.schema.addRule(this.fieldName, new ValidationRule(
      /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/,
      message
    ));
    return this;
  }
  
  phone(message = 'Invalid phone number') {
    this.schema.addRule(this.fieldName, new ValidationRule(
      /^[\+]?[1-9][\d]{0,15}$/,
      message,
      { transform: value => value?.replace(/\D/g, '') }
    ));
    return this;
  }
  
  password(message = 'Password must be at least 8 characters with uppercase, lowercase, number and special character') {
    this.schema.addRule(this.fieldName, new ValidationRule(
      /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$/,
      message
    ));
    return this;
  }
  
  username(message = 'Username must be 3-20 characters, letters, numbers and underscores only') {
    this.schema.addRule(this.fieldName, new ValidationRule(
      /^[a-zA-Z0-9_]{3,20}$/,
      message,
      { transform: value => value?.toLowerCase() }
    ));
    return this;
  }
  
  url(message = 'Invalid URL format') {
    this.schema.addRule(this.fieldName, new ValidationRule(
      /^https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{1,256}\.[a-zA-Z0-9()]{1,6}\b([-a-zA-Z0-9()@:%_\+.~#?&//=]*)$/,
      message
    ));
    return this;
  }
  
  pattern(regex, message = 'Invalid format') {
    this.schema.addRule(this.fieldName, new ValidationRule(regex, message));
    return this;
  }
  
  transform(transformFn) {
    this.schema.addRule(this.fieldName, new ValidationRule(
      /.*/,
      '',
      { transform: transformFn }
    ));
    return this;
  }
  
  length(min, max, message) {
    const pattern = max ? new RegExp(`^.{${min},${max}}$`) : new RegExp(`^.{${min},}$`);
    const defaultMessage = max 
      ? `Must be between ${min} and ${max} characters`
      : `Must be at least ${min} characters`;
    
    this.schema.addRule(this.fieldName, new ValidationRule(
      pattern,
      message || defaultMessage
    ));
    return this;
  }
}

// Create common schemas
export const createUserSchema = () => new ValidationSchema()
  .field('username').required().username()
  .field('email').required().email()
  .field('password').required().password()
  .field('confirmPassword').required().pattern(
    /.*/, 
    'Passwords do not match'
  );

export const createContactSchema = () => new ValidationSchema()
  .field('name').required().length(2, 50)
  .field('email').required().email()
  .field('phone').phone()
  .field('message').required().length(10, 1000);

// ============= VALIDATION HOOKS =============
// hooks/useValidation.js
import { useState, useCallback } from 'react';

export const useValidation = (schema) => {
  const [errors, setErrors] = useState({});
  const [isValid, setIsValid] = useState(true);
  const [touched, setTouched] = useState({});
  
  const validate = useCallback((data) => {
    const result = schema.validate(data);
    setErrors(result.errors);
    setIsValid(result.valid);
    return result;
  }, [schema]);
  
  const validateField = useCallback((fieldName, value, allData = {}) => {
    const fieldData = { ...allData, [fieldName]: value };
    const result = schema.validate(fieldData);
    
    setErrors(prev => ({
      ...prev,
      [fieldName]: result.errors[fieldName] || null
    }));
    
    return !result.errors[fieldName];
  }, [schema]);
  
  const touch = useCallback((fieldName) => {
    setTouched(prev => ({ ...prev, [fieldName]: true }));
  }, []);
  
  const reset = useCallback(() => {
    setErrors({});
    setIsValid(true);
    setTouched({});
  }, []);
  
  const getFieldError = useCallback((fieldName) => {
    return touched[fieldName] ? errors[fieldName] : null;
  }, [errors, touched]);
  
  return {
    errors,
    isValid,
    touched,
    validate,
    validateField,
    touch,
    reset,
    getFieldError
  };
};

// ============= FORM COMPONENTS =============
// components/ValidatedInput.jsx
import { useState } from 'react';

const ValidatedInput = ({
  name,
  label,
  type = 'text',
  value,
  onChange,
  onBlur,
  error,
  placeholder,
  className = '',
  ...props
}) => {
  const [isFocused, setIsFocused] = useState(false);
  
  const handleFocus = () => setIsFocused(true);
  const handleBlur = (e) => {
    setIsFocused(false);
    onBlur?.(e);
  };
  
  return (
    <div className={`form-field ${className}`}>
      {label && (
        <label htmlFor={name} className="form-label">
          {label}
        </label>
      )}
      
      <input
        id={name}
        name={name}
        type={type}
        value={value || ''}
        onChange={onChange}
        onFocus={handleFocus}
        onBlur={handleBlur}
        placeholder={placeholder}
        className={`form-input ${error ? 'error' : ''} ${isFocused ? 'focused' : ''}`}
        {...props}
      />
      
      {error && (
        <div className="form-error" role="alert">
          {error}
        </div>
      )}
    </div>
  );
};

export default ValidatedInput;

// ============= COMPLETE FORM EXAMPLE =============
// components/ContactForm.jsx
import { useState } from 'react';
import { useValidation } from '../hooks/useValidation';
import { createContactSchema } from '../lib/validation';
import ValidatedInput from './ValidatedInput';

const ContactForm = ({ onSubmit }) => {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    phone: '',
    message: ''
  });
  
  const schema = createContactSchema();
  const { validate, validateField, touch, getFieldError, isValid, reset } = useValidation(schema);
  
  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
    
    // Validate field if it has been touched
    validateField(name, value, formData);
  };
  
  const handleBlur = (e) => {
    const { name } = e.target;
    touch(name);
  };
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    
    // Touch all fields
    Object.keys(formData).forEach(touch);
    
    // Validate form
    const result = validate(formData);
    
    if (result.valid) {
      try {
        await onSubmit(result.data);
        setFormData({ name: '', email: '', phone: '', message: '' });
        reset();
      } catch (error) {
        console.error('Submission error:', error);
      }
    }
  };
  
  return (
    <form onSubmit={handleSubmit} className="contact-form">
      <ValidatedInput
        name="name"
        label="Full Name"
        value={formData.name}
        onChange={handleChange}
        onBlur={handleBlur}
        error={getFieldError('name')}
        placeholder="Enter your full name"
        required
      />
      
      <ValidatedInput
        name="email"
        label="Email Address"
        type="email"
        value={formData.email}
        onChange={handleChange}
        onBlur={handleBlur}
        error={getFieldError('email')}
        placeholder="Enter your email"
        required
      />
      
      <ValidatedInput
        name="phone"
        label="Phone Number"
        type="tel"
        value={formData.phone}
        onChange={handleChange}
        onBlur={handleBlur}
        error={getFieldError('phone')}
        placeholder="Enter your phone number"
      />
      
      <div className="form-field">
        <label htmlFor="message" className="form-label">
          Message
        </label>
        <textarea
          id="message"
          name="message"
          value={formData.message}
          onChange={handleChange}
          onBlur={handleBlur}
          placeholder="Enter your message"
          className={`form-textarea ${getFieldError('message') ? 'error' : ''}`}
          rows={5}
          required
        />
        {getFieldError('message') && (
          <div className="form-error" role="alert">
            {getFieldError('message')}
          </div>
        )}
      </div>
      
      <button
        type="submit"
        className={`form-submit ${!isValid ? 'disabled' : ''}`}
        disabled={!isValid}
      >
        Send Message
      </button>
    </form>
  );
};

export default ContactForm;

// ============= REAL-TIME SEARCH COMPONENT =============
// components/SearchBox.jsx
import { useState, useEffect, useMemo } from 'react';

const SearchBox = ({ 
  data, 
  searchFields, 
  onResults,
  placeholder = "Search...",
  highlightMatches = true 
}) => {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  
  // Create search regex from query
  const searchRegex = useMemo(() => {
    if (!query.trim()) return null;
    
    // Escape special regex characters and split by spaces
    const terms = query.trim().split(/\s+/).map(term =>
      term.replace(/[.*+?^${}()|[\]\\]/g, '\\$&')
    );
    
    // Create pattern that matches any term
    return new RegExp(`(${terms.join('|')})`, 'gi');
  }, [query]);
  
  // Perform search
  useEffect(() => {
    if (!searchRegex) {
      setResults([]);
      onResults?.([]);
      return;
    }
    
    const filtered = data.filter(item => {
      return searchFields.some(field => {
        const value = String(item[field] || '');
        return searchRegex.test(value);
      });
    });
    
    setResults(filtered);
    onResults?.(filtered);
  }, [data, searchFields, searchRegex, onResults]);
  
  // Highlight matching text
  const highlightText = (text, regex) => {
    if (!highlightMatches || !regex) return text;
    
    return text.replace(regex, '<mark>$1</mark>');
  };
  
  return (
    <div className="search-box">
      <input
        type="text"
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder={placeholder}
        className="search-input"
      />
      
      {query && (
        <div className="search-results">
          <div className="search-stats">
            {results.length} result{results.length !== 1 ? 's' : ''} found
          </div>
          
          {results.map((item, index) => (
            <div key={index} className="search-result">
              {searchFields.map(field => (
                <div key={field} className="search-field">
                  <strong>{field}:</strong>{' '}
                  <span
                    dangerouslySetInnerHTML={{
                      __html: highlightText(String(item[field] || ''), searchRegex)
                    }}
                  />
                </div>
              ))}
            </div>
          ))}
        </div>
      )}
    </div>
  );
};

export default SearchBox;
```

### **🔍 Content Processing System**

```jsx
// ============= CONTENT PARSER =============
// lib/content-parser.js
export class ContentParser {
  static processors = new Map();
  
  // Register content processor
  static registerProcessor(name, pattern, processor) {
    this.processors.set(name, { pattern, processor });
  }
  
  // Process content with all registered processors
  static processContent(content) {
    let processed = content;
    
    for (const [name, { pattern, processor }] of this.processors) {
      processed = processed.replace(pattern, processor);
    }
    
    return processed;
  }
  
  // Extract metadata from content
  static extractMetadata(content) {
    const metadata = {};
    
    // Extract title
    const titleMatch = /^#\s+(.+)$/m.exec(content);
    if (titleMatch) {
      metadata.title = titleMatch[1];
    }
    
    // Extract description (first paragraph)
    const descMatch = /^(?!#)(.+)$/m.exec(content);
    if (descMatch) {
      metadata.description = descMatch[1].substring(0, 160);
    }
    
    // Extract tags
    const tagMatches = content.match(/#\w+/g);
    if (tagMatches) {
      metadata.tags = tagMatches.map(tag => tag.substring(1));
    }
    
    // Extract reading time estimate
    const wordCount = content.split(/\s+/).length;
    metadata.readingTime = Math.ceil(wordCount / 200); // 200 words per minute
    
    return metadata;
  }
  
  // Extract table of contents
  static generateTOC(content) {
    const headings = content.match(/^(#{1,6})\s+(.+)$/gm) || [];
    
    return headings.map(heading => {
      const match = /^(#{1,6})\s+(.+)$/.exec(heading);
      const level = match[1].length;
      const text = match[2];
      const slug = text.toLowerCase().replace(/[^a-z0-9]+/g, '-').replace(/^-|-$/g, '');
      
      return { level, text, slug };
    });
  }
}

// Register default processors
ContentParser.registerProcessor('links', 
  /\[([^\]]+)\]\(([^)]+)\)/g,
  (match, text, url) => `<a href="${url}" target="_blank">${text}</a>`
);

ContentParser.registerProcessor('bold',
  /\*\*(.*?)\*\*/g,
  (match, text) => `<strong>${text}</strong>`
);

ContentParser.registerProcessor('italic',
  /\*(.*?)\*/g,
  (match, text) => `<em>${text}</em>`
);

ContentParser.registerProcessor('code',
  /`([^`]+)`/g,
  (match, code) => `<code>${code}</code>`
);

ContentParser.registerProcessor('codeBlocks',
  /```(\w+)?\n([\s\S]*?)```/g,
  (match, language, code) => {
    const lang = language || 'text';
    return `<pre><code class="language-${lang}">${code.trim()}</code></pre>`;
  }
);

// ============= LOG ANALYZER =============
// lib/log-analyzer.js
export class LogAnalyzer {
  constructor() {
    this.patterns = {
      timestamp: /\[(\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2})\]/,
      level: /\[(DEBUG|INFO|WARN|ERROR|FATAL)\]/,
      ip: /(\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3})/,
      httpMethod: /(GET|POST|PUT|DELETE|PATCH|HEAD|OPTIONS)/,
      httpStatus: /\s(1\d{2}|2\d{2}|3\d{2}|4\d{2}|5\d{2})\s/,
      url: /("(?:GET|POST|PUT|DELETE|PATCH|HEAD|OPTIONS)\s([^"]+))/,
      userAgent: /("Mozilla[^"]*"|"[^"]*Chrome[^"]*"|"[^"]*Safari[^"]*")/,
      responseTime: /(\d+)ms/
    };
  }
  
  parseLine(line) {
    const entry = {
      raw: line,
      timestamp: null,
      level: null,
      ip: null,
      method: null,
      url: null,
      status: null,
      responseTime: null,
      userAgent: null
    };
    
    // Extract each component
    for (const [key, pattern] of Object.entries(this.patterns)) {
      const match = pattern.exec(line);
      if (match) {
        switch (key) {
          case 'timestamp':
            entry.timestamp = new Date(match[1]);
            break;
          case 'url':
            entry.url = match[2];
            break;
          case 'httpStatus':
            entry.status = parseInt(match[1]);
            break;
          case 'responseTime':
            entry.responseTime = parseInt(match[1]);
            break;
          default:
            entry[key] = match[1];
        }
      }
    }
    
    return entry;
  }
  
  parseMultiple(logs) {
    return logs.split('\n')
      .filter(line => line.trim())
      .map(line => this.parseLine(line));
  }
  
  analyze(entries) {
    const stats = {
      total: entries.length,
      levels: {},
      statusCodes: {},
      methods: {},
      avgResponseTime: 0,
      errorRate: 0,
      topUrls: {},
      timeRange: null
    };
    
    let totalResponseTime = 0;
    let responseTimeCount = 0;
    let errorCount = 0;
    
    entries.forEach(entry => {
      // Count levels
      if (entry.level) {
        stats.levels[entry.level] = (stats.levels[entry.level] || 0) + 1;
        if (['ERROR', 'FATAL'].includes(entry.level)) {
          errorCount++;
        }
      }
      
      // Count status codes
      if (entry.status) {
        stats.statusCodes[entry.status] = (stats.statusCodes[entry.status] || 0) + 1;
        if (entry.status >= 400) {
          errorCount++;
        }
      }
      
      // Count methods
      if (entry.method) {
        stats.methods[entry.method] = (stats.methods[entry.method] || 0) + 1;
      }
      
      // Response time
      if (entry.responseTime) {
        totalResponseTime += entry.responseTime;
        responseTimeCount++;
      }
      
      // URLs
      if (entry.url) {
        stats.topUrls[entry.url] = (stats.topUrls[entry.url] || 0) + 1;
      }
    });
    
    // Calculate averages
    stats.avgResponseTime = responseTimeCount > 0 ? totalResponseTime / responseTimeCount : 0;
    stats.errorRate = entries.length > 0 ? (errorCount / entries.length) * 100 : 0;
    
    // Time range
    const timestamps = entries.map(e => e.timestamp).filter(Boolean);
    if (timestamps.length > 0) {
      stats.timeRange = {
        start: new Date(Math.min(...timestamps)),
        end: new Date(Math.max(...timestamps))
      };
    }
    
    // Sort top URLs
    stats.topUrls = Object.entries(stats.topUrls)
      .sort(([, a], [, b]) => b - a)
      .slice(0, 10)
      .reduce((obj, [url, count]) => ({ ...obj, [url]: count }), {});
    
    return stats;
  }
}

// ============= DATA EXTRACTOR COMPONENT =============
// components/DataExtractor.jsx
import { useState } from 'react';

const DataExtractor = () => {
  const [input, setInput] = useState('');
  const [extractedData, setExtractedData] = useState({});
  const [extractionType, setExtractionType] = useState('emails');
  
  const extractors = {
    emails: {
      pattern: /[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}/g,
      label: 'Email Addresses'
    },
    phones: {
      pattern: /(\+?1)?[-.\s]?\(?([0-9]{3})\)?[-.\s]?([0-9]{3})[-.\s]?([0-9]{4})/g,
      label: 'Phone Numbers'
    },
    urls: {
      pattern: /https?:\/\/[^\s]+/g,
      label: 'URLs'
    },
    ips: {
      pattern: /\b(?:(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\b/g,
      label: 'IP Addresses'
    },
    dates: {
      pattern: /\b\d{4}-\d{2}-\d{2}\b|\b\d{2}\/\d{2}\/\d{4}\b|\b\d{2}-\d{2}-\d{4}\b/g,
      label: 'Dates'
    },
    hashtags: {
      pattern: /#\w+/g,
      label: 'Hashtags'
    },
    mentions: {
      pattern: /@\w+/g,
      label: 'Mentions'
    }
  };
  
  const extractData = () => {
    const results = {};
    
    if (extractionType === 'all') {
      // Extract all types
      for (const [type, { pattern, label }] of Object.entries(extractors)) {
        const matches = input.match(pattern) || [];
        results[type] = {
          label,
          matches: [...new Set(matches)], // Remove duplicates
          count: matches.length
        };
      }
    } else {
      // Extract specific type
      const extractor = extractors[extractionType];
      const matches = input.match(extractor.pattern) || [];
      results[extractionType] = {
        label: extractor.label,
        matches: [...new Set(matches)],
        count: matches.length
      };
    }
    
    setExtractedData(results);
  };
  
  const copyToClipboard = (text) => {
    navigator.clipboard.writeText(text);
  };
  
  return (
    <div className="data-extractor">
      <h2>Data Extractor</h2>
      
      <div className="controls">
        <select
          value={extractionType}
          onChange={(e) => setExtractionType(e.target.value)}
          className="extraction-type"
        >
          <option value="all">Extract All Types</option>
          {Object.entries(extractors).map(([type, { label }]) => (
            <option key={type} value={type}>{label}</option>
          ))}
        </select>
        
        <button onClick={extractData} className="extract-btn">
          Extract Data
        </button>
      </div>
      
      <textarea
        value={input}
        onChange={(e) => setInput(e.target.value)}
        placeholder="Paste your text here to extract data..."
        className="input-text"
        rows={10}
      />
      
      {Object.keys(extractedData).length > 0 && (
        <div className="results">
          <h3>Extraction Results</h3>
          
          {Object.entries(extractedData).map(([type, data]) => (
            <div key={type} className="result-section">
              <h4>{data.label} ({data.count} found)</h4>
              
              {data.matches.length > 0 ? (
                <div className="matches">
                  {data.matches.map((match, index) => (
                    <div key={index} className="match-item">
                      <span>{match}</span>
                      <button
                        onClick={() => copyToClipboard(match)}
                        className="copy-btn"
                        title="Copy to clipboard"
                      >
                        📋
                      </button>
                    </div>
                  ))}
                  
                  <button
                    onClick={() => copyToClipboard(data.matches.join('\n'))}
                    className="copy-all-btn"
                  >
                    Copy All {data.label}
                  </button>
                </div>
              ) : (
                <div className="no-matches">No {data.label.toLowerCase()} found</div>
              )}
            </div>
          ))}
        </div>
      )}
      
      <style jsx>{`
        .data-extractor {
          max-width: 800px;
          margin: 0 auto;
          padding: 20px;
        }
        
        .controls {
          display: flex;
          gap: 10px;
          margin-bottom: 20px;
        }
        
        .extraction-type {
          padding: 8px;
          border: 1px solid #ddd;
          border-radius: 4px;
        }
        
        .extract-btn {
          padding: 8px 16px;
          background: #007bff;
          color: white;
          border: none;
          border-radius: 4px;
          cursor: pointer;
        }
        
        .input-text {
          width: 100%;
          padding: 12px;
          border: 1px solid #ddd;
          border-radius: 4px;
          font-family: monospace;
          margin-bottom: 20px;
        }
        
        .results {
          border: 1px solid #ddd;
          border-radius: 4px;
          padding: 20px;
        }
        
        .result-section {
          margin-bottom: 20px;
        }
        
        .result-section h4 {
          margin: 0 0 10px 0;
          color: #333;
        }
        
        .matches {
          background: #f8f9fa;
          padding: 10px;
          border-radius: 4px;
        }
        
        .match-item {
          display: flex;
          justify-content: space-between;
          align-items: center;
          padding: 5px 0;
          border-bottom: 1px solid #eee;
        }
        
        .copy-btn {
          background: none;
          border: none;
          cursor: pointer;
          font-size: 14px;
        }
        
        .copy-all-btn {
          margin-top: 10px;
          padding: 6px 12px;
          background: #28a745;
          color: white;
          border: none;
          border-radius: 4px;
          cursor: pointer;
        }
        
        .no-matches {
          color: #666;
          font-style: italic;
        }
      `}</style>
    </div>
  );
};

export default DataExtractor;
```

## 🎯 แบบฝึกหัด

### **🔍 แบบฝึกหัดที่ 1: Advanced Regex Patterns**
สร้าง complex regex patterns:

```javascript
// TODO: สร้าง advanced regex patterns

// 1. Credit card validator
class CreditCardValidator {
  static patterns = {
    visa: /^4[0-9]{12}(?:[0-9]{3})?$/,
    mastercard: /^5[1-5][0-9]{14}$/,
    amex: /^3[47][0-9]{13}$/,
    discover: /^6(?:011|5[0-9]{2})[0-9]{12}$/
  };
  
  static validate(number) {
    // Remove spaces and dashes
    // Detect card type
    // Validate with Luhn algorithm
  }
}

// 2. SQL injection detector
class SQLInjectionDetector {
  static patterns = [
    // Add patterns for common SQL injection attempts
  ];
  
  static isSuspicious(input) {
    // Check input against patterns
  }
}

// 3. Password strength analyzer
class PasswordAnalyzer {
  static analyze(password) {
    // Analyze password strength with detailed feedback
    // Check for common patterns, dictionary words, etc.
  }
}
```

### **✅ แบบฝึกหัดที่ 2: Form Validation System**
สร้าง comprehensive validation:

```jsx
// TODO: สร้าง advanced form validation

// 1. Dynamic form validator
const DynamicFormValidator = ({ schema, children }) => {
  // Create dynamic form based on schema
  // Support conditional validation
  // Real-time validation feedback
};

// 2. Multi-step form with validation
const MultiStepForm = ({ steps }) => {
  // Support validation per step
  // Progress indicator
  // Data persistence between steps
};

// 3. File upload validator
const FileUploadValidator = ({ 
  acceptedTypes, 
  maxSize, 
  onValidation 
}) => {
  // Validate file types with regex
  // Check file size
  // Validate file names
};
```

### **🔍 แบบฝึกหัดที่ 3: Content Processing**
สร้าง content processing system:

```jsx
// TODO: สร้าง content processing system

// 1. Markdown processor
const MarkdownProcessor = ({ content }) => {
  // Parse markdown with regex
  // Support custom extensions
  // Generate table of contents
};

// 2. Code syntax highlighter
const SyntaxHighlighter = ({ code, language }) => {
  // Highlight code with regex patterns
  // Support multiple languages
  // Line numbers and themes
};

// 3. Data migration tool
const DataMigrator = () => {
  // Extract data with regex
  // Transform formats
  // Validate transformed data
};
```

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 12: Symbols & Maps/Sets](./12-symbols-maps-sets.md)

**บทถัดไป**: [บทที่ 14: Error Handling & Debugging](./14-error-handling.md)

**กลับไปหน้าหลัก**: [README](../README.md)

---

## 📚 เอกสารอ้างอิง

- [MDN: Regular Expressions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions)
- [RegExp Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp)
- [String.match()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/match)

---

## 🎉 สรุป

ในบทนี้เราได้เรียนรู้:

- **Regex Fundamentals** - syntax, flags, และ character classes
- **Advanced Patterns** - groups, lookahead/lookbehind, quantifiers
- **Validation Systems** - complex validation with detailed feedback
- **Content Processing** - parsing และ data extraction
- **Next.js Integration** - form validation และ content management

Regular Expressions เป็นเครื่องมือที่ทรงพลังสำหรับ pattern matching และ text processing ใน JavaScript! 🚀
