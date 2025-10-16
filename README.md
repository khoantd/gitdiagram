[![Image](./docs/readme_img.png "GitDiagram Front Page")](https://gitdiagram.com/)

![License](https://img.shields.io/badge/license-MIT-blue.svg)
[![Kofi](https://img.shields.io/badge/Kofi-F16061.svg?logo=ko-fi&logoColor=white)](https://ko-fi.com/ahmedkhaleel2004)

# GitDiagram

Turn any GitHub repository into an interactive diagram for visualization in seconds.

You can also replace `hub` with `diagram` in any Github URL to access its diagram.

## 🚀 Features

- 👀 **Instant Visualization**: Convert any GitHub repository structure into a system design / architecture diagram
- 🎨 **Interactivity**: Click on components to navigate directly to source files and relevant directories
- ⚡ **Fast Generation**: Powered by OpenAI o4-mini for quick and accurate diagrams
- 🔄 **Customization**: Modify and regenerate diagrams with custom instructions
- 🌐 **API Access**: Public API available for integration (WIP)

## ⚙️ Tech Stack

- **Frontend**: Next.js, TypeScript, Tailwind CSS, ShadCN
- **Backend**: FastAPI, Python, Server Actions
- **Database**: PostgreSQL (with Drizzle ORM)
- **AI**: OpenAI o4-mini
- **Deployment**: Vercel (Frontend), EC2 (Backend)
- **CI/CD**: GitHub Actions
- **Analytics**: PostHog, Api-Analytics

## 🤔 About

I created this because I wanted to contribute to open-source projects but quickly realized their codebases are too massive for me to dig through manually, so this helps me get started - but it's definitely got many more use cases!

Given any public (or private!) GitHub repository it generates diagrams in Mermaid.js with OpenAI's o4-mini! (Previously Claude 3.5 Sonnet)

I extract information from the file tree and README for details and interactivity (you can click components to be taken to relevant files and directories)

Most of what you might call the "processing" of this app is done with prompt engineering - see `/backend/app/prompts.py`. This basically extracts and pipelines data and analysis for a larger action workflow, ending in the diagram code.

## 🔒 How to diagram private repositories

You can simply click on "Private Repos" in the header and follow the instructions by providing a GitHub personal access token with the `repo` scope.

You can also self-host this app locally (backend separated as well!) with the steps below.

## 🛠️ Self-hosting / Local Development

### Quick Start with Docker

1. Clone the repository

```bash
git clone https://github.com/ahmedkhaleel2004/gitdiagram.git
cd gitdiagram
```

2. Set up environment variables

```bash
cp .env.example .env
```

Edit the `.env` file with your required API keys:
- `OPENAI_API_KEY` - Your OpenAI API key
- `GITHUB_TOKEN` - Optional GitHub personal access token for private repos
- `DATABASE_URL` - PostgreSQL connection string
- `NEXT_PUBLIC_POSTHOG_KEY` - PostHog analytics key (optional)

3. Start all services with Docker Compose

```bash
# Development mode (with hot-reload)
docker-compose -f docker-compose.dev.yml up

# Production mode
docker-compose up
```

4. Access the application

- Frontend: http://localhost:3000
- Backend API: http://localhost:8000
- API Documentation: http://localhost:8000/docs

### Manual Development Setup

1. Install dependencies

```bash
pnpm i
```

2. Run backend

```bash
docker-compose up --build -d
```

Logs available at `docker-compose logs -f`
The FastAPI server will be available at `localhost:8000`

3. Start local database

```bash
chmod +x start-database.sh
./start-database.sh
```

When prompted to generate a random password, input yes.
The Postgres database will start in a container at `localhost:5432`

4. Initialize the database schema

```bash
pnpm db:push
```

You can view and interact with the database using `pnpm db:studio`

5. Run Frontend

```bash
pnpm dev
```

You can now access the website at `localhost:3000` and edit the rate limits defined in `backend/app/routers/generate.py` in the generate function decorator.

## 🚀 Production Deployment

### Docker Deployment

#### Option 1: Docker Compose (Recommended)

1. **Prepare your server**
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Docker and Docker Compose
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER

# Install Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

2. **Deploy the application**
```bash
# Clone repository
git clone https://github.com/ahmedkhaleel2004/gitdiagram.git
cd gitdiagram

# Set up environment
cp .env.example .env
# Edit .env with your production values

# Start production services
docker-compose up -d

# Check service health
docker-compose ps
docker-compose logs -f
```

3. **Configure reverse proxy (Nginx)**
```nginx
server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /api/ {
        proxy_pass http://localhost:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

#### Option 2: Docker Swarm

```bash
# Initialize swarm
docker swarm init

# Deploy stack
docker stack deploy -c docker-compose.yml gitdiagram

# Scale services
docker service scale gitdiagram_frontend=3 gitdiagram_api=2
```

### Cloud Deployment

#### Vercel (Frontend) + Railway/Render (Backend)

1. **Frontend on Vercel**
   - Connect your GitHub repository
   - Set environment variables in Vercel dashboard
   - Deploy automatically on push

2. **Backend on Railway/Render**
   - Connect your repository
   - Set build command: `docker build -f backend/Dockerfile .`
   - Configure environment variables
   - Deploy

#### AWS ECS/Fargate

1. **Create ECS Cluster**
```bash
aws ecs create-cluster --cluster-name gitdiagram
```

2. **Build and push Docker images**
```bash
# Build images
docker build -t gitdiagram-frontend .
docker build -t gitdiagram-api ./backend

# Tag for ECR
docker tag gitdiagram-frontend:latest your-account.dkr.ecr.region.amazonaws.com/gitdiagram-frontend:latest
docker tag gitdiagram-api:latest your-account.dkr.ecr.region.amazonaws.com/gitdiagram-api:latest

# Push to ECR
docker push your-account.dkr.ecr.region.amazonaws.com/gitdiagram-frontend:latest
docker push your-account.dkr.ecr.region.amazonaws.com/gitdiagram-api:latest
```

3. **Create ECS Task Definitions and Services**

### Environment Variables

Create a `.env` file with the following variables:

```env
# Database
DATABASE_URL=postgresql://username:password@host:port/database

# GitHub Integration
GITHUB_TOKEN=your_github_personal_access_token

# AI Service
OPENAI_API_KEY=your_openai_api_key

# Analytics (Optional)
NEXT_PUBLIC_POSTHOG_KEY=your_posthog_key
NEXT_PUBLIC_POSTHOG_HOST=https://app.posthog.com

# Environment
NODE_ENV=production
ENVIRONMENT=production
```

### Health Monitoring

The application includes health checks:

- **Frontend**: `GET /api/health`
- **Backend**: `GET /health`

Monitor with:
```bash
# Check service health
curl http://localhost:3000/api/health
curl http://localhost:8000/health

# Docker health status
docker-compose ps
```

### Scaling and Performance

#### Horizontal Scaling
```bash
# Scale frontend instances
docker-compose up --scale frontend=3

# Scale backend instances
docker-compose up --scale api=2
```

#### Load Balancing
Use Nginx or Traefik for load balancing multiple instances.

#### Database Optimization
- Use connection pooling
- Configure PostgreSQL for production
- Set up database backups

### Security Considerations

1. **Environment Variables**
   - Never commit `.env` files
   - Use secrets management in production
   - Rotate API keys regularly

2. **Network Security**
   - Use HTTPS in production
   - Configure firewall rules
   - Implement rate limiting

3. **Container Security**
   - Run containers as non-root users
   - Keep base images updated
   - Scan for vulnerabilities

### Backup and Recovery

#### Database Backup
```bash
# Create backup
docker exec gitdiagram_postgres pg_dump -U username database_name > backup.sql

# Restore backup
docker exec -i gitdiagram_postgres psql -U username database_name < backup.sql
```

#### Application Backup
```bash
# Backup application data
docker-compose exec frontend tar -czf /backup/app-data.tar.gz /app/data
```

### Troubleshooting

#### Common Issues

1. **Services won't start**
```bash
# Check logs
docker-compose logs -f

# Check resource usage
docker stats

# Restart services
docker-compose restart
```

2. **Database connection issues**
```bash
# Check database status
docker-compose exec postgres psql -U username -d database_name -c "SELECT 1;"
```

3. **Memory issues**
```bash
# Check memory usage
docker stats

# Adjust resource limits in docker-compose.yml
```

#### Logs and Monitoring
```bash
# View all logs
docker-compose logs -f

# View specific service logs
docker-compose logs -f frontend
docker-compose logs -f api

# Follow logs in real-time
docker-compose logs -f --tail=100
```

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## Acknowledgements

Shoutout to [Romain Courtois](https://github.com/cyclotruc)'s [Gitingest](https://gitingest.com/) for inspiration and styling

## 📈 Rate Limits

I am currently hosting it for free with no rate limits though this is somewhat likely to change in the future.

<!-- If you would like to bypass these, self-hosting instructions are provided. I also plan on adding an input for your own Anthropic API key.

Diagram generation:

- 1 request per minute
- 5 requests per day -->

## 🤔 Future Steps

- Implement font-awesome icons in diagram
- Implement an embedded feature like star-history.com but for diagrams. The diagram could also be updated progressively as commits are made.
