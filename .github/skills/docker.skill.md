---
description: "Create and manage Dockerfiles, docker-compose configurations, and container-related settings."
name: Docker
---

# Docker Skill

## Capabilities
- **Dockerfiles** — create optimized, multi-stage build files
- **Docker Compose** — define multi-container applications
- **Image optimization** — minimize image size and layers
- **.dockerignore** — exclude unnecessary files from build context

## Best Practices
1. Use multi-stage builds to separate build and runtime.
2. Use slim/alpine base images when possible.
3. Pin base image versions (don't use `latest`).
4. Order layers from least to most frequently changing.
5. Use `.dockerignore` to exclude node_modules, .git, etc.
6. Run as non-root user in production images.
7. Use HEALTHCHECK for production containers.
8. Don't store secrets in images — use environment variables or secrets managers.

## Common Patterns
```dockerfile
# Multi-stage build pattern
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine AS runtime
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
USER node
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

## When to Use
- Containerizing an application for deployment.
- Setting up local development environments.
- Creating reproducible build environments.

