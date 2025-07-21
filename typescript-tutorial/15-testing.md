# บทที่ 15: Testing TypeScript

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้การเขียน Unit Tests สำหรับ TypeScript
- เข้าใจการใช้งาน Jest กับ TypeScript
- เรียนรู้ Mocking และ Test Doubles
- เข้าใจ Integration Testing และ E2E Testing

## 🧪 การติดตั้งและตั้งค่า Testing Environment

### 1. **Setup Jest with TypeScript**

```bash
# ติดตั้ง dependencies
npm install --save-dev jest typescript ts-jest @types/jest

# สร้าง jest config
npm init @types/jest
```

```json
// jest.config.js
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  roots: ['<rootDir>/src'],
  testMatch: [
    '**/__tests__/**/*.ts',
    '**/?(*.)+(spec|test).ts'
  ],
  transform: {
    '^.+\\.ts$': 'ts-jest',
  },
  collectCoverageFrom: [
    'src/**/*.ts',
    '!src/**/*.d.ts',
    '!src/**/*.test.ts',
    '!src/**/*.spec.ts'
  ],
  coverageDirectory: 'coverage',
  coverageReporters: ['text', 'lcov', 'html'],
  setupFilesAfterEnv: ['<rootDir>/src/test-setup.ts']
};
```

```typescript
// src/test-setup.ts
// Global test setup
beforeEach(() => {
  // Reset any global state
  jest.clearAllMocks();
});

// Custom matchers
expect.extend({
  toBeValidEmail(received: string) {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    const pass = emailRegex.test(received);
    
    if (pass) {
      return {
        message: () => `expected ${received} not to be a valid email`,
        pass: true,
      };
    } else {
      return {
        message: () => `expected ${received} to be a valid email`,
        pass: false,
      };
    }
  },
});

declare global {
  namespace jest {
    interface Matchers<R> {
      toBeValidEmail(): R;
    }
  }
}
```

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020"],
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "declaration": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "types": ["jest", "node"]
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### 2. **Package.json Scripts**

```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "test:debug": "node --inspect-brk node_modules/.bin/jest --runInBand",
    "test:ci": "jest --coverage --watchAll=false --passWithNoTests"
  }
}
```

## 🔧 Unit Testing พื้นฐาน

### 1. **Testing Functions และ Classes**

```typescript
// src/utils/calculator.ts
export class Calculator {
  add(a: number, b: number): number {
    return a + b;
  }
  
  subtract(a: number, b: number): number {
    return a - b;
  }
  
  multiply(a: number, b: number): number {
    return a * b;
  }
  
  divide(a: number, b: number): number {
    if (b === 0) {
      throw new Error('Division by zero is not allowed');
    }
    return a / b;
  }
  
  power(base: number, exponent: number): number {
    return Math.pow(base, exponent);
  }
}

export function formatNumber(num: number, decimals: number = 2): string {
  return num.toFixed(decimals);
}

export function isEven(num: number): boolean {
  return num % 2 === 0;
}

// src/utils/calculator.test.ts
import { Calculator, formatNumber, isEven } from './calculator';

describe('Calculator', () => {
  let calculator: Calculator;
  
  beforeEach(() => {
    calculator = new Calculator();
  });
  
  describe('add', () => {
    it('should add two positive numbers', () => {
      const result = calculator.add(2, 3);
      expect(result).toBe(5);
    });
    
    it('should add positive and negative numbers', () => {
      const result = calculator.add(5, -2);
      expect(result).toBe(3);
    });
    
    it('should add two negative numbers', () => {
      const result = calculator.add(-3, -4);
      expect(result).toBe(-7);
    });
    
    it('should handle zero', () => {
      expect(calculator.add(5, 0)).toBe(5);
      expect(calculator.add(0, 0)).toBe(0);
    });
    
    it('should handle decimal numbers', () => {
      const result = calculator.add(0.1, 0.2);
      expect(result).toBeCloseTo(0.3);
    });
  });
  
  describe('subtract', () => {
    it('should subtract two numbers', () => {
      expect(calculator.subtract(5, 3)).toBe(2);
      expect(calculator.subtract(3, 5)).toBe(-2);
    });
  });
  
  describe('multiply', () => {
    it('should multiply two numbers', () => {
      expect(calculator.multiply(3, 4)).toBe(12);
      expect(calculator.multiply(-2, 3)).toBe(-6);
      expect(calculator.multiply(0, 100)).toBe(0);
    });
  });
  
  describe('divide', () => {
    it('should divide two numbers', () => {
      expect(calculator.divide(10, 2)).toBe(5);
      expect(calculator.divide(7, 2)).toBe(3.5);
    });
    
    it('should throw error when dividing by zero', () => {
      expect(() => calculator.divide(5, 0)).toThrow('Division by zero is not allowed');
    });
  });
  
  describe('power', () => {
    it('should calculate power correctly', () => {
      expect(calculator.power(2, 3)).toBe(8);
      expect(calculator.power(5, 0)).toBe(1);
      expect(calculator.power(3, 2)).toBe(9);
    });
  });
});

describe('formatNumber', () => {
  it('should format number with default 2 decimals', () => {
    expect(formatNumber(3.14159)).toBe('3.14');
  });
  
  it('should format number with specified decimals', () => {
    expect(formatNumber(3.14159, 3)).toBe('3.142');
    expect(formatNumber(10, 0)).toBe('10');
  });
  
  it('should handle integers', () => {
    expect(formatNumber(42)).toBe('42.00');
  });
});

describe('isEven', () => {
  it('should return true for even numbers', () => {
    expect(isEven(2)).toBe(true);
    expect(isEven(0)).toBe(true);
    expect(isEven(-4)).toBe(true);
  });
  
  it('should return false for odd numbers', () => {
    expect(isEven(1)).toBe(false);
    expect(isEven(3)).toBe(false);
    expect(isEven(-3)).toBe(false);
  });
});
```

### 2. **Testing với Parameterized Tests**

```typescript
// src/utils/validation.ts
export interface ValidationRule<T> {
  validate(value: T): boolean;
  message: string;
}

export class EmailValidator implements ValidationRule<string> {
  message = 'Invalid email format';
  
  validate(email: string): boolean {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(email);
  }
}

export class PasswordValidator implements ValidationRule<string> {
  message = 'Password must be at least 8 characters long and contain uppercase, lowercase, and number';
  
  validate(password: string): boolean {
    const hasUpperCase = /[A-Z]/.test(password);
    const hasLowerCase = /[a-z]/.test(password);
    const hasNumbers = /\d/.test(password);
    const isLongEnough = password.length >= 8;
    
    return hasUpperCase && hasLowerCase && hasNumbers && isLongEnough;
  }
}

export class RangeValidator implements ValidationRule<number> {
  message: string;
  
  constructor(private min: number, private max: number) {
    this.message = `Value must be between ${min} and ${max}`;
  }
  
  validate(value: number): boolean {
    return value >= this.min && value <= this.max;
  }
}

// src/utils/validation.test.ts
import { EmailValidator, PasswordValidator, RangeValidator } from './validation';

describe('EmailValidator', () => {
  let validator: EmailValidator;
  
  beforeEach(() => {
    validator = new EmailValidator();
  });
  
  // Parameterized tests
  describe.each([
    ['valid@example.com', true],
    ['user@domain.co.uk', true],
    ['test.email@sub.domain.com', true],
    ['invalid-email', false],
    ['@domain.com', false],
    ['user@', false],
    ['', false],
    ['user name@domain.com', false], // space not allowed
  ])('email validation', (email, expected) => {
    it(`should return ${expected} for "${email}"`, () => {
      expect(validator.validate(email)).toBe(expected);
    });
  });
});

describe('PasswordValidator', () => {
  let validator: PasswordValidator;
  
  beforeEach(() => {
    validator = new PasswordValidator();
  });
  
  test.each([
    ['Password123', true, 'valid password with all requirements'],
    ['MySecret99', true, 'valid password'],
    ['password123', false, 'missing uppercase'],
    ['PASSWORD123', false, 'missing lowercase'],
    ['Password', false, 'missing number'],
    ['Pass1', false, 'too short'],
    ['', false, 'empty password'],
  ])('%s - %s', (password, expected, description) => {
    expect(validator.validate(password)).toBe(expected);
  });
});

describe('RangeValidator', () => {
  describe('age validator (18-65)', () => {
    let validator: RangeValidator;
    
    beforeEach(() => {
      validator = new RangeValidator(18, 65);
    });
    
    test.each([
      [18, true],
      [25, true],
      [65, true],
      [17, false],
      [66, false],
      [0, false],
      [-5, false],
    ])('age %i should be %s', (age, expected) => {
      expect(validator.validate(age)).toBe(expected);
    });
  });
});
```

## 🎭 Mocking และ Test Doubles

### 1. **Mocking Dependencies**

```typescript
// src/services/user-service.ts
export interface User {
  id: string;
  name: string;
  email: string;
  createdAt: Date;
}

export interface UserRepository {
  findById(id: string): Promise<User | null>;
  save(user: Omit<User, 'id' | 'createdAt'>): Promise<User>;
  delete(id: string): Promise<boolean>;
  findByEmail(email: string): Promise<User | null>;
}

export interface EmailService {
  sendWelcomeEmail(user: User): Promise<void>;
  sendPasswordResetEmail(email: string, resetToken: string): Promise<void>;
}

export interface Logger {
  info(message: string, meta?: object): void;
  error(message: string, error?: Error): void;
  warn(message: string, meta?: object): void;
}

export class UserService {
  constructor(
    private userRepository: UserRepository,
    private emailService: EmailService,
    private logger: Logger
  ) {}
  
  async createUser(userData: { name: string; email: string }): Promise<User> {
    try {
      // Check if user already exists
      const existingUser = await this.userRepository.findByEmail(userData.email);
      if (existingUser) {
        throw new Error('User with this email already exists');
      }
      
      // Create new user
      const newUser = await this.userRepository.save(userData);
      this.logger.info('User created successfully', { userId: newUser.id });
      
      // Send welcome email
      try {
        await this.emailService.sendWelcomeEmail(newUser);
        this.logger.info('Welcome email sent', { userId: newUser.id });
      } catch (error) {
        this.logger.warn('Failed to send welcome email', { 
          userId: newUser.id, 
          error: error instanceof Error ? error.message : 'Unknown error' 
        });
      }
      
      return newUser;
    } catch (error) {
      this.logger.error('Failed to create user', error instanceof Error ? error : new Error('Unknown error'));
      throw error;
    }
  }
  
  async getUserById(id: string): Promise<User> {
    const user = await this.userRepository.findById(id);
    if (!user) {
      throw new Error(`User with id ${id} not found`);
    }
    return user;
  }
  
  async deleteUser(id: string): Promise<void> {
    const user = await this.userRepository.findById(id);
    if (!user) {
      throw new Error(`User with id ${id} not found`);
    }
    
    const deleted = await this.userRepository.delete(id);
    if (!deleted) {
      throw new Error('Failed to delete user');
    }
    
    this.logger.info('User deleted successfully', { userId: id });
  }
}

// src/services/user-service.test.ts
import { UserService, UserRepository, EmailService, Logger, User } from './user-service';

// Mock implementations
const createMockUserRepository = (): jest.Mocked<UserRepository> => ({
  findById: jest.fn(),
  save: jest.fn(),
  delete: jest.fn(),
  findByEmail: jest.fn(),
});

const createMockEmailService = (): jest.Mocked<EmailService> => ({
  sendWelcomeEmail: jest.fn(),
  sendPasswordResetEmail: jest.fn(),
});

const createMockLogger = (): jest.Mocked<Logger> => ({
  info: jest.fn(),
  error: jest.fn(),
  warn: jest.fn(),
});

describe('UserService', () => {
  let userService: UserService;
  let mockUserRepository: jest.Mocked<UserRepository>;
  let mockEmailService: jest.Mocked<EmailService>;
  let mockLogger: jest.Mocked<Logger>;
  
  beforeEach(() => {
    mockUserRepository = createMockUserRepository();
    mockEmailService = createMockEmailService();
    mockLogger = createMockLogger();
    
    userService = new UserService(
      mockUserRepository,
      mockEmailService,
      mockLogger
    );
  });
  
  describe('createUser', () => {
    const userData = { name: 'John Doe', email: 'john@example.com' };
    const mockUser: User = {
      id: '123',
      name: 'John Doe',
      email: 'john@example.com',
      createdAt: new Date('2023-01-01')
    };
    
    it('should create a new user successfully', async () => {
      // Arrange
      mockUserRepository.findByEmail.mockResolvedValue(null);
      mockUserRepository.save.mockResolvedValue(mockUser);
      mockEmailService.sendWelcomeEmail.mockResolvedValue();
      
      // Act
      const result = await userService.createUser(userData);
      
      // Assert
      expect(result).toEqual(mockUser);
      expect(mockUserRepository.findByEmail).toHaveBeenCalledWith(userData.email);
      expect(mockUserRepository.save).toHaveBeenCalledWith(userData);
      expect(mockEmailService.sendWelcomeEmail).toHaveBeenCalledWith(mockUser);
      expect(mockLogger.info).toHaveBeenCalledWith(
        'User created successfully', 
        { userId: mockUser.id }
      );
    });
    
    it('should throw error if user already exists', async () => {
      // Arrange
      mockUserRepository.findByEmail.mockResolvedValue(mockUser);
      
      // Act & Assert
      await expect(userService.createUser(userData))
        .rejects.toThrow('User with this email already exists');
      
      expect(mockUserRepository.save).not.toHaveBeenCalled();
      expect(mockEmailService.sendWelcomeEmail).not.toHaveBeenCalled();
    });
    
    it('should create user even if welcome email fails', async () => {
      // Arrange
      mockUserRepository.findByEmail.mockResolvedValue(null);
      mockUserRepository.save.mockResolvedValue(mockUser);
      mockEmailService.sendWelcomeEmail.mockRejectedValue(new Error('Email service unavailable'));
      
      // Act
      const result = await userService.createUser(userData);
      
      // Assert
      expect(result).toEqual(mockUser);
      expect(mockLogger.warn).toHaveBeenCalledWith(
        'Failed to send welcome email',
        { userId: mockUser.id, error: 'Email service unavailable' }
      );
    });
    
    it('should log error and re-throw if user creation fails', async () => {
      // Arrange
      const error = new Error('Database error');
      mockUserRepository.findByEmail.mockResolvedValue(null);
      mockUserRepository.save.mockRejectedValue(error);
      
      // Act & Assert
      await expect(userService.createUser(userData)).rejects.toThrow('Database error');
      
      expect(mockLogger.error).toHaveBeenCalledWith('Failed to create user', error);
      expect(mockEmailService.sendWelcomeEmail).not.toHaveBeenCalled();
    });
  });
  
  describe('getUserById', () => {
    it('should return user when found', async () => {
      // Arrange
      const mockUser: User = {
        id: '123',
        name: 'John Doe',
        email: 'john@example.com',
        createdAt: new Date()
      };
      mockUserRepository.findById.mockResolvedValue(mockUser);
      
      // Act
      const result = await userService.getUserById('123');
      
      // Assert
      expect(result).toEqual(mockUser);
      expect(mockUserRepository.findById).toHaveBeenCalledWith('123');
    });
    
    it('should throw error when user not found', async () => {
      // Arrange
      mockUserRepository.findById.mockResolvedValue(null);
      
      // Act & Assert
      await expect(userService.getUserById('123'))
        .rejects.toThrow('User with id 123 not found');
    });
  });
  
  describe('deleteUser', () => {
    const mockUser: User = {
      id: '123',
      name: 'John Doe',
      email: 'john@example.com',
      createdAt: new Date()
    };
    
    it('should delete user successfully', async () => {
      // Arrange
      mockUserRepository.findById.mockResolvedValue(mockUser);
      mockUserRepository.delete.mockResolvedValue(true);
      
      // Act
      await userService.deleteUser('123');
      
      // Assert
      expect(mockUserRepository.findById).toHaveBeenCalledWith('123');
      expect(mockUserRepository.delete).toHaveBeenCalledWith('123');
      expect(mockLogger.info).toHaveBeenCalledWith(
        'User deleted successfully',
        { userId: '123' }
      );
    });
    
    it('should throw error if user not found', async () => {
      // Arrange
      mockUserRepository.findById.mockResolvedValue(null);
      
      // Act & Assert
      await expect(userService.deleteUser('123'))
        .rejects.toThrow('User with id 123 not found');
      
      expect(mockUserRepository.delete).not.toHaveBeenCalled();
    });
    
    it('should throw error if delete operation fails', async () => {
      // Arrange
      mockUserRepository.findById.mockResolvedValue(mockUser);
      mockUserRepository.delete.mockResolvedValue(false);
      
      // Act & Assert
      await expect(userService.deleteUser('123'))
        .rejects.toThrow('Failed to delete user');
    });
  });
});
```

### 2. **Spy Functions และ Partial Mocks**

```typescript
// src/utils/file-service.ts
import fs from 'fs/promises';
import path from 'path';

export class FileService {
  async readFile(filePath: string): Promise<string> {
    try {
      const content = await fs.readFile(filePath, 'utf-8');
      return content;
    } catch (error) {
      throw new Error(`Failed to read file: ${filePath}`);
    }
  }
  
  async writeFile(filePath: string, content: string): Promise<void> {
    try {
      const dir = path.dirname(filePath);
      await fs.mkdir(dir, { recursive: true });
      await fs.writeFile(filePath, content, 'utf-8');
    } catch (error) {
      throw new Error(`Failed to write file: ${filePath}`);
    }
  }
  
  async fileExists(filePath: string): Promise<boolean> {
    try {
      await fs.access(filePath);
      return true;
    } catch {
      return false;
    }
  }
  
  async deleteFile(filePath: string): Promise<void> {
    try {
      await fs.unlink(filePath);
    } catch (error) {
      throw new Error(`Failed to delete file: ${filePath}`);
    }
  }
}

// src/utils/file-service.test.ts
import fs from 'fs/promises';
import path from 'path';
import { FileService } from './file-service';

// Mock the fs module
jest.mock('fs/promises');
jest.mock('path');

const mockedFs = fs as jest.Mocked<typeof fs>;
const mockedPath = path as jest.Mocked<typeof path>;

describe('FileService', () => {
  let fileService: FileService;
  
  beforeEach(() => {
    fileService = new FileService();
    jest.clearAllMocks();
  });
  
  describe('readFile', () => {
    it('should read file content successfully', async () => {
      // Arrange
      const filePath = '/test/file.txt';
      const content = 'Hello, World!';
      mockedFs.readFile.mockResolvedValue(content);
      
      // Act
      const result = await fileService.readFile(filePath);
      
      // Assert
      expect(result).toBe(content);
      expect(mockedFs.readFile).toHaveBeenCalledWith(filePath, 'utf-8');
    });
    
    it('should throw error when file read fails', async () => {
      // Arrange
      const filePath = '/test/nonexistent.txt';
      mockedFs.readFile.mockRejectedValue(new Error('ENOENT'));
      
      // Act & Assert
      await expect(fileService.readFile(filePath))
        .rejects.toThrow('Failed to read file: /test/nonexistent.txt');
    });
  });
  
  describe('writeFile', () => {
    it('should write file successfully', async () => {
      // Arrange
      const filePath = '/test/dir/file.txt';
      const content = 'Hello, World!';
      const dir = '/test/dir';
      
      mockedPath.dirname.mockReturnValue(dir);
      mockedFs.mkdir.mockResolvedValue(undefined);
      mockedFs.writeFile.mockResolvedValue();
      
      // Act
      await fileService.writeFile(filePath, content);
      
      // Assert
      expect(mockedPath.dirname).toHaveBeenCalledWith(filePath);
      expect(mockedFs.mkdir).toHaveBeenCalledWith(dir, { recursive: true });
      expect(mockedFs.writeFile).toHaveBeenCalledWith(filePath, content, 'utf-8');
    });
    
    it('should throw error when write fails', async () => {
      // Arrange
      const filePath = '/readonly/file.txt';
      const content = 'content';
      
      mockedPath.dirname.mockReturnValue('/readonly');
      mockedFs.mkdir.mockResolvedValue(undefined);
      mockedFs.writeFile.mockRejectedValue(new Error('Permission denied'));
      
      // Act & Assert
      await expect(fileService.writeFile(filePath, content))
        .rejects.toThrow('Failed to write file: /readonly/file.txt');
    });
  });
  
  describe('fileExists', () => {
    it('should return true when file exists', async () => {
      // Arrange
      mockedFs.access.mockResolvedValue();
      
      // Act
      const result = await fileService.fileExists('/test/file.txt');
      
      // Assert
      expect(result).toBe(true);
      expect(mockedFs.access).toHaveBeenCalledWith('/test/file.txt');
    });
    
    it('should return false when file does not exist', async () => {
      // Arrange
      mockedFs.access.mockRejectedValue(new Error('ENOENT'));
      
      // Act
      const result = await fileService.fileExists('/test/nonexistent.txt');
      
      // Assert
      expect(result).toBe(false);
    });
  });
});

// Spy example
describe('FileService with spies', () => {
  let fileService: FileService;
  
  beforeEach(() => {
    fileService = new FileService();
  });
  
  it('should call readFile method when reading file', async () => {
    // Create spy on the instance method
    const readFileSpy = jest.spyOn(fileService, 'readFile');
    readFileSpy.mockResolvedValue('mocked content');
    
    // Act
    const result = await fileService.readFile('/test/file.txt');
    
    // Assert
    expect(readFileSpy).toHaveBeenCalledWith('/test/file.txt');
    expect(result).toBe('mocked content');
    
    // Cleanup
    readFileSpy.mockRestore();
  });
});
```

## 🔗 Integration Testing

### 1. **Database Integration Tests**

```typescript
// src/repositories/user-repository.ts
import { User } from '../models/user';

export interface DatabaseConnection {
  query<T>(sql: string, params?: any[]): Promise<T[]>;
  execute(sql: string, params?: any[]): Promise<{ affectedRows: number; insertId?: number }>;
}

export class UserRepository {
  constructor(private db: DatabaseConnection) {}
  
  async findById(id: string): Promise<User | null> {
    const results = await this.db.query<User>(
      'SELECT * FROM users WHERE id = ?',
      [id]
    );
    
    return results[0] || null;
  }
  
  async save(user: Omit<User, 'id' | 'createdAt'>): Promise<User> {
    const result = await this.db.execute(
      'INSERT INTO users (name, email) VALUES (?, ?)',
      [user.name, user.email]
    );
    
    if (!result.insertId) {
      throw new Error('Failed to create user');
    }
    
    return {
      id: result.insertId.toString(),
      ...user,
      createdAt: new Date()
    };
  }
  
  async findByEmail(email: string): Promise<User | null> {
    const results = await this.db.query<User>(
      'SELECT * FROM users WHERE email = ?',
      [email]
    );
    
    return results[0] || null;
  }
  
  async delete(id: string): Promise<boolean> {
    const result = await this.db.execute(
      'DELETE FROM users WHERE id = ?',
      [id]
    );
    
    return result.affectedRows > 0;
  }
}

// src/repositories/user-repository.integration.test.ts
import { UserRepository, DatabaseConnection } from './user-repository';
import { User } from '../models/user';

// Mock database connection for integration testing
class MockDatabaseConnection implements DatabaseConnection {
  private users: User[] = [];
  private nextId = 1;
  
  async query<T>(sql: string, params: any[] = []): Promise<T[]> {
    if (sql.includes('SELECT * FROM users WHERE id = ?')) {
      const id = params[0];
      const user = this.users.find(u => u.id === id);
      return user ? [user as T] : [];
    }
    
    if (sql.includes('SELECT * FROM users WHERE email = ?')) {
      const email = params[0];
      const user = this.users.find(u => u.email === email);
      return user ? [user as T] : [];
    }
    
    return [];
  }
  
  async execute(sql: string, params: any[] = []): Promise<{ affectedRows: number; insertId?: number }> {
    if (sql.includes('INSERT INTO users')) {
      const [name, email] = params;
      const newUser: User = {
        id: this.nextId.toString(),
        name,
        email,
        createdAt: new Date()
      };
      this.users.push(newUser);
      return { affectedRows: 1, insertId: this.nextId++ };
    }
    
    if (sql.includes('DELETE FROM users WHERE id = ?')) {
      const id = params[0];
      const index = this.users.findIndex(u => u.id === id);
      if (index !== -1) {
        this.users.splice(index, 1);
        return { affectedRows: 1 };
      }
      return { affectedRows: 0 };
    }
    
    return { affectedRows: 0 };
  }
  
  // Helper methods for testing
  reset(): void {
    this.users = [];
    this.nextId = 1;
  }
  
  seed(users: User[]): void {
    this.users = [...users];
    this.nextId = Math.max(...users.map(u => parseInt(u.id)), 0) + 1;
  }
}

describe('UserRepository Integration Tests', () => {
  let repository: UserRepository;
  let mockDb: MockDatabaseConnection;
  
  beforeEach(() => {
    mockDb = new MockDatabaseConnection();
    repository = new UserRepository(mockDb);
  });
  
  afterEach(() => {
    mockDb.reset();
  });
  
  describe('save and findById', () => {
    it('should save user and retrieve by id', async () => {
      // Arrange
      const userData = { name: 'John Doe', email: 'john@example.com' };
      
      // Act
      const savedUser = await repository.save(userData);
      const retrievedUser = await repository.findById(savedUser.id);
      
      // Assert
      expect(savedUser).toEqual(expect.objectContaining({
        id: expect.any(String),
        name: userData.name,
        email: userData.email,
        createdAt: expect.any(Date)
      }));
      
      expect(retrievedUser).toEqual(savedUser);
    });
    
    it('should return null when user not found', async () => {
      // Act
      const user = await repository.findById('nonexistent');
      
      // Assert
      expect(user).toBeNull();
    });
  });
  
  describe('findByEmail', () => {
    it('should find user by email', async () => {
      // Arrange
      const userData = { name: 'Jane Doe', email: 'jane@example.com' };
      const savedUser = await repository.save(userData);
      
      // Act
      const foundUser = await repository.findByEmail(userData.email);
      
      // Assert
      expect(foundUser).toEqual(savedUser);
    });
    
    it('should return null when email not found', async () => {
      // Act
      const user = await repository.findByEmail('notfound@example.com');
      
      // Assert
      expect(user).toBeNull();
    });
  });
  
  describe('delete', () => {
    it('should delete existing user', async () => {
      // Arrange
      const userData = { name: 'Bob Smith', email: 'bob@example.com' };
      const savedUser = await repository.save(userData);
      
      // Act
      const deleted = await repository.delete(savedUser.id);
      const user = await repository.findById(savedUser.id);
      
      // Assert
      expect(deleted).toBe(true);
      expect(user).toBeNull();
    });
    
    it('should return false when deleting non-existent user', async () => {
      // Act
      const deleted = await repository.delete('nonexistent');
      
      // Assert
      expect(deleted).toBe(false);
    });
  });
  
  describe('complex scenarios', () => {
    it('should handle multiple users correctly', async () => {
      // Arrange
      const users = [
        { name: 'User 1', email: 'user1@example.com' },
        { name: 'User 2', email: 'user2@example.com' },
        { name: 'User 3', email: 'user3@example.com' }
      ];
      
      // Act
      const savedUsers = await Promise.all(
        users.map(user => repository.save(user))
      );
      
      // Assert
      expect(savedUsers).toHaveLength(3);
      expect(savedUsers.map(u => u.email)).toEqual(
        expect.arrayContaining(users.map(u => u.email))
      );
      
      // Verify each user can be retrieved
      for (const savedUser of savedUsers) {
        const retrieved = await repository.findById(savedUser.id);
        expect(retrieved).toEqual(savedUser);
      }
    });
  });
});
```

### 2. **API Integration Tests**

```typescript
// src/controllers/user-controller.ts
import { Request, Response } from 'express';
import { UserService } from '../services/user-service';

export class UserController {
  constructor(private userService: UserService) {}
  
  async createUser(req: Request, res: Response): Promise<void> {
    try {
      const { name, email } = req.body;
      
      if (!name || !email) {
        res.status(400).json({
          error: 'Name and email are required'
        });
        return;
      }
      
      const user = await this.userService.createUser({ name, email });
      res.status(201).json(user);
    } catch (error) {
      if (error instanceof Error && error.message.includes('already exists')) {
        res.status(409).json({ error: error.message });
      } else {
        res.status(500).json({ error: 'Internal server error' });
      }
    }
  }
  
  async getUser(req: Request, res: Response): Promise<void> {
    try {
      const { id } = req.params;
      const user = await this.userService.getUserById(id);
      res.json(user);
    } catch (error) {
      if (error instanceof Error && error.message.includes('not found')) {
        res.status(404).json({ error: error.message });
      } else {
        res.status(500).json({ error: 'Internal server error' });
      }
    }
  }
  
  async deleteUser(req: Request, res: Response): Promise<void> {
    try {
      const { id } = req.params;
      await this.userService.deleteUser(id);
      res.status(204).send();
    } catch (error) {
      if (error instanceof Error && error.message.includes('not found')) {
        res.status(404).json({ error: error.message });
      } else {
        res.status(500).json({ error: 'Internal server error' });
      }
    }
  }
}

// src/controllers/user-controller.integration.test.ts
import request from 'supertest';
import express, { Express } from 'express';
import { UserController } from './user-controller';
import { UserService } from '../services/user-service';
import { User } from '../models/user';

// Create test app
function createTestApp(userService: UserService): Express {
  const app = express();
  app.use(express.json());
  
  const userController = new UserController(userService);
  
  app.post('/users', (req, res) => userController.createUser(req, res));
  app.get('/users/:id', (req, res) => userController.getUser(req, res));
  app.delete('/users/:id', (req, res) => userController.deleteUser(req, res));
  
  return app;
}

describe('UserController Integration Tests', () => {
  let app: Express;
  let mockUserService: jest.Mocked<UserService>;
  
  beforeEach(() => {
    mockUserService = {
      createUser: jest.fn(),
      getUserById: jest.fn(),
      deleteUser: jest.fn(),
    } as any;
    
    app = createTestApp(mockUserService);
  });
  
  describe('POST /users', () => {
    const validUserData = { name: 'John Doe', email: 'john@example.com' };
    const mockUser: User = {
      id: '123',
      ...validUserData,
      createdAt: new Date('2023-01-01')
    };
    
    it('should create user successfully', async () => {
      // Arrange
      mockUserService.createUser.mockResolvedValue(mockUser);
      
      // Act & Assert
      const response = await request(app)
        .post('/users')
        .send(validUserData)
        .expect(201);
      
      expect(response.body).toEqual({
        id: '123',
        name: 'John Doe',
        email: 'john@example.com',
        createdAt: '2023-01-01T00:00:00.000Z'
      });
      
      expect(mockUserService.createUser).toHaveBeenCalledWith(validUserData);
    });
    
    it('should return 400 when name is missing', async () => {
      // Act & Assert
      const response = await request(app)
        .post('/users')
        .send({ email: 'john@example.com' })
        .expect(400);
      
      expect(response.body).toEqual({
        error: 'Name and email are required'
      });
      
      expect(mockUserService.createUser).not.toHaveBeenCalled();
    });
    
    it('should return 400 when email is missing', async () => {
      // Act & Assert
      const response = await request(app)
        .post('/users')
        .send({ name: 'John Doe' })
        .expect(400);
      
      expect(response.body).toEqual({
        error: 'Name and email are required'
      });
    });
    
    it('should return 409 when user already exists', async () => {
      // Arrange
      mockUserService.createUser.mockRejectedValue(
        new Error('User with this email already exists')
      );
      
      // Act & Assert
      const response = await request(app)
        .post('/users')
        .send(validUserData)
        .expect(409);
      
      expect(response.body).toEqual({
        error: 'User with this email already exists'
      });
    });
    
    it('should return 500 for unexpected errors', async () => {
      // Arrange
      mockUserService.createUser.mockRejectedValue(
        new Error('Database connection failed')
      );
      
      // Act & Assert
      const response = await request(app)
        .post('/users')
        .send(validUserData)
        .expect(500);
      
      expect(response.body).toEqual({
        error: 'Internal server error'
      });
    });
  });
  
  describe('GET /users/:id', () => {
    const mockUser: User = {
      id: '123',
      name: 'John Doe',
      email: 'john@example.com',
      createdAt: new Date('2023-01-01')
    };
    
    it('should return user when found', async () => {
      // Arrange
      mockUserService.getUserById.mockResolvedValue(mockUser);
      
      // Act & Assert
      const response = await request(app)
        .get('/users/123')
        .expect(200);
      
      expect(response.body).toEqual({
        id: '123',
        name: 'John Doe',
        email: 'john@example.com',
        createdAt: '2023-01-01T00:00:00.000Z'
      });
      
      expect(mockUserService.getUserById).toHaveBeenCalledWith('123');
    });
    
    it('should return 404 when user not found', async () => {
      // Arrange
      mockUserService.getUserById.mockRejectedValue(
        new Error('User with id 123 not found')
      );
      
      // Act & Assert
      const response = await request(app)
        .get('/users/123')
        .expect(404);
      
      expect(response.body).toEqual({
        error: 'User with id 123 not found'
      });
    });
  });
  
  describe('DELETE /users/:id', () => {
    it('should delete user successfully', async () => {
      // Arrange
      mockUserService.deleteUser.mockResolvedValue();
      
      // Act & Assert
      await request(app)
        .delete('/users/123')
        .expect(204);
      
      expect(mockUserService.deleteUser).toHaveBeenCalledWith('123');
    });
    
    it('should return 404 when user not found', async () => {
      // Arrange
      mockUserService.deleteUser.mockRejectedValue(
        new Error('User with id 123 not found')
      );
      
      // Act & Assert
      const response = await request(app)
        .delete('/users/123')
        .expect(404);
      
      expect(response.body).toEqual({
        error: 'User with id 123 not found'
      });
    });
  });
});
```

## 📝 แบบฝึกหัด

### 1. **Shopping Cart Testing**
สร้าง test suite สำหรับ shopping cart system:
- Unit tests สำหรับ cart operations
- Mock external services (payment, inventory)
- Integration tests สำหรับ complete purchase flow
- Error handling scenarios

### 2. **Authentication Service Testing**
ออกแบบ test สำหรับ authentication system:
- Password hashing and validation
- JWT token generation and verification
- Session management
- Security edge cases

### 3. **File Processing Service Testing**
สร้าง comprehensive tests สำหรับ file processing:
- Different file types handling
- Error scenarios (corrupted files, permissions)
- Async processing with progress tracking
- Integration with storage services

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 14: Error Handling](./14-error-handling.md)

**บทต่อไป**: [บทที่ 16: Performance Optimization](./16-performance.md)

**กลับไปหน้าหลัก**: [README](./README.md)
