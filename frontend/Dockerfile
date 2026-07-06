
FROM dhi.io/node@sha256:0f52e8901797d837f6586d7ece8a524f68ee07f536f6bf130ec5dd0d864013dd AS build

WORKDIR /app

COPY package*.json ./

RUN npm install --legacy-peer-deps

# Install vite explicitly if not already in package.json
RUN npm list vite > /dev/null 2>&1 || npm install --no-save vite@5.4.11

COPY . .

RUN npm run build

# Prune to production-only dependencies before copying node_modules into the
# runtime stage (vite itself is only needed here if you run `vite preview`
# in prod — see stage 2 note below).

# Stage 2 — distroless runtime (minimal, secure)
# Same digest-pinning approach as above — resolve with:
#   docker pull dhi.io/node:26 && docker inspect --format='{{index .RepoDigests 0}}' dhi.io/node:26
FROM dhi.io/node@sha256:08c6347fc51b41180f06c90a2496320c0da495a7768a4cf5403ed541dcaf93dd

WORKDIR /app

# Copy built files from builder
COPY --from=build /app/dist ./dist

# Copy pruned (production-only) node_modules from builder.
# NOTE: if vite is a devDependency, `npm prune --omit=dev` will remove it,
# and `vite preview` below will fail. Either move vite to "dependencies",
# or better, serve /dist with a lightweight static server instead of
# `vite preview` (which isn't meant for production use anyway).
COPY --from=build /app/node_modules ./node_modules

COPY vite.preview.config.js ./vite.config.js

EXPOSE 4173

# Use exec form for proper signal handling
ENTRYPOINT ["node", "--no-warnings"]
CMD ["./node_modules/.bin/vite", "preview", "--host", "0.0.0.0", "--port", "4173"]
