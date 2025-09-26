# API Documentation

This document provides comprehensive information about the News Posts Microservices Application API endpoints.

## 🌐 Base URLs

- **Main API Server**: `http://localhost:8000`
- **User Service**: `http://localhost:3001`
- **Logging Service**: `http://localhost:3002`
- **Interactive Documentation**: `http://localhost:8000/api-docs` (Swagger UI)

## 🔐 Authentication

The application uses JWT (JSON Web Tokens) for authentication. Include the token in the Authorization header:

```
Authorization: Bearer <your-jwt-token>
```

### Authentication Flow

1. **Register**: `POST /users/create`
2. **Login**: `POST /users/login`
3. **Use Token**: Include in subsequent requests

## 📰 Main API Endpoints (Port 8000)

### News Posts Management

#### Get All Posts
```http
GET /api/newsposts
```

**Query Parameters:**
| Parameter | Type | Description | Default |
|-----------|------|-------------|---------|
| `page` | number | Page number for pagination | 1 |
| `size` | number | Number of items per page | 10 |
| `category` | string | Filter by category name | - |

**Response:**
```json
{
  "data": [
    {
      "id": "uuid",
      "title": "Post Title",
      "content": "Post content...",
      "category": "Technology",
      "createdAt": "2025-09-27T00:00:00Z",
      "updatedAt": "2025-09-27T00:00:00Z",
      "deleted": false
    }
  ],
  "totalCount": 50,
  "page": 1,
  "size": 10,
  "totalPages": 5
}
```

#### Get Post by ID
```http
GET /api/newsposts/{id}
```

**Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | string | Post UUID |

**Response:**
```json
{
  "id": "uuid",
  "title": "Post Title",
  "content": "Post content...",
  "category": "Technology",
  "createdAt": "2025-09-27T00:00:00Z",
  "updatedAt": "2025-09-27T00:00:00Z",
  "deleted": false
}
```

#### Create New Post
```http
POST /api/newsposts
```
**🔒 Requires Authentication**

**Request Body:**
```json
{
  "title": "New Post Title",
  "content": "Post content goes here...",
  "category": "Technology"
}
```

**Response:**
```json
{
  "id": "uuid",
  "title": "New Post Title",
  "content": "Post content goes here...",
  "category": "Technology",
  "createdAt": "2025-09-27T00:00:00Z",
  "updatedAt": "2025-09-27T00:00:00Z",
  "deleted": false
}
```

#### Update Post
```http
PUT /api/newsposts/{id}
```
**🔒 Requires Authentication**

**Request Body:**
```json
{
  "title": "Updated Post Title",
  "content": "Updated content...",
  "category": "Technology"
}
```

#### Delete Post
```http
DELETE /api/newsposts/{id}
```
**🔒 Requires Authentication**

**Response:**
```json
{
  "message": "Post deleted successfully"
}
```

#### Get Categories
```http
GET /api/newsposts/categories
```

**Response:**
```json
[
  {
    "id": "uuid",
    "name": "Technology",
    "description": "Technology related posts"
  },
  {
    "id": "uuid",
    "name": "Sports",
    "description": "Sports news and updates"
  }
]
```

### Health Check
```http
GET /api/health
```

**Response:**
```json
{
  "status": "OK",
  "timestamp": "2025-09-27T00:00:00Z",
  "services": {
    "database": "connected",
    "redis": "connected"
  }
}
```

## 👤 User Service Endpoints (Port 3001)

### User Registration
```http
POST /users/create
```

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "password123",
  "confirmPassword": "password123"
}
```

**Response:**
```json
{
  "message": "User created successfully",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": "uuid",
    "email": "user@example.com",
    "createdAt": "2025-09-27T00:00:00Z"
  }
}
```

**Error Responses:**
```json
// 400 - Validation Error
{
  "error": "Email, password, and confirmPassword are required"
}

// 409 - Conflict
{
  "error": "User with this email already exists"
}
```

### User Login
```http
POST /users/login
```

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "password123"
}
```

**Response:**
```json
{
  "message": "Login successful",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": "uuid",
    "email": "user@example.com",
    "createdAt": "2025-09-27T00:00:00Z"
  }
}
```

**Error Responses:**
```json
// 400 - Validation Error
{
  "error": "Email and password are required"
}

// 401 - Authentication Error
{
  "error": "Invalid email or password"
}
```

### Get User by ID
```http
GET /users/{id}
```

**Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | string | User UUID |

**Response:**
```json
{
  "user": {
    "id": "uuid",
    "email": "user@example.com",
    "createdAt": "2025-09-27T00:00:00Z"
  }
}
```

### User Service Health Check
```http
GET /health
```

**Response:**
```json
{
  "status": "OK",
  "service": "user-service"
}
```

## 📊 Logging Service Endpoints (Port 3002)

### Health Check
```http
GET /health
```

**Response:**
```json
{
  "status": "OK",
  "service": "logging-service"
}
```

### Get Application Logs (if implemented)
```http
GET /logs
```

**Query Parameters:**
| Parameter | Type | Description | Default |
|-----------|------|-------------|---------|
| `level` | string | Log level (info, error, warn) | all |
| `limit` | number | Number of log entries | 100 |
| `date` | string | Filter by date (YYYY-MM-DD) | today |

## 🔄 Data Models

### Post Model
```typescript
interface Post {
  id: string;                    // UUID
  title: string;                 // Post title
  content: string;               // Post content (HTML/text)
  category: string;              // Category name
  createdAt: string;             // ISO date string
  updatedAt: string;             // ISO date string
  deleted: boolean;              // Soft delete flag
}
```

### User Model
```typescript
interface User {
  id: string;                    // UUID
  email: string;                 // User email (unique)
  createdAt: string;             // ISO date string
}
```

### Category Model
```typescript
interface Category {
  id: string;                    // UUID
  name: string;                  // Category name (unique)
  description?: string;          // Optional description
}
```

### Paginated Response
```typescript
interface PaginatedResponse<T> {
  data: T[];                     // Array of items
  totalCount: number;            // Total number of items
  page: number;                  // Current page number
  size: number;                  // Items per page
  totalPages: number;            // Total number of pages
}
```

### Authentication Response
```typescript
interface AuthResponse {
  message: string;               // Success message
  token: string;                 // JWT token
  user: User;                    // User information
}
```

## 🚨 Error Handling

### HTTP Status Codes

| Code | Description | When Used |
|------|-------------|-----------|
| 200 | OK | Successful GET, PUT requests |
| 201 | Created | Successful POST requests |
| 400 | Bad Request | Invalid request data |
| 401 | Unauthorized | Missing or invalid authentication |
| 403 | Forbidden | Valid auth but insufficient permissions |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Resource already exists |
| 500 | Internal Server Error | Server-side errors |

### Error Response Format
```json
{
  "error": "Error message describing what went wrong",
  "details": {
    "field": "Additional error details if applicable"
  },
  "timestamp": "2025-09-27T00:00:00Z"
}
```

## 🧪 API Testing Examples

### Using cURL

#### Register a new user
```bash
curl -X POST http://localhost:3001/users/create \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@example.com",
    "password": "password123",
    "confirmPassword": "password123"
  }'
```

#### Login
```bash
curl -X POST http://localhost:3001/users/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@example.com",
    "password": "password123"
  }'
```

#### Get all posts
```bash
curl http://localhost:8000/api/newsposts?page=1&size=5
```

#### Create a new post (with auth)
```bash
curl -X POST http://localhost:8000/api/newsposts \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -d '{
    "title": "New Post",
    "content": "This is the content of the new post.",
    "category": "Technology"
  }'
```

### Using JavaScript/Fetch

#### Login and create post
```javascript
// Login
const loginResponse = await fetch('http://localhost:3001/users/login', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    email: 'test@example.com',
    password: 'password123'
  })
});

const { token } = await loginResponse.json();

// Create post with auth token
const postResponse = await fetch('http://localhost:8000/api/newsposts', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${token}`
  },
  body: JSON.stringify({
    title: 'My New Post',
    content: 'Content of my new post',
    category: 'Technology'
  })
});

const newPost = await postResponse.json();
```

## 🔧 Rate Limiting

Currently, no rate limiting is implemented, but for production deployment, consider:

- **User Registration**: 5 requests per hour per IP
- **User Login**: 10 requests per minute per IP
- **API Calls**: 100 requests per minute per authenticated user
- **Public Endpoints**: 50 requests per minute per IP

## 📱 CORS Configuration

The API supports CORS for the following origins:
- `http://localhost:5173` (development)
- Production domain (when deployed)

## 🔍 Swagger Documentation

For interactive API testing and detailed schemas, visit:
- **Swagger UI**: http://localhost:8000/api-docs
- **OpenAPI JSON**: http://localhost:8000/api-docs.json

The Swagger documentation includes:
- Complete endpoint documentation
- Request/response schemas
- Interactive testing interface
- Authentication examples
- Error response examples

---

For more information about implementation details, see the [Developer Guide](DEVELOPER_GUIDE.md) and main [README](README.md).
