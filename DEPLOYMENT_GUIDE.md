# Deployment Guide

This guide provides comprehensive instructions for deploying the News Posts Microservices Application in various environments.

## 🚀 Deployment Options

1. **Development Deployment** - Local development with hot reload
2. **Production Deployment** - Optimized build for production
3. **Docker Deployment** - Containerized deployment
4. **Cloud Deployment** - Cloud platform deployment (AWS, Google Cloud, etc.)

## 🛠️ Development Deployment

### Prerequisites
- Node.js v18+
- PostgreSQL database
- Docker (for Redis)
- Git

### Quick Development Setup

```bash
# 1. Clone and setup
git clone <repository-url>
cd nd-6-hw17

# 2. Install all dependencies
npm run deps:client
npm run deps:server
npm run deps:user-service
npm run deps:logging-service

# 3. Setup environment files (see Environment Configuration below)

# 4. Start services
npm run start:redis
npm run setup:db

# 5. Start development servers (in separate terminals)
cd server && npm run dev:server
cd microservices/user-service && npm run dev
cd microservices/logging-service && npm run dev
cd client && npm run dev:client
```

### Development Environment Configuration

#### Server Environment (`server/.env`)
```env
# Database
DB_HOST=localhost
DB_PORT=5432
DB_USERNAME=your_db_user
DB_PASSWORD=your_db_password
DB_DATABASE=news_posts_dev

# JWT
JWT_SECRET=dev-jwt-secret-key-change-in-production
JWT_EXPIRES_IN=7d

# Server
PORT=8000
NODE_ENV=development

# Redis
REDIS_HOST=localhost
REDIS_PORT=6379

# Services
USER_SERVICE_URL=http://localhost:3001
LOGGING_SERVICE_URL=http://localhost:3002
```

#### User Service Environment (`microservices/user-service/.env`)
```env
PORT=3001
JWT_SECRET=dev-jwt-secret-key-change-in-production
JWT_EXPIRES_IN=7d
REDIS_HOST=localhost
REDIS_PORT=6379
NODE_ENV=development
```

#### Logging Service Environment (`microservices/logging-service/.env`)
```env
PORT=3002
REDIS_HOST=localhost
REDIS_PORT=6379
NODE_ENV=development
```

## 🏭 Production Deployment

### Prerequisites
- Production server (Linux recommended)
- PostgreSQL database
- Redis server
- Domain name and SSL certificate
- Reverse proxy (Nginx recommended)

### Production Build Process

```bash
# 1. Build all services
npm run prod:client
npm run prod:server
npm run prod:microservices

# 2. Setup production database
npm run setup:db

# 3. Start production services
npm run start:app
```

### Production Environment Configuration

#### Server Environment (`server/.env`)
```env
# Database
DB_HOST=your-production-db-host
DB_PORT=5432
DB_USERNAME=your_prod_db_user
DB_PASSWORD=your_secure_db_password
DB_DATABASE=news_posts_prod

# JWT - Use strong random secret
JWT_SECRET=your-super-secure-jwt-secret-256-bit-minimum
JWT_EXPIRES_IN=7d

# Server
PORT=8000
NODE_ENV=production

# Redis
REDIS_HOST=your-redis-host
REDIS_PORT=6379
REDIS_PASSWORD=your-redis-password

# Services
USER_SERVICE_URL=https://your-domain.com/user-service
LOGGING_SERVICE_URL=https://your-domain.com/logging-service

# Security
SWAGGER_ENABLED=false  # Disable Swagger in production
CORS_ORIGIN=https://your-domain.com
```

### Nginx Configuration

Create `/etc/nginx/sites-available/news-app`:

```nginx
server {
    listen 80;
    server_name your-domain.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name your-domain.com;

    ssl_certificate /path/to/your/certificate.crt;
    ssl_certificate_key /path/to/your/private.key;

    # Frontend
    location / {
        root /path/to/news-app/client/dist;
        try_files $uri $uri/ /index.html;
    }

    # Main API
    location /api/ {
        proxy_pass http://localhost:8000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }

    # User Service
    location /user-service/ {
        proxy_pass http://localhost:3001/;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Logging Service
    location /logging-service/ {
        proxy_pass http://localhost:3002/;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header Referrer-Policy "no-referrer-when-downgrade" always;
    add_header Content-Security-Policy "default-src 'self' http: https: data: blob: 'unsafe-inline'" always;
}
```

### Process Management with PM2

Install PM2:
```bash
npm install -g pm2
```

Create `ecosystem.config.js`:
```javascript
module.exports = {
  apps: [
    {
      name: 'news-server',
      script: 'server/dist/server.js',
      cwd: '/path/to/news-app',
      env: {
        NODE_ENV: 'production',
        PORT: 8000
      },
      instances: 'max',
      exec_mode: 'cluster'
    },
    {
      name: 'user-service',
      script: 'microservices/user-service/dist/server.js',
      cwd: '/path/to/news-app',
      env: {
        NODE_ENV: 'production',
        PORT: 3001
      },
      instances: 2,
      exec_mode: 'cluster'
    },
    {
      name: 'logging-service',
      script: 'microservices/logging-service/dist/server.js',
      cwd: '/path/to/news-app',
      env: {
        NODE_ENV: 'production',
        PORT: 3002
      },
      instances: 1
    }
  ]
};
```

Start services with PM2:
```bash
pm2 start ecosystem.config.js
pm2 save
pm2 startup
```

## 🐳 Docker Deployment

### Docker Compose Setup

Create `docker-compose.yml`:

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: news_posts_prod
      POSTGRES_USER: newsapp
      POSTGRES_PASSWORD: secure_password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    networks:
      - news-network

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    networks:
      - news-network

  user-service:
    build:
      context: .
      dockerfile: microservices/user-service/Dockerfile
    environment:
      NODE_ENV: production
      PORT: 3001
      REDIS_HOST: redis
      JWT_SECRET: your-production-jwt-secret
    ports:
      - "3001:3001"
    depends_on:
      - redis
    networks:
      - news-network

  logging-service:
    build:
      context: .
      dockerfile: microservices/logging-service/Dockerfile
    environment:
      NODE_ENV: production
      PORT: 3002
      REDIS_HOST: redis
    ports:
      - "3002:3002"
    depends_on:
      - redis
    networks:
      - news-network

  news-server:
    build:
      context: .
      dockerfile: server/Dockerfile
    environment:
      NODE_ENV: production
      PORT: 8000
      DB_HOST: postgres
      DB_PORT: 5432
      DB_USERNAME: newsapp
      DB_PASSWORD: secure_password
      DB_DATABASE: news_posts_prod
      REDIS_HOST: redis
      JWT_SECRET: your-production-jwt-secret
      USER_SERVICE_URL: http://user-service:3001
      LOGGING_SERVICE_URL: http://logging-service:3002
    ports:
      - "8000:8000"
    depends_on:
      - postgres
      - redis
      - user-service
      - logging-service
    networks:
      - news-network

  news-client:
    build:
      context: .
      dockerfile: client/Dockerfile
    ports:
      - "80:80"
    depends_on:
      - news-server
    networks:
      - news-network

volumes:
  postgres_data:
  redis_data:

networks:
  news-network:
    driver: bridge
```

### Individual Dockerfiles

#### Server Dockerfile (`server/Dockerfile`)
```dockerfile
FROM node:18-alpine

WORKDIR /app

# Copy package files
COPY server/package*.json ./
RUN npm ci --only=production

# Copy source code
COPY server/ ./

# Build the application
RUN npm run build

EXPOSE 8000

CMD ["npm", "run", "start:prod"]
```

#### User Service Dockerfile (`microservices/user-service/Dockerfile`)
```dockerfile
FROM node:18-alpine

WORKDIR /app

# Copy package files
COPY microservices/user-service/package*.json ./
RUN npm ci --only=production

# Copy source code
COPY microservices/user-service/ ./

# Build the application
RUN npm run build

EXPOSE 3001

CMD ["npm", "start"]
```

#### Client Dockerfile (`client/Dockerfile`)
```dockerfile
# Build stage
FROM node:18-alpine as builder

WORKDIR /app

# Copy package files
COPY client/package*.json ./
RUN npm ci

# Copy source code
COPY client/ ./

# Build the application
RUN npm run build:client

# Production stage
FROM nginx:alpine

# Copy built assets
COPY --from=builder /app/dist /usr/share/nginx/html

# Copy nginx configuration
COPY client/nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

#### Client Nginx Configuration (`client/nginx.conf`)
```nginx
server {
    listen 80;
    server_name localhost;
    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location /api/ {
        proxy_pass http://news-server:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Docker Deployment Commands

```bash
# Build and start all services
docker-compose up -d --build

# View logs
docker-compose logs -f

# Scale services
docker-compose up -d --scale user-service=3

# Stop all services
docker-compose down

# Remove volumes (careful - this deletes data)
docker-compose down -v
```

## ☁️ Cloud Deployment

### AWS Deployment

#### Using AWS ECS (Elastic Container Service)

1. **Push images to ECR**:
```bash
# Build and tag images
docker build -t news-server ./server
docker build -t user-service ./microservices/user-service
docker build -t logging-service ./microservices/logging-service
docker build -t news-client ./client

# Push to ECR
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 123456789012.dkr.ecr.us-east-1.amazonaws.com
docker tag news-server:latest 123456789012.dkr.ecr.us-east-1.amazonaws.com/news-server:latest
docker push 123456789012.dkr.ecr.us-east-1.amazonaws.com/news-server:latest
```

2. **Create ECS Task Definition** (`task-definition.json`):
```json
{
  "family": "news-app",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "1024",
  "memory": "2048",
  "executionRoleArn": "arn:aws:iam::123456789012:role/ecsTaskExecutionRole",
  "containerDefinitions": [
    {
      "name": "news-server",
      "image": "123456789012.dkr.ecr.us-east-1.amazonaws.com/news-server:latest",
      "portMappings": [
        {
          "containerPort": 8000,
          "protocol": "tcp"
        }
      ],
      "environment": [
        {
          "name": "NODE_ENV",
          "value": "production"
        }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/news-app",
          "awslogs-region": "us-east-1",
          "awslogs-stream-prefix": "ecs"
        }
      }
    }
  ]
}
```

#### Using AWS RDS for PostgreSQL
```bash
# Create RDS instance
aws rds create-db-instance \
  --db-instance-identifier news-app-db \
  --db-instance-class db.t3.micro \
  --engine postgres \
  --master-username newsapp \
  --master-user-password YourSecurePassword \
  --allocated-storage 20
```

#### Using AWS ElastiCache for Redis
```bash
# Create ElastiCache cluster
aws elasticache create-cache-cluster \
  --cache-cluster-id news-app-redis \
  --cache-node-type cache.t3.micro \
  --engine redis \
  --num-cache-nodes 1
```

### Google Cloud Platform Deployment

#### Using Google Cloud Run

1. **Build and push to Container Registry**:
```bash
# Build images
gcloud builds submit --tag gcr.io/PROJECT-ID/news-server ./server
gcloud builds submit --tag gcr.io/PROJECT-ID/user-service ./microservices/user-service
```

2. **Deploy to Cloud Run**:
```bash
# Deploy services
gcloud run deploy news-server \
  --image gcr.io/PROJECT-ID/news-server \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated

gcloud run deploy user-service \
  --image gcr.io/PROJECT-ID/user-service \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated
```

## 🔒 Security Considerations

### Environment Variables Security
- Use secret management services (AWS Secrets Manager, Azure Key Vault)
- Never commit `.env` files to version control
- Rotate JWT secrets regularly
- Use strong database passwords

### Network Security
- Configure firewall rules
- Use VPC/subnet isolation
- Enable SSL/TLS for all communications
- Implement rate limiting

### Application Security
- Keep dependencies updated
- Use security headers (helmet.js)
- Validate all inputs
- Implement proper CORS policies

## 📊 Monitoring and Logging

### Application Monitoring
- Use PM2 monitoring for Node.js processes
- Implement health checks for all services
- Set up log aggregation (ELK stack, Fluentd)
- Monitor database performance

### Alerting Setup
```javascript
// Example health check endpoint
app.get('/health', async (req, res) => {
  try {
    await database.query('SELECT 1');
    await redis.ping();
    
    res.json({
      status: 'healthy',
      timestamp: new Date().toISOString(),
      services: {
        database: 'connected',
        redis: 'connected'
      }
    });
  } catch (error) {
    res.status(503).json({
      status: 'unhealthy',
      error: error.message
    });
  }
});
```

## 🚀 Continuous Deployment

### GitHub Actions Example (`.github/workflows/deploy.yml`)
```yaml
name: Deploy to Production

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v2
    
    - name: Setup Node.js
      uses: actions/setup-node@v2
      with:
        node-version: '18'
    
    - name: Install dependencies
      run: |
        npm run deps:client
        npm run deps:server
        npm run deps:user-service
        npm run deps:logging-service
    
    - name: Build applications
      run: |
        npm run prod:client
        npm run prod:server
        npm run prod:microservices
    
    - name: Deploy to server
      run: |
        # Your deployment commands here
        rsync -avz --delete ./dist/ user@your-server:/path/to/app/
        ssh user@your-server 'pm2 restart all'
```

## 🔧 Troubleshooting

### Common Deployment Issues

1. **Port conflicts**:
```bash
# Check what's using ports
netstat -tulpn | grep :8000
lsof -i :8000
```

2. **Database connection issues**:
```bash
# Test database connection
psql -h localhost -U username -d database_name
```

3. **Redis connection issues**:
```bash
# Test Redis connection
redis-cli ping
```

4. **Memory issues**:
```bash
# Monitor memory usage
free -h
htop
```

5. **Log debugging**:
```bash
# View application logs
pm2 logs
docker-compose logs -f service-name
```

For more detailed troubleshooting, see the main [README](README.md) and [Developer Guide](DEVELOPER_GUIDE.md).

---

This deployment guide covers various deployment scenarios. Choose the one that best fits your infrastructure requirements and scale needs.
