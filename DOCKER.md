# Docker Setup for GitDiagram

This document explains how to run GitDiagram using Docker in both development and production environments.

## Quick Start

### Development
```bash
# Start development environment with hot-reload
docker-compose -f docker-compose.dev.yml up

# Or use the default (development) configuration
docker-compose up
```

### Production
```bash
# Start production environment
docker-compose up

# With production overrides
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up
```

## Configuration Files

- `docker-compose.yml` - Production configuration (default)
- `docker-compose.dev.yml` - Development configuration with hot-reload
- `docker-compose.prod.yml` - Production overrides and optimizations
- `Dockerfile` - Multi-stage build for both dev and production

## Services

### Frontend (Next.js)
- **Development**: Port 3000 with hot-reload
- **Production**: Port 3000 with optimized build
- **Health Check**: `/api/health` endpoint
- **Resources**: 512MB limit, 256MB reservation

### Backend (FastAPI)
- **Port**: 8000
- **Health Check**: `/health` endpoint
- **Resources**: 1GB limit, 512MB reservation

## Environment Variables

Create a `.env` file in the project root with required variables:

```env
# Database
DATABASE_URL=your_database_url

# GitHub
GITHUB_TOKEN=your_github_token

# AI Service
OPENAI_API_KEY=your_openai_key

# Analytics
NEXT_PUBLIC_POSTHOG_KEY=your_posthog_key
NEXT_PUBLIC_POSTHOG_HOST=your_posthog_host
```

## Production Deployment

### Using Docker Compose
```bash
# Build and start production services
docker-compose up -d

# View logs
docker-compose logs -f

# Stop services
docker-compose down
```

### Using Docker Swarm
```bash
# Initialize swarm
docker swarm init

# Deploy stack
docker stack deploy -c docker-compose.yml gitdiagram

# View services
docker service ls
```

## Health Checks

Both services include health checks:
- Frontend: Checks `/api/health` endpoint
- Backend: Checks `/health` endpoint

## Resource Management

Production configuration includes resource limits:
- **Frontend**: 512MB memory, 0.5 CPU
- **Backend**: 1GB memory, 1.0 CPU

## Logging

Production setup includes log rotation:
- Maximum file size: 10MB
- Maximum files: 3
- Driver: json-file

## Security

- Services run as non-root users
- Isolated network configuration
- Environment variable validation
- Resource limits to prevent resource exhaustion

## Troubleshooting

### Check service health
```bash
# Check all services
docker-compose ps

# Check specific service logs
docker-compose logs frontend
docker-compose logs api
```

### Rebuild services
```bash
# Rebuild and restart
docker-compose up --build

# Rebuild specific service
docker-compose up --build frontend
```

### Clean up
```bash
# Stop and remove containers
docker-compose down

# Remove volumes and networks
docker-compose down -v --remove-orphans
```
