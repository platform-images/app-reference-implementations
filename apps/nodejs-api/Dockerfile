# syntax=docker/dockerfile:1
# Build stage — shell and npm available
FROM ghcr.io/platform-images/nodejs-base:1.0.0@sha256:c894dd3a20d57013311dac55e476501f5f8388d52ab99ebf1d9106447ed13668 AS builder
USER root
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev

# Runtime stage — no shell, minimal attack surface
FROM ghcr.io/platform-images/nodejs-base:1.0.0-distroless@sha256:69218ac8a47dc49cf13cfa74280d696ec4e6e8e9772181cb7b91356620398e37
ARG BUILD_DATE
ARG GIT_REVISION
LABEL org.opencontainers.image.source="https://github.com/platform-images/app-reference-implementations"
LABEL org.opencontainers.image.licenses="Apache-2.0"
LABEL org.opencontainers.image.created="${BUILD_DATE}"
LABEL org.opencontainers.image.revision="${GIT_REVISION}"
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY src/ ./src/
EXPOSE 3000
CMD ["src/index.js"]
