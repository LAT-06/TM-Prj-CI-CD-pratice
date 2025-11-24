# Teams Management Platform - DevOps Practice Project

> This is a DevOps practice project forked from [caphefalumi/TM-Project](https://github.com/caphefalumi/TM-Project).
> Original project by Hồ Quốc Khánh, Đặng Duy Toàn, and Phan Lê Minh Hiếu.

## Project Overview

A full-stack Teams Management Platform with complete CI/CD pipeline implementation using Docker, Docker Hub, and GitHub Actions for automated deployment to VPS.

## Technology Stack

### Frontend
- Vue.js 3 with Vite
- Vuetify (Material Design)
- Pinia (State Management)
- Vue Router

### Backend
- Bun Runtime
- Express.js
- MongoDB with Mongoose
- JWT Authentication
- Google OAuth 2.0

### DevOps & Infrastructure
- Docker & Docker Compose
- GitHub Actions (CI/CD)
- Docker Hub (Container Registry)
- Nginx (Reverse Proxy & Static File Serving)
- VPS Deployment

## CI/CD Pipeline

This project implements a complete CI/CD pipeline with the following workflow:

```
Code Push → GitHub Actions → Build Docker Images → Push to Docker Hub → SSH to VPS → Pull Images → Deploy
```

### Pipeline Features

- **Automated Building**: Docker images built on every push to main branch
- **Container Registry**: Images stored on Docker Hub with version tags
- **Automated Deployment**: Zero-downtime deployment to VPS
- **Health Checks**: Automatic container health monitoring
- **Rollback Support**: Tagged images for easy rollback

### Deployment Architecture

```
GitHub Repository
    ↓
GitHub Actions (CI/CD)
    ↓
Docker Hub (Image Registry)
    ↓
VPS Server
    ├── Frontend (Nginx:80)
    ├── Backend (Bun:3000)
    └── MongoDB (27017)
```

## Quick Start

### Local Development

#### Prerequisites
- Bun v1.0+
- MongoDB
- Docker & Docker Compose (optional)

#### Installation

1. Clone the repository:
   ```bash
   git clone https://github.com/caphefalumi/TM-Project
   cd TM-Project
   ```

2. Install dependencies:
   ```bash
   bun install
   cd client && bun install
   cd ../server && bun install
   ```

3. Setup environment variables:
   - Create `.env` in `server/` folder
   - Create `.env` in `client/` folder

4. Run locally:
   ```bash
   # Run both frontend and backend
   bun run app
   
   # Or run separately
   bun run dev      # Frontend only
   bun run server   # Backend only
   ```

5. Visit `http://localhost:5173`

### Docker Development

```bash
# Build and run with Docker Compose
docker-compose up -d --build

# View logs
docker-compose logs -f

# Stop containers
docker-compose down
```

Frontend: `http://localhost:8080`
Backend: `http://localhost:3000`

## Production Deployment

### VPS Setup

1. **Install Docker on VPS:**
   ```bash
   curl -fsSL https://get.docker.com -o get-docker.sh
   sh get-docker.sh
   ```

2. **Install Docker Compose:**
   ```bash
   curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
   chmod +x /usr/local/bin/docker-compose
   ```

3. **Create project directory:**
   ```bash
   mkdir -p /var/www/TM-Project
   cd /var/www/TM-Project
   ```

4. **Create `docker-compose.prod.yml`:**
   ```yaml
   version: '3.8'
   
   services:
     frontend:
       image: your-username/tm-frontend:latest
       ports:
         - "80:80"
       restart: unless-stopped
       depends_on:
         - backend
   
     backend:
       image: your-username/tm-backend:latest
       ports:
         - "3000:3000"
       environment:
         - DB_CONNECTION_STRING=mongodb://database:27017/tm-project
         - NODE_ENV=production
       restart: unless-stopped
       depends_on:
         - database
   
     database:
       image: mongo:latest
       ports:
         - "27017:27017"
       volumes:
         - mongo-data:/data/db
       restart: unless-stopped
   
   volumes:
     mongo-data:
   ```

### GitHub Actions Setup

1. **Create Docker Hub Account:**
   - Sign up at https://hub.docker.com
   - Create repositories: `tm-frontend` and `tm-backend`
   - Generate Access Token

2. **Add GitHub Secrets:**
   
   Go to `Settings → Secrets and variables → Actions` and add:
   
   - `DOCKER_USERNAME`: Your Docker Hub username
   - `DOCKER_TOKEN`: Docker Hub access token
   - `VPS_HOST`: VPS IP address
   - `VPS_USERNAME`: SSH username (usually `root`)
   - `VPS_PASSWORD`: SSH password (or use `VPS_SSH_KEY`)

3. **Workflow automatically triggers on push to main branch**

### Manual Deployment

If you need to deploy manually:

```bash
# SSH to VPS
ssh root@your-vps-ip

# Navigate to project
cd /var/www/TM-Project

# Pull latest images
docker-compose -f docker-compose.prod.yml pull

# Restart containers
docker-compose -f docker-compose.prod.yml up -d --force-recreate

# View logs
docker-compose -f docker-compose.prod.yml logs -f
```

## Project Structure

```
TM-Project/
├── .github/
│   └── workflows/
│       └── deploy.yml          # CI/CD pipeline
├── client/                     # Vue.js frontend
│   ├── src/
│   ├── nginx.conf             # Nginx config for SPA
│   └── package.json
├── server/                     # Express.js backend
│   ├── controllers/
│   ├── models/
│   ├── routes/
│   └── package.json
├── Dockerfile.frontend         # Frontend Docker config
├── Dockerfile.backend          # Backend Docker config
├── docker-compose.yml          # Local development
└── docker-compose.prod.yml     # Production deployment
```

## Features

### User Management
- Email/Password authentication
- Google OAuth 2.0 integration
- JWT-based sessions
- Role-based access control

### Team Management
- Create and manage teams
- Invite team members
- Role assignments (Admin, Member)
- Team activity tracking

### Task Management
- Create, assign, and track tasks
- Priority levels and status tracking
- Task comments and attachments
- Sprint management

### Notifications
- Real-time notifications
- Team announcements
- Activity feed

## Environment Variables

### Frontend (.env)
```env
VITE_API_BASE_URL=http://localhost:3000
```

### Backend (.env)
```env
MONGODB_URI=mongodb://localhost:27017/tm-project
JWT_SECRET=your-secret-key
PORT=3000
NODE_ENV=development
GOOGLE_CLIENT_ID=your-google-client-id
GOOGLE_CLIENT_SECRET=your-google-client-secret
```

## Development Workflow

1. **Make changes** to code
2. **Commit** changes:
   ```bash
   git add .
   git commit -m "description"
   ```
3. **Push** to GitHub:
   ```bash
   git push origin main
   ```
4. **GitHub Actions** automatically:
   - Builds Docker images
   - Pushes to Docker Hub
   - Deploys to VPS
5. **Website updates** automatically (5-8 minutes)

## Monitoring & Maintenance

### View Logs
```bash
# All services
docker-compose -f docker-compose.prod.yml logs -f

# Specific service
docker-compose -f docker-compose.prod.yml logs -f frontend
docker-compose -f docker-compose.prod.yml logs -f backend
```

### Container Status
```bash
docker-compose -f docker-compose.prod.yml ps
```

### Resource Usage
```bash
docker stats
```

### Cleanup
```bash
# Remove unused images
docker image prune -af

# Full cleanup
docker system prune -af
```

## Troubleshooting

### Containers not starting
```bash
docker-compose -f docker-compose.prod.yml logs
```

### Port conflicts
```bash
# Check ports in use
sudo lsof -i :80
sudo lsof -i :3000
```

### Database connection issues
```bash
# Check MongoDB logs
docker-compose -f docker-compose.prod.yml logs database
```

## Credits

### Original Project
- **Repository**: [caphefalumi/TM-Project](https://github.com/caphefalumi/TM-Project)
- **Authors**:
  - Hồ Quốc Khánh ([khanhkelvin08122006@gmail.com](mailto:khanhkelvin08122006@gmail.com))
  - Đặng Duy Toàn ([dangduytoan13l@gmail.com](mailto:dangduytoan13l@gmail.com))
  - Phan Lê Minh Hiếu ([hphan5283@gmail.com](mailto:hphan5283@gmail.com))

### DevOps Implementation
This fork implements CI/CD pipeline and containerization for educational purposes.

## License

This project is for educational purposes. Please refer to the original repository for licensing information.

## Support

For issues related to:
- **Original features**: Check [original repository](https://github.com/caphefalumi/TM-Project)
- **DevOps/Deployment**: Open an issue in this repository
