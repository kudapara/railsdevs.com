# Production Deployment Guide

This guide covers deploying the RailsDevs application to production using Docker Compose.

## Prerequisites

- Docker and Docker Compose installed on your production server
- Domain name configured to point to your server
- Basic understanding of Docker and server administration

## Quick Start

### 1. Clone and Setup

```bash
# Clone the repository
git clone <your-repo-url>
cd railsdevs.com

# Copy environment variables
cp env.production.example .env

# Edit the .env file with your actual values
nano .env
```

### 2. Configure Environment Variables

Update the `.env` file with your production values:

```bash
# Rails Configuration
RAILS_MASTER_KEY=your_actual_rails_master_key
RAILSDEVS_DATABASE_PASSWORD=your_secure_password

# Application Host (replace with your actual domain)
HOST=your-domain.com

# Performance Tuning (optional)
RAILS_MAX_THREADS=5
WEB_CONCURRENCY=2

# Scaling Configuration
WEB_REPLICAS=2
WORKER_REPLICAS=2
```
### 3. Deploy

```bash
# Build and start all services
docker-compose -f docker-compose.prod.yml up -d

# Run database migrations
docker-compose -f docker-compose.prod.yml exec web bundle exec rails db:migrate

# Seed the database (optional)
docker-compose -f docker-compose.prod.yml exec web bundle exec rails db:seed
```

## Production Features

### Automatic HTTPS
- Caddy automatically obtains SSL certificates from Let's Encrypt
- Automatic HTTP to HTTPS redirect
- Certificate auto-renewal

### High Availability
- Multiple web server replicas (configurable via `WEB_REPLICAS`)
- Multiple background worker replicas (configurable via `WORKER_REPLICAS`)
- Health checks for all services
- Automatic restart on failure

### Security
- Rate limiting for API endpoints and login attempts
- Security headers (HSTS, XSS protection, etc.)
- Non-root user in containers
- Resource limits to prevent resource exhaustion

### Monitoring
- Health checks for all services
- Structured JSON logging
- Resource usage monitoring

### Backup
- Automated daily database backups
- 30-day backup retention
- Compressed backups to save space

## Scaling

### Horizontal Scaling

```bash
# Scale web servers
docker-compose -f docker-compose.prod.yml up -d --scale web=4

# Scale background workers
docker-compose -f docker-compose.prod.yml up -d --scale worker=3
```

### Vertical Scaling

Update resource limits in `docker-compose.prod.yml`:

```yaml
deploy:
  resources:
    limits:
      memory: 2G      # Increase memory
      cpus: '2.0'     # Increase CPU
```

## Maintenance

### Database Backups

Backups are automatically created daily. To manually create a backup:

```bash
docker-compose -f docker-compose.prod.yml exec db_backup /backup.sh
```

### View Logs

```bash
# All services
docker-compose -f docker-compose.prod.yml logs -f

# Specific service
docker-compose -f docker-compose.prod.yml logs -f web
docker-compose -f docker-compose.prod.yml logs -f worker
```

### Update Application

```bash
# Pull latest changes
git pull origin main

# Rebuild and restart
docker-compose -f docker-compose.prod.yml build
docker-compose -f docker-compose.prod.yml up -d

# Run migrations if needed
docker-compose -f docker-compose.prod.yml exec web bundle exec rails db:migrate
```

### Restart Services

```bash
# Restart all services
docker-compose -f docker-compose.prod.yml restart

# Restart specific service
docker-compose -f docker-compose.prod.yml restart web
```

## Troubleshooting

### Common Issues

1. **SSL Certificate Issues**
   - Ensure your domain points to the server
   - Check firewall settings (ports 80 and 443 must be open)
   - Verify DNS propagation

2. **Database Connection Issues**
   - Check if PostgreSQL is running: `docker-compose -f docker-compose.prod.yml ps postgres`
   - Verify database credentials in `.env`

3. **High Memory Usage**
   - Adjust resource limits in `docker-compose.prod.yml`
   - Monitor with: `docker stats`

4. **Background Jobs Not Processing**
   - Check Sidekiq worker logs: `docker-compose -f docker-compose.prod.yml logs worker`
   - Verify Redis connection

### Useful Commands

```bash
# Check service status
docker-compose -f docker-compose.prod.yml ps

# View resource usage
docker stats

# Execute Rails console
docker-compose -f docker-compose.prod.yml exec web bundle exec rails console

# Check database
docker-compose -f docker-compose.prod.yml exec postgres psql -U railsdevs -d railsdevs_production

# View Caddy logs
docker-compose -f docker-compose.prod.yml logs caddy
```

## Security Considerations

1. **Firewall**: Only expose ports 80 and 443
2. **Updates**: Regularly update Docker images and system packages
3. **Secrets**: Use a secrets management system for sensitive data
4. **Monitoring**: Set up monitoring and alerting
5. **Backups**: Test backup restoration regularly

## Performance Optimization

1. **Database**: Configure PostgreSQL for your workload
2. **Caching**: Optimize Redis configuration
3. **Assets**: Consider using a CDN for static assets
4. **Monitoring**: Use application performance monitoring (APM)

## Support

For issues and questions:
- Check the logs first: `docker-compose -f docker-compose.prod.yml logs`
- Review this documentation
- Check the main README for development setup
