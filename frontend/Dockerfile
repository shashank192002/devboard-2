
# Stage 1 — build the app into static files
FROM dhi.io/node:22-debian AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci --legacy-peer-deps
COPY . .
RUN npm run build

# Stage 2 — small image that only serves the build (no nginx)
FROM dhi.io/node:22-debian
WORKDIR /app
RUN npm install -g vite@5 && npm cache clean --force

# Create non-root user
RUN addgroup -g 1001 -S nodejs && adduser -S nodejs -u 1001

COPY --from=build /app/dist ./dist
COPY vite.preview.config.js ./vite.config.js

# Set ownership to non-root user
RUN chown -R nodejs:nodejs /app

USER nodejs

EXPOSE 4173
CMD ["vite", "preview", "--host", "0.0.0.0", "--port", "4173"]

