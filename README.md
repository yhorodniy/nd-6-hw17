# News Posts Microservices Application

A full-stack news management application built with microservices architecture, featuring a React frontend, Express.js backend, and dedicated microservices for user management and logging.

## 🏗️ Architecture Overview

This application follows a microservices architecture pattern with the## 📚 Documentation

| Document | Description |
|----------|-------------|
| [Quick Start Guide](QUICK_START.md) | Get running in under 5 minutes |
| [API Documentation](API_DOCUMENTATION.md) | Complete API reference with examples |
| [Developer Guide](DEVELOPER_GUIDE.md) | Detailed development setup and patterns |
| [Deployment Guide](DEPLOYMENT_GUIDE.md) | Production deployment instructions |
| [FAQ](FAQ.md) | Frequently asked questions and troubleshooting |
| [Swagger Documentation](http://localhost:8000/api-docs) | Interactive API docs (when server is running) |ng components:

- **Frontend Client** (React + TypeScript + Vite) - Port 5173
- **Main API Server** (Express.js + TypeScript + TypeORM) - Port 8000
- **User Service** (Express.js + TypeScript + Redis) - Port 3001
- **Logging Service** (Express.js + TypeScript + Redis) - Port 3002
- **Redis** (Docker container) - Port 6379
- **PostgreSQL Database** (for main server)

## 📋 Features

### Main Application
- 📰 **News Posts Management**: Create, read, update, delete news posts
- 🏷️ **Category Management**: Organize posts by categories
- 📄 **Pagination**: Efficient content browsing
- 🔍 **Genre Filtering**: Filter posts by categories
- 📱 **Responsive Design**: Mobile-friendly interface

### Authentication & Authorization
- 🔐 **User Registration**: Create new user accounts
- 🔑 **User Login**: JWT-based authentication
- 👤 **User Profile Management**: View user information
- 🛡️ **Protected Routes**: Secure access to authenticated features

### Documentation & Monitoring
- 📚 **Swagger API Documentation**: Auto-generated API docs at `/api-docs`
- 📊 **Health Checks**: Service status monitoring
- 📝 **Comprehensive Logging**: Activity tracking across services

## 🚀 Quick Start

### Prerequisites

- Node.js (v18 or higher)
- Docker and Docker Compose
- PostgreSQL database
- Git

### Installation & Setup

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd nd-6-hw17
   ```

2. **Install dependencies for all services**
   ```bash
   # Install all dependencies
   npm run deps:client
   npm run deps:server
   npm run deps:user-service
   npm run deps:logging-service
   ```

3. **Start Redis container**
   ```bash
   npm run start:redis
   ```

4. **Setup database and seed data**
   ```bash
   npm run setup:db
   ```

5. **Start all services in development mode**
   ```bash
   # Terminal 1 - Start microservices
   npm run start:user-service
   npm run start:logging-service
   
   # Terminal 2 - Start main server
   cd server && npm run dev:server
   
   # Terminal 3 - Start client
   cd client && npm run dev:client
   ```

6. **Access the application**
   - Frontend: http://localhost:5173
   - API Documentation: http://localhost:8000/api-docs
   - Main API: http://localhost:8000/api
   - User Service: http://localhost:3001
   - Logging Service: http://localhost:3002

## 📁 Project Structure

```
nd-6-hw17/
├── client/                     # React frontend application
│   ├── src/
│   │   ├── components/         # React components
│   │   ├── pages/             # Page components
│   │   ├── contexts/          # React contexts (Auth, etc.)
│   │   ├── services/          # API service layer
│   │   └── types/             # TypeScript type definitions
│   ├── public/                # Static assets
│   └── package.json
│
├── server/                     # Main API server
│   ├── src/                   # Source code
│   ├── controller/            # Express controllers
│   ├── entities/              # TypeORM entities
│   ├── routes/                # API routes
│   ├── services/              # Business logic services
│   ├── middleware/            # Express middleware
│   ├── scripts/               # Database and utility scripts
│   ├── migrations/            # Database migrations
│   └── package.json
│
├── microservices/             # Microservices
│   ├── user-service/          # User management service
│   │   ├── src/
│   │   │   ├── controllers/   # User controllers
│   │   │   ├── services/      # User business logic
│   │   │   ├── models/        # Data models
│   │   │   └── types/         # Type definitions
│   │   └── package.json
│   │
│   └── logging-service/       # Logging and monitoring service
│       ├── src/
│       │   ├── controllers/   # Logging controllers
│       │   ├── services/      # Logging business logic
│       │   └── logs/          # Log files
│       └── package.json
│
├── docker-compose.microservices.yml  # Docker Compose config
└── package.json                      # Root package.json with scripts
```

## 🔧 Available Scripts

### Root Level Scripts

| Script | Description |
|--------|-------------|
| `npm run deps:client` | Install client dependencies |
| `npm run deps:server` | Install server dependencies |
| `npm run deps:user-service` | Install user service dependencies |
| `npm run deps:logging-service` | Install logging service dependencies |
| `npm run start:redis` | Start Redis Docker container |
| `npm run stop:redis` | Stop and remove Redis container |
| `npm run start:user-service` | Build and start user service |
| `npm run start:logging-service` | Build and start logging service |
| `npm run start:microservices` | Start both microservices |
| `npm run setup:db` | Setup database and load demo data |
| `npm run start:app` | Build and start entire application |

### Development Scripts

| Service | Script | Description |
|---------|--------|-------------|
| Client | `npm run dev:client` | Start development server |
| Server | `npm run dev:server` | Start development server with hot reload |
| Server | `npm run swagger:generate` | Generate Swagger documentation |

## 🌐 API Documentation

### Main API Endpoints (Port 8000)

The main server provides RESTful API endpoints for news posts management:

- **GET** `/api/newsposts` - Get all posts (with pagination and filtering)
- **GET** `/api/newsposts/:id` - Get post by ID
- **POST** `/api/newsposts` - Create new post (authenticated)
- **PUT** `/api/newsposts/:id` - Update post (authenticated)
- **DELETE** `/api/newsposts/:id` - Delete post (authenticated)
- **GET** `/api/newsposts/categories` - Get all categories

### User Service Endpoints (Port 3001)

- **POST** `/users/create` - Register new user
- **POST** `/users/login` - Login user
- **GET** `/users/:id` - Get user by ID
- **GET** `/health` - Health check

### Logging Service Endpoints (Port 3002)

- **GET** `/health` - Health check
- **GET** `/logs` - Get application logs (if implemented)

### Interactive API Documentation

Visit http://localhost:8000/api-docs for interactive Swagger documentation with:
- Complete API specification
- Request/response examples
- Try-it-out functionality
- Schema definitions

## 🔐 Authentication

The application uses JWT (JSON Web Tokens) for authentication:

1. **Registration**: POST to `/users/create` with email, password, and confirmPassword
2. **Login**: POST to `/users/login` with email and password
3. **Token Usage**: Include JWT token in Authorization header: `Bearer <token>`
4. **Token Storage**: Tokens are stored in localStorage on the client side

### Example Authentication Flow

```javascript
// Register
const registerResponse = await fetch('http://localhost:3001/users/create', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    email: 'user@example.com',
    password: 'password123',
    confirmPassword: 'password123'
  })
});

// Login
const loginResponse = await fetch('http://localhost:3001/users/login', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    email: 'user@example.com',
    password: 'password123'
  })
});

const { token } = await loginResponse.json();

// Use token for authenticated requests
const postsResponse = await fetch('http://localhost:8000/api/newsposts', {
  headers: { 'Authorization': `Bearer ${token}` }
});
```

## 🐳 Docker Support

The application includes Docker configuration for easy deployment:

```bash
# Start Redis with Docker
npm run start:redis

# Or use Docker Compose for microservices
docker-compose -f docker-compose.microservices.yml up
```

## 🗄️ Database

The application uses PostgreSQL as the main database with TypeORM for data management:

### Entities
- **Post**: News posts with title, content, category, etc.
- **User**: User accounts (managed by User Service)
- **Category**: Post categories

### Database Scripts
- `npm run setup:db` - Initialize database and run migrations
- `npm run load:demo-data` - Load sample data
- `npm run typeorm:migration:run` - Run database migrations
- `npm run typeorm:migration:generate` - Generate new migration

## 🔍 Development

### Adding New Features

1. **Frontend Components**: Add to `client/src/components/`
2. **API Endpoints**: Add to `server/routes/` and `server/controller/`
3. **Database Changes**: Create migrations in `server/migrations/`
4. **Microservices**: Extend services in `microservices/`

### Code Structure Guidelines

- Use TypeScript for type safety
- Follow RESTful API conventions
- Implement proper error handling
- Add comprehensive logging
- Write unit tests (when test framework is set up)

## 🚨 Troubleshooting

### Common Issues

1. **Port Already in Use**
   ```bash
   # Check what's using the port
   netstat -an | findstr "3001\|3002\|8000\|5173\|6379"
   
   # Kill processes if needed
   taskkill /F /PID <process_id>
   ```

2. **Redis Connection Issues**
   ```bash
   # Restart Redis container
   npm run stop:redis
   npm run start:redis
   ```

3. **Database Connection Issues**
   - Check PostgreSQL is running
   - Verify database credentials in `.env` files
   - Run `npm run setup:db` to reinitialize

4. **Build Errors**
   ```bash
   # Clean install all dependencies
   rm -rf node_modules client/node_modules server/node_modules
   npm run deps:client
   npm run deps:server
   npm run deps:user-service
   npm run deps:logging-service
   ```

### Logs and Debugging

- **Server logs**: Check `server/logs/` directory
- **Microservice logs**: Check respective service directories
- **Client logs**: Browser developer console
- **Redis logs**: Docker container logs

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature-name`
3. Make your changes
4. Add tests if applicable
5. Commit your changes: `git commit -am 'Add feature'`
6. Push to the branch: `git push origin feature-name`
7. Submit a pull request

## 📄 License

This project is licensed under the ISC License.

## � Documentation

| Document | Description |
|----------|-------------|
| [API Documentation](API_DOCUMENTATION.md) | Complete API reference with examples |
| [Developer Guide](DEVELOPER_GUIDE.md) | Detailed development setup and patterns |
| [Deployment Guide](DEPLOYMENT_GUIDE.md) | Production deployment instructions |
| [Swagger Documentation](http://localhost:8000/api-docs) | Interactive API docs (when server is running) |

## �🔗 Links

- [GitHub Repository](https://github.com/yhorodniy/nd-6-hw17)
- [Interactive API Documentation](http://localhost:8000/api-docs) (when server is running)
- [Frontend Application](http://localhost:5173) (when client is running)

---

For more detailed information about specific components, check the documentation files above and individual service directories.
