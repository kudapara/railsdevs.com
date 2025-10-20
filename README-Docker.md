# Docker Production Setup

This document explains how to run the RailsDevs application in production using Docker and Docker Compose.

## Prerequisites

- Docker and Docker Compose installed
- Environment variables configured (see Environment Variables section)

## Environment Variables

Create a `.env` file in the project root with the following variables:

```bash
# Rails Master Key (required for production)
RAILS_MASTER_KEY=your_rails_master_key_here

# Database Configuration
RAILSDEVS_DATABASE_PASSWORD=your_secure_database_password_here

# Application Host (replace with your actual domain)
HOST=your-domain.com
```
## Production Deployment

### 1. Build and Start Services

```bash
# Build the application image
docker-compose build

# Start all services
docker-compose up -d
```

### 2. Run Database Migrations

```bash
# Run migrations
docker-compose exec web bundle exec rails db:migrate

# Seed the database (optional)
docker-compose exec web bundle exec rails db:seed
```

### 3. Verify Services

```bash
# Check service status
docker-compose ps

# View logs
docker-compose logs -f web
docker-compose logs -f worker
```

## Services

The Docker Compose setup includes:

- **web**: Rails application server (Puma)
- **worker**: Sidekiq background job processor
- **postgres**: PostgreSQL database
- **redis**: Redis for Sidekiq and caching
- **caddy**: Reverse proxy with automatic HTTPS

## Development

For development, use the override file:

```bash
# Start development environment
docker-compose -f docker-compose.yml -f docker-compose.override.yml up
```

## Production Considerations

### Security

1. **Change default passwords**: Update `RAILSDEVS_DATABASE_PASSWORD` with a strong password
2. **SSL/TLS**: Caddy automatically handles SSL certificates via Let's Encrypt
3. **Firewall**: Ensure only necessary ports are exposed
4. **Secrets**: Store sensitive data in environment variables or secret management systems
5. **Rate limiting**: Configured for API endpoints and login attempts

### Performance

1. **Resource limits**: Add resource constraints to docker-compose.yml
2. **Database optimization**: Configure PostgreSQL for production workloads
3. **Caching**: Configure Redis for optimal caching performance
4. **Asset serving**: Consider using a CDN for static assets

### Monitoring

1. **Health checks**: Services include health checks for monitoring
2. **Logging**: Configure centralized logging
3. **Metrics**: Set up application monitoring (Scout APM is already configured)

### Backup

1. **Database backups**: Implement regular PostgreSQL backups
2. **File storage**: Backup any uploaded files in the storage directory
3. **Configuration**: Backup environment variables and configuration files

## Troubleshooting

### Common Issues

1. **Database connection**: Ensure PostgreSQL is running and accessible
2. **Asset compilation**: Check that Node.js dependencies are installed
3. **Permissions**: Ensure proper file permissions for the app user

### Useful Commands

```bash
# View service logs
docker-compose logs [service_name]

# Execute commands in containers
docker-compose exec web bundle exec rails console
docker-compose exec postgres psql -U railsdevs -d railsdevs_production

# Restart services
docker-compose restart [service_name]

# Scale workers
docker-compose up -d --scale worker=3
```

## Scaling

To scale the application:

```bash
# Scale web servers
docker-compose up -d --scale web=3

# Scale background workers
docker-compose up -d --scale worker=2
```

Note: When scaling, Caddy will automatically load balance across multiple web instances using round-robin.
