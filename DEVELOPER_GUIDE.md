# Developer Guide

This guide provides detailed information for developers working on the News Posts Microservices Application.

## 🛠️ Development Environment Setup

### 1. Prerequisites Installation

#### Node.js and npm
```bash
# Download and install Node.js (v18+) from nodejs.org
node --version  # Should show v18 or higher
npm --version   # Should show v9 or higher
```

#### PostgreSQL
```bash
# Install PostgreSQL and create a database
# Update connection details in server/.env
```

#### Docker
```bash
# Install Docker Desktop for your OS
docker --version
docker-compose --version
```

### 2. Environment Configuration

#### Server Environment (.env)
Create `server/.env` file:
```env
# Database Configuration
DB_HOST=localhost
DB_PORT=5432
DB_USERNAME=your_db_user
DB_PASSWORD=your_db_password
DB_DATABASE=news_posts_db

# JWT Configuration
JWT_SECRET=your-super-secret-jwt-key
JWT_EXPIRES_IN=7d

# Server Configuration
PORT=8000
NODE_ENV=development

# Redis Configuration
REDIS_HOST=localhost
REDIS_PORT=6379

# User Service Configuration
USER_SERVICE_URL=http://localhost:3001
LOGGING_SERVICE_URL=http://localhost:3002
```

#### User Service Environment
Create `microservices/user-service/.env`:
```env
PORT=3001
JWT_SECRET=your-super-secret-jwt-key
JWT_EXPIRES_IN=7d
REDIS_HOST=localhost
REDIS_PORT=6379
NODE_ENV=development
```

#### Logging Service Environment
Create `microservices/logging-service/.env`:
```env
PORT=3002
REDIS_HOST=localhost
REDIS_PORT=6379
NODE_ENV=development
```

## 🔧 Development Workflow

### 1. Starting Development Environment

```bash
# 1. Start Redis
npm run start:redis

# 2. Start microservices (in separate terminals)
cd microservices/user-service && npm run dev
cd microservices/logging-service && npm run dev

# 3. Start main server (in separate terminal)
cd server && npm run dev:server

# 4. Start client (in separate terminal)
cd client && npm run dev:client
```

### 2. Development Scripts

| Service | Command | Description |
|---------|---------|-------------|
| Client | `npm run dev:client` | Hot reload development server |
| Server | `npm run dev:server` | Nodemon with TypeScript |
| User Service | `npm run dev` | Development mode with hot reload |
| Logging Service | `npm run dev` | Development mode with hot reload |

### 3. Building for Production

```bash
# Build all services
npm run build:client
npm run build:server
npm run build:user-service
npm run build:logging-service

# Or use combined scripts
npm run prod:client
npm run prod:server
npm run prod:microservices
```

## 📂 Code Organization

### Frontend (React + TypeScript)

```
client/src/
├── components/           # Reusable UI components
│   ├── AuthModal/       # Authentication modal
│   ├── PostCard/        # News post card component
│   ├── Pagination/      # Pagination component
│   ├── GenreFilter/     # Category filter component
│   └── ...
├── pages/               # Page-level components
│   ├── HomePage/        # Home page with news list
│   ├── PostDetailPage/  # Individual post view
│   └── PostFormPage/    # Create/edit post form
├── contexts/            # React Context providers
│   └── AuthContext.tsx  # Authentication state management
├── services/            # API communication
│   ├── api.ts          # HTTP client and API calls
│   └── dateFormatter.ts # Date utility functions
└── types/               # TypeScript type definitions
    └── index.ts        # Shared type definitions
```

#### Key Frontend Patterns

```typescript
// API Service Pattern
export const newsAPI = {
  getAllPosts: async (params?: PostQueryParams): Promise<PaginatedResponse<Post>> => {
    // Implementation
  },
  // ... other methods
};

// Context Pattern for State Management
export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
};

// Protected Route Pattern
const ProtectedRoute: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const { isAuthenticated, loading } = useAuth();
  // Implementation
};
```

### Backend (Express + TypeORM)

```
server/src/
├── controller/          # Express route handlers
│   └── newspostsController.ts
├── entities/           # TypeORM database entities
│   ├── Post.ts
│   ├── User.ts
│   └── Category.ts
├── routes/             # Express route definitions
│   ├── newsPosts.ts
│   ├── staticGet.ts
│   └── health.ts
├── services/           # Business logic layer
│   └── postsService.ts
├── middleware/         # Express middleware
│   └── swaggerGuard.ts
├── migrations/         # Database migrations
└── scripts/           # Utility scripts
    ├── generateSwagger.ts
    ├── seedDatabase.ts
    └── resetDatabase.ts
```

#### Key Backend Patterns

```typescript
// Controller Pattern
export const getAllPosts = async (req: Request, res: Response): Promise<void> => {
  try {
    const result = await postsService.getAllPosts(queryParams);
    res.json(result);
  } catch (error) {
    handleError(error, res);
  }
};

// Service Layer Pattern
export class PostsService {
  async getAllPosts(params: QueryParams): Promise<PaginatedResponse<Post>> {
    // Business logic implementation
  }
}

// Entity Pattern (TypeORM)
@Entity('posts')
export class Post {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  title: string;
  
  // ... other properties
}
```

### Microservices Architecture

```
microservices/
├── user-service/       # User management microservice
│   ├── controllers/    # User-related controllers
│   ├── services/      # User business logic
│   ├── models/        # User data models
│   └── types/         # User-specific types
└── logging-service/   # Logging microservice
    ├── controllers/   # Logging controllers
    ├── services/     # Logging logic
    └── logs/         # Log file storage
```

#### Microservice Communication Pattern

```typescript
// Redis Pub/Sub for inter-service communication
export class UserService {
  async createUser(userData: CreateUserRequest) {
    const user = await this.saveUser(userData);
    
    // Publish event to logging service
    await this.redisClient.publish('user:created', JSON.stringify({
      userId: user.id,
      email: user.email,
      timestamp: new Date()
    }));
    
    return user;
  }
}
```

## 🧪 Testing

### Setting Up Tests

```bash
# Install testing dependencies (if not already installed)
npm install --save-dev jest @types/jest supertest
```

### Test Structure

```
__tests__/
├── unit/              # Unit tests
│   ├── services/     # Service layer tests
│   ├── controllers/  # Controller tests
│   └── utils/        # Utility function tests
├── integration/       # Integration tests
│   ├── api/          # API endpoint tests
│   └── database/     # Database interaction tests
└── e2e/              # End-to-end tests
    └── user-flows/   # Complete user journey tests
```

### Example Test

```typescript
// Example service test
describe('PostsService', () => {
  let postsService: PostsService;
  
  beforeEach(() => {
    postsService = new PostsService();
  });
  
  it('should return paginated posts', async () => {
    const result = await postsService.getAllPosts({ page: 1, size: 10 });
    
    expect(result).toHaveProperty('data');
    expect(result).toHaveProperty('totalCount');
    expect(result.data).toBeInstanceOf(Array);
  });
});
```

## 🚀 Deployment

### Production Build

```bash
# Build all services
npm run prod:client
npm run prod:server
npm run prod:microservices

# Start production services
npm run start:app
```

### Docker Deployment

```bash
# Build and run with Docker Compose
docker-compose -f docker-compose.microservices.yml up --build
```

### Environment Variables for Production

```env
NODE_ENV=production
JWT_SECRET=your-production-jwt-secret
DB_HOST=production-db-host
REDIS_HOST=production-redis-host
```

## 🔍 Debugging

### Debug Configuration (VS Code)

Create `.vscode/launch.json`:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug Server",
      "type": "node",
      "request": "launch",
      "program": "${workspaceFolder}/server/server.ts",
      "outFiles": ["${workspaceFolder}/server/dist/**/*.js"],
      "runtimeArgs": ["-r", "ts-node/register"],
      "env": {
        "NODE_ENV": "development"
      }
    },
    {
      "name": "Debug User Service",
      "type": "node",
      "request": "launch",
      "program": "${workspaceFolder}/microservices/user-service/src/server.ts",
      "outFiles": ["${workspaceFolder}/microservices/user-service/dist/**/*.js"],
      "runtimeArgs": ["-r", "ts-node/register"]
    }
  ]
}
```

### Logging and Monitoring

#### Server Logging
```typescript
import { logger } from './helpers/logger';

// Use structured logging
logger.info('User created', { userId, email });
logger.error('Database error', { error: error.message, stack: error.stack });
```

#### Client-side Debugging
```typescript
// Use React Developer Tools
// Enable Redux DevTools if using Redux
// Use browser network tab for API debugging
```

## 📊 Performance Optimization

### Database Optimization

```typescript
// Use proper indexing
@Entity('posts')
export class Post {
  @Index()
  @Column()
  category: string;
  
  @Index()
  @Column()
  createdAt: Date;
}

// Use query optimization
const posts = await this.postRepository
  .createQueryBuilder('post')
  .leftJoinAndSelect('post.category', 'category')
  .where('post.deleted = :deleted', { deleted: false })
  .orderBy('post.createdAt', 'DESC')
  .take(limit)
  .skip(offset)
  .getMany();
```

### Client-side Optimization

```typescript
// Implement lazy loading
const LazyPostForm = React.lazy(() => import('./pages/PostFormPage/PostFormPage'));

// Use React.memo for expensive components
const PostCard = React.memo(({ post }: { post: Post }) => {
  // Component implementation
});

// Optimize API calls with caching
const usePostsQuery = (params: QueryParams) => {
  return useQuery(['posts', params], () => newsAPI.getAllPosts(params), {
    staleTime: 5 * 60 * 1000, // 5 minutes
  });
};
```

## 🔐 Security Best Practices

### JWT Security
```typescript
// Use strong JWT secrets
const JWT_SECRET = process.env.JWT_SECRET || crypto.randomBytes(64).toString('hex');

// Implement token refresh
const refreshToken = async (token: string) => {
  // Verify and refresh logic
};

// Sanitize JWT payload
const createToken = (user: User) => {
  return jwt.sign(
    { 
      userId: user.id, 
      email: user.email 
      // Don't include sensitive data
    },
    JWT_SECRET,
    { expiresIn: '7d' }
  );
};
```

### Input Validation
```typescript
// Use validation middleware
import { body, validationResult } from 'express-validator';

export const validateCreatePost = [
  body('title').notEmpty().withMessage('Title is required'),
  body('content').isLength({ min: 10 }).withMessage('Content must be at least 10 characters'),
  body('category').notEmpty().withMessage('Category is required'),
];
```

### CORS Configuration
```typescript
app.use(cors({
  origin: process.env.NODE_ENV === 'production' 
    ? ['https://yourdomain.com'] 
    : ['http://localhost:5173'],
  credentials: true
}));
```

## 🔄 Common Development Tasks

### Adding a New API Endpoint

1. **Define the route** in `server/routes/`
2. **Create controller method** in `server/controller/`
3. **Add service logic** in `server/services/`
4. **Update Swagger documentation** with comments
5. **Add frontend API call** in `client/src/services/api.ts`
6. **Update TypeScript types** in respective `types/` directories

### Adding a New React Component

1. **Create component directory** in `client/src/components/`
2. **Implement component** with TypeScript
3. **Add styles** (CSS/SCSS file)
4. **Export from index** if needed
5. **Add to parent component** or page

### Database Schema Changes

1. **Update entity** in `server/entities/`
2. **Generate migration**: `npm run typeorm:migration:generate -- MigrationName`
3. **Review generated migration** in `server/migrations/`
4. **Run migration**: `npm run typeorm:migration:run`
5. **Update seed data** if necessary

This developer guide should help you navigate the codebase and implement new features efficiently. For specific questions, refer to the individual service documentation or the main README.md file.
