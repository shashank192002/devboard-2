# Stage 1 — build the app into static files
FROM node:23-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci --legacy-peer-deps
COPY . .
RUN npm run build

# Stage 2 — small image that only serves the build
FROM node:23-alpine
WORKDIR /app

# Create non-root user
RUN addgroup -g 1001 -S nodejs && adduser -S nodejs -u 1001

# Install vite and copy files with proper ownership
RUN npm install -g vite@5 && npm cache clean --force
COPY --from=build --chown=nodejs:nodejs /app/dist ./dist
COPY --chown=nodejs:nodejs vite.preview.config.js ./vite.config.js

# Switch to non-root user before CMD
USER nodejs

EXPOSE 5173

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget --quiet --tries=1 --spider http://localhost:5173 || exit 1

CMD ["vite", "preview", "--host", "0.0.0.0", "--port", "5173"]
