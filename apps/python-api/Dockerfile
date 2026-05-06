# syntax=docker/dockerfile:1
# Build stage — pip available
FROM ghcr.io/platform-images/python-base:1.0.1@sha256:dbba5a79e019934a6c303ddb61b38b0a0c12576cea056aba749a0a163416ad0d AS builder
USER root
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir --target=/app/packages -r requirements.txt

# Runtime stage — no shell, no pip
FROM ghcr.io/platform-images/python-base:1.0.1-distroless@sha256:3630d3661957ca78b4d24ee9724c3d7fdefdb07ee992216054f6327aa5faf91b
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
