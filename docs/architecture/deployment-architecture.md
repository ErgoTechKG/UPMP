# Deployment Architecture

## Deployment Strategy

**Frontend Deployment:**
- **Platform:** Alibaba Cloud CDN + OSS
- **Build Command:** `nx build web --prod`
- **Output Directory:** `dist/apps/web`
- **CDN/Edge:** Alibaba Cloud CDN with China acceleration

**Backend Deployment:**
- **Platform:** Alibaba Cloud Container Service (ACK)
- **Build Command:** `nx build api --prod`
- **Deployment Method:** Docker containers orchestrated by Kubernetes

## CI/CD Pipeline
```yaml