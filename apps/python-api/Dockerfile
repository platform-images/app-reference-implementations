# syntax=docker/dockerfile:1
# Build stage — pip available
FROM ghcr.io/platform-images/python-base:1.0.0@sha256:b67d1344e2bfaf0c79fe44d95ad4c8fe365aa23bf7a27445eabab5fe0b033b67 AS builder
USER root
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir --target=/app/packages -r requirements.txt

# Runtime stage — no shell, no pip
FROM ghcr.io/platform-images/python-base:1.0.0-distroless@sha256:0f3bad0f20f5bcd8d89a9382001a2244164cfd8c4b1e94710317a7f3c2b0811d
ARG BUILD_DATE
ARG GIT_REVISION
LABEL org.opencontainers.image.source="https://github.com/platform-images/app-reference-implementations"
LABEL org.opencontainers.image.licenses="Apache-2.0"
LABEL org.opencontainers.image.created="${BUILD_DATE}"
LABEL org.opencontainers.image.revision="${GIT_REVISION}"
WORKDIR /app
COPY --from=builder /app/packages ./packages
COPY src/ ./src/
ENV PYTHONPATH=/app/packages
EXPOSE 8000
CMD ["/app/src/main.py"]
