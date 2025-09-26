# Quick Start Guide

Get the News Posts Microservices Application running in under 5 minutes!

## ⚡ Prerequisites

- Node.js 18+ installed
- Docker installed
- PostgreSQL database available

## 🚀 Quick Setup (Development)

```bash
# 1. Clone and navigate
git clone <repository-url>
cd nd-6-hw17

# 2. Install all dependencies (one command)
npm run deps:client && npm run deps:server && npm run deps:user-service && npm run deps:logging-service

# 3. Start Redis
npm run start:redis

# 4. Setup database (make sure PostgreSQL is running)
npm run setup:db

# 5. Start all services (use separate terminals)
# Terminal 1: Main server
cd server && npm run dev:server

# Terminal 2: User service  
cd microservices/user-service && npm run dev

# Terminal 3: Logging service
cd microservices/logging-service && npm run dev

# Terminal 4: Frontend client
cd client && npm run dev:client
```

## 🌐 Access Points

Once all services are running:

- **Frontend**: http://localhost:5173
- **API Docs**: http://localhost:8000/api-docs
- **Main API**: http://localhost:8000/api
- **User Service**: http://localhost:3001
- **Logging Service**: http://localhost:3002

## 🧪 Test the Application

1. **Open Frontend**: Go to http://localhost:5173
2. **Register Account**: Click "Sign Up" and create an account
3. **Login**: Sign in with your credentials
4. **Create Post**: Add a new news post
5. **View Posts**: Browse the news feed

## 🛠️ Environment Setup

Create these `.env` files:

### `server/.env`
```env
DB_HOST=localhost
DB_PORT=5432
DB_USERNAME=your_db_user
DB_PASSWORD=your_db_password
DB_DATABASE=news_posts_dev
JWT_SECRET=your-jwt-secret-key
NODE_ENV=development
```

### `microservices/user-service/.env`
```env
PORT=3001
JWT_SECRET=your-jwt-secret-key
REDIS_HOST=localhost
REDIS_PORT=6379
NODE_ENV=development
```

### `microservices/logging-service/.env`
```env
PORT=3002
REDIS_HOST=localhost
REDIS_PORT=6379
NODE_ENV=development
```

## 🔧 Common Issues

### Port Already in Use
```bash
# Check ports
netstat -an | findstr "3001\|3002\|8000\|5173\|6379"

# Kill process if needed
taskkill /F /PID <process_id>
```

### Redis Issues
```bash
# Restart Redis
npm run stop:redis
npm run start:redis
```

### Database Issues
```bash
# Reset database
npm run setup:db
```

## 📖 Need More Help?

- **Full Documentation**: [README.md](README.md)
- **API Reference**: [API_DOCUMENTATION.md](API_DOCUMENTATION.md)
- **Development Guide**: [DEVELOPER_GUIDE.md](DEVELOPER_GUIDE.md)
- **Deployment**: [DEPLOYMENT_GUIDE.md](DEPLOYMENT_GUIDE.md)

## 🎯 Next Steps

1. **Explore API**: Visit http://localhost:8000/api-docs
2. **Create Content**: Add news posts through the web interface
3. **Check Logs**: View application logs in terminal outputs
4. **Customize**: Modify components in `client/src/components/`
5. **Extend API**: Add new endpoints in `server/routes/`

Happy coding! 🎉
