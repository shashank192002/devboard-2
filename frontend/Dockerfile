# Stage 1 — build the app into static files
FROM dhi.io/node:24-alpine AS build
SHELL ["/bin/ash", "-c"]
WORKDIR /app
COPY package*.json ./
RUN npm ci --legacy-peer-deps
COPY . .
RUN npm run build

# Stage 2 — small image that only serves the build
FROM dhi.io/node:24-alpine
SHELL ["/bin/ash", "-c"]
WORKDIR /app

# Create non-root user
RUN addgroup -g 1001 nodejs && adduser -D -S -u 1001 -G nodejs nodejs

# Copy only dist and config, install vite from npm install (not global)
COPY --from=build --chown=nodejs:nodejs /app/dist ./dist
COPY --chown=nodejs:nodejs package*.json ./
COPY --chown=nodejs:nodejs vite.preview.config.js ./vite.config.js

# Install only production dependencies
RUN npm ci --legacy-peer-deps --omit=dev && npm cache clean --force

# Switch to non-root user before CMD
USER nodejs

EXPOSE 5173

# Use node instead of wget for health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node -e "require('http').get('http://localhost:5173', (r) => {if (r.statusCode !== 200) throw new Error(r.statusCode)})"

CMD ["./node_modules/.bin/vite", "preview", "--host", "0.0.0.0", "--port", "5173"]

