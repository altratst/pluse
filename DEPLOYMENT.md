# Deployment Guide

## Prerequisites

- Docker & Docker Compose installed
- Node.js 16+ (for local development)
- PostgreSQL 12+ (for local development)
- SSH access to production server
- Domain name configured

## Local Development

### Quick Start

1. **Clone and setup**
```bash
git clone https://github.com/altratst/pluse.git
cd pluse
npm install
cp .env.example .env
```

2. **Run with Docker Compose**
```bash
docker-compose up --build
```

3. **Access the API**
```
http://localhost:5000/health
```

### Without Docker

1. **Start PostgreSQL**
```bash
psql -U postgres
CREATE DATABASE pluse_health;
```

2. **Configure environment**
```bash
cp .env.example .env
# Edit .env with your local settings
```

3. **Run migrations and seed**
```bash
npm run migrate
npm run seed
```

4. **Start server**
```bash
npm run dev
```

---

## Production Deployment

### Option 1: Docker Compose on Linux Server

1. **SSH to server**
```bash
ssh user@your-server.com
```

2. **Clone repository**
```bash
git clone https://github.com/altratst/pluse.git
cd pluse
```

3. **Configure production environment**
```bash
cp .env.example .env
# Edit .env with production settings
```

4. **Deploy**
```bash
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

### Option 2: Using GitHub Actions

1. **Add secrets to GitHub**
   - `DEPLOY_KEY`: SSH private key
   - `DEPLOY_USER`: SSH username
   - `DEPLOY_SERVER`: Server IP/hostname

2. **Push to main branch**
```bash
git push origin main
```

3. **GitHub Actions will automatically**
   - Run tests
   - Build Docker image
   - Push to registry
   - Deploy to production

### Option 3: Using Kubernetes

1. **Create namespace**
```bash
kubectl create namespace pluse
```

2. **Deploy database**
```bash
kubectl apply -f k8s/postgres-deployment.yml -n pluse
kubectl apply -f k8s/postgres-service.yml -n pluse
```

3. **Deploy application**
```bash
kubectl apply -f k8s/app-deployment.yml -n pluse
kubectl apply -f k8s/app-service.yml -n pluse
```

4. **Setup ingress**
```bash
kubectl apply -f k8s/ingress.yml -n pluse
```

---

## Production Configuration

### Environment Variables

```env
NODE_ENV=production
PORT=5000

# Database
DB_HOST=postgres
DB_PORT=5432
DB_NAME=pluse_health
DB_USER=postgres
DB_PASSWORD=strong_password_here

# Security
JWT_SECRET=your_super_secret_key_change_this
JWT_EXPIRE=7d

# API
API_URL=https://api.pluse.example.com
CORS_ORIGIN=https://pluse.example.com

# Logging
LOG_LEVEL=info
```

### Nginx Reverse Proxy

```nginx
upstream pluse_api {
  server app:5000;
}

server {
  listen 80;
  server_name api.pluse.example.com;
  
  location / {
    proxy_pass http://pluse_api;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
  }
}
```

### SSL/TLS with Let's Encrypt

```bash
# Install Certbot
sudo apt-get install certbot python3-certbot-nginx

# Generate certificate
sudo certbot certonly --nginx -d api.pluse.example.com

# Auto-renewal (should be automatic)
sudo systemctl enable certbot.timer
```

---

## Database Backup

### Automatic Backup Script

Create `backup.sh`:
```bash
#!/bin/bash
BACKUP_DIR="/backups/pluse"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p $BACKUP_DIR

docker exec pluse_postgres pg_dump -U postgres pluse_health | \
  gzip > $BACKUP_DIR/pluse_health_$DATE.sql.gz

# Keep only last 30 days
find $BACKUP_DIR -name "*.sql.gz" -mtime +30 -delete
```

### Schedule with Cron

```bash
0 2 * * * /path/to/backup.sh
```

---

## Monitoring and Logging

### Health Check Endpoint

```bash
curl http://localhost:5000/health
```

### Docker Logs

```bash
docker-compose logs -f app
docker-compose logs -f postgres
```

### PostgreSQL Logs

```bash
docker exec pluse_postgres tail -f /var/log/postgresql/postgresql.log
```

---

## Scaling

### Horizontal Scaling with Load Balancer

1. **Deploy multiple instances**
```bash
docker-compose up -d --scale app=3
```

2. **Configure load balancer (HAProxy)**
```
frontend api
  bind *:80
  default_backend app_servers

backend app_servers
  server app1 app1:5000
  server app2 app2:5000
  server app3 app3:5000
```

### Database Connection Pooling

Already configured in `config/database.js`:
```javascript
max: 20,  // Maximum connections
idleTimeoutMillis: 30000
```

---

## Troubleshooting

### Container won't start
```bash
docker-compose logs app
docker-compose down -v
docker-compose up --build
```

### Database connection error
```bash
docker-compose exec postgres psql -U postgres -c "SELECT 1"
```

### Port already in use
```bash
lsof -i :5000
kill -9 <PID>
```

### Performance issues
```bash
# Check container stats
docker stats

# Check database queries
docker exec pluse_postgres psql -U postgres pluse_health -c "\dt"
```

---

## Security Checklist

- [ ] Change default database password
- [ ] Change JWT secret
- [ ] Enable HTTPS/SSL
- [ ] Configure firewall rules
- [ ] Setup backup schedule
- [ ] Enable database backups
- [ ] Configure monitoring alerts
- [ ] Setup log aggregation
- [ ] Enable audit logging
- [ ] Regular security updates

---

## Rollback Plan

If deployment fails:

```bash
# Revert to previous version
git revert <commit-hash>
git push origin main

# Or manually rollback
docker-compose down
git checkout <previous-tag>
docker-compose up -d --build
```

---

## Support

For deployment issues, check:
1. Docker logs: `docker-compose logs`
2. Application logs: Check `/logs` directory
3. Database logs: `docker exec pluse_postgres tail -f /var/log/postgresql/`
4. GitHub Issues: Report problems on the repository
