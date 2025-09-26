# Frequently Asked Questions (FAQ)

Common questions and solutions for the News Posts Microservices Application.

## 🚀 Getting Started

### Q: What are the system requirements?
**A:** 
- Node.js 18 or higher
- PostgreSQL database
- Docker (for Redis)
- At least 4GB RAM
- 2GB free disk space

### Q: How long does initial setup take?
**A:** 
- First-time setup: 10-15 minutes
- Subsequent starts: 2-3 minutes
- Using pre-built images: 1-2 minutes

### Q: Can I run this on Windows/Mac/Linux?
**A:** Yes, the application is cross-platform and works on all major operating systems.

## 🔧 Installation & Setup

### Q: npm install fails with permission errors
**A:** 
```bash
# On macOS/Linux, avoid using sudo with npm
# Instead, configure npm to use a different directory
npm config set prefix ~/.npm-global

# On Windows, run as administrator or use:
npm install --no-optional
```

### Q: Database connection fails
**A:** 
1. Ensure PostgreSQL is running
2. Check credentials in `.env` files
3. Verify database exists:
   ```bash
   psql -h localhost -U your_user -d your_database
   ```
4. Run database setup:
   ```bash
   npm run setup:db
   ```

### Q: Redis connection errors
**A:** 
```bash
# Check if Redis container is running
docker ps | grep redis

# Restart Redis if needed
npm run stop:redis
npm run start:redis

# Check Redis connectivity
docker exec -it redis-microservices redis-cli ping
```

### Q: Port already in use errors
**A:** 
```bash
# Find what's using the port
netstat -tulpn | grep :8000  # Linux/Mac
netstat -an | findstr :8000  # Windows

# Kill the process
kill -9 <PID>  # Linux/Mac
taskkill /F /PID <PID>  # Windows
```

## 🏗️ Development

### Q: How do I add a new API endpoint?
**A:** 
1. Add route in `server/routes/`
2. Create controller method in `server/controller/`
3. Add service logic in `server/services/`
4. Update Swagger comments
5. Add TypeScript types in `server/types/`

### Q: How do I add a new React component?
**A:** 
1. Create component directory in `client/src/components/`
2. Create `.tsx` and `.css` files
3. Export from component directory
4. Import and use in parent components

### Q: How do I modify the database schema?
**A:** 
1. Update entity in `server/entities/`
2. Generate migration:
   ```bash
   npm run typeorm:migration:generate -- MigrationName
   ```
3. Review and run migration:
   ```bash
   npm run typeorm:migration:run
   ```

### Q: Hot reload not working
**A:** 
1. Check if using correct dev scripts (`dev:server`, `dev:client`)
2. Verify file watchers aren't being blocked by antivirus
3. Try restarting the dev server
4. Clear node_modules and reinstall if persistent

## 🔐 Authentication

### Q: JWT token expires too quickly
**A:** 
Modify `JWT_EXPIRES_IN` in `.env` files:
```env
JWT_EXPIRES_IN=30d  # 30 days
JWT_EXPIRES_IN=24h  # 24 hours
JWT_EXPIRES_IN=7d   # 7 days (default)
```

### Q: Login returns "Invalid email or password" but credentials are correct
**A:** 
1. Check if user exists in database
2. Verify User Service is running on port 3001
3. Check network requests in browser dev tools
4. Ensure client API URL is correct (port 3001, not 3011)

### Q: Can't access protected routes
**A:** 
1. Verify JWT token is stored in localStorage
2. Check Authorization header in network requests
3. Ensure token hasn't expired
4. Try logging out and back in

## 📊 Data & Database

### Q: How do I reset the database?
**A:** 
```bash
# Reset database and reload seed data
npm run setup:db

# Or manually:
npm run typeorm:schema:drop
npm run typeorm:migration:run
npm run load:demo-data
```

### Q: How do I add sample data?
**A:** 
Modify `server/scripts/seedDatabase.ts` and run:
```bash
npm run load:demo-data
```

### Q: Database migrations fail
**A:** 
1. Check database connection
2. Ensure proper permissions
3. Run migrations one by one:
   ```bash
   npm run typeorm:migration:run
   ```
4. If still failing, drop and recreate:
   ```bash
   npm run typeorm:schema:drop
   npm run typeorm:migration:run
   ```

## 🐳 Docker & Deployment

### Q: Docker containers won't start
**A:** 
1. Check Docker is running
2. Verify port availability
3. Check container logs:
   ```bash
   docker logs redis-microservices
   ```
4. Remove and recreate containers:
   ```bash
   npm run stop:redis
   npm run start:redis
   ```

### Q: How do I deploy to production?
**A:** 
See [DEPLOYMENT_GUIDE.md](DEPLOYMENT_GUIDE.md) for comprehensive deployment instructions.

### Q: Services can't communicate in Docker
**A:** 
1. Ensure all services are on same Docker network
2. Use service names instead of localhost in Docker Compose
3. Check firewall settings
4. Verify environment variables in containers

## 🔍 Debugging

### Q: How do I debug API issues?
**A:** 
1. Check server logs in terminal
2. Use Swagger UI at http://localhost:8000/api-docs
3. Monitor network requests in browser dev tools
4. Check `server/logs/` directory for detailed logs

### Q: Frontend not loading data
**A:** 
1. Check browser console for errors
2. Verify API endpoints are responding
3. Check CORS configuration
4. Ensure authentication tokens are valid

### Q: Microservices not communicating
**A:** 
1. Verify Redis is running and accessible
2. Check service URLs in environment variables
3. Monitor Redis pub/sub channels:
   ```bash
   docker exec -it redis-microservices redis-cli monitor
   ```

## 📱 Frontend Issues

### Q: Vite dev server won't start
**A:** 
1. Clear Vite cache:
   ```bash
   rm -rf client/.vite
   ```
2. Reinstall dependencies:
   ```bash
   cd client && npm install
   ```
3. Check port 5173 availability
4. Try different port:
   ```bash
   npm run dev:client -- --port 3000
   ```

### Q: Build fails with TypeScript errors
**A:** 
1. Check TypeScript configuration
2. Ensure all imports have proper types
3. Update type definitions:
   ```bash
   npm install --save-dev @types/node @types/react
   ```

### Q: Styles not loading correctly
**A:** 
1. Check CSS imports in components
2. Verify file paths are correct
3. Clear browser cache
4. Check for CSS conflicts

## 🚨 Performance

### Q: Application is slow
**A:** 
1. Check database query performance
2. Monitor memory usage of Node.js processes
3. Verify Redis is properly connected
4. Consider implementing caching
5. Check network latency between services

### Q: Database queries are slow
**A:** 
1. Add database indexes for frequently queried fields
2. Optimize query patterns in services
3. Consider pagination for large datasets
4. Monitor database performance logs

## 🔒 Security

### Q: How secure is this application?
**A:** 
The application includes basic security measures:
- JWT authentication
- Input validation
- CORS configuration
- Helmet.js security headers

For production, consider additional measures in [DEPLOYMENT_GUIDE.md](DEPLOYMENT_GUIDE.md).

### Q: How do I change JWT secret?
**A:** 
1. Update `JWT_SECRET` in all `.env` files
2. Restart all services
3. Note: This will invalidate all existing tokens

## 📖 Documentation

### Q: Where can I find API documentation?
**A:** 
- Interactive Swagger UI: http://localhost:8000/api-docs
- Comprehensive guide: [API_DOCUMENTATION.md](API_DOCUMENTATION.md)

### Q: How do I contribute to the project?
**A:** 
1. Fork the repository
2. Create a feature branch
3. Follow coding standards in [DEVELOPER_GUIDE.md](DEVELOPER_GUIDE.md)
4. Add tests if applicable
5. Submit a pull request

### Q: Where can I report bugs?
**A:** 
Create an issue on the GitHub repository with:
- Steps to reproduce
- Expected vs actual behavior
- System information
- Relevant log outputs

## 🆘 Still Need Help?

If your question isn't answered here:

1. Check the comprehensive [README.md](README.md)
2. Review [DEVELOPER_GUIDE.md](DEVELOPER_GUIDE.md) for detailed development info
3. Consult [API_DOCUMENTATION.md](API_DOCUMENTATION.md) for API details
4. Look at [DEPLOYMENT_GUIDE.md](DEPLOYMENT_GUIDE.md) for deployment issues
5. Search existing GitHub issues
6. Create a new GitHub issue with detailed information

---

**Pro Tip**: Most issues can be resolved by ensuring all services are running correctly and checking the logs for specific error messages.
