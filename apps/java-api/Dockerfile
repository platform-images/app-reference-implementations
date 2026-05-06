# syntax=docker/dockerfile:1
# Build stage — Maven available
FROM ghcr.io/platform-images/openjdk-base:1.0.1@sha256:f7ce7ecd44931f50d034c97ff1bca943dcb541861c4e65475feee56ade3c4c7a AS builder
USER root
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline -q
COPY src/ ./src/
RUN mvn package -DskipTests -q

# Runtime stage — no shell, no Maven
FROM ghcr.io/platform-images/openjdk-base:1.0.1-distroless@sha256:85645adc9f50f598a1b99b3f715774c8e0ff24b1d58ba4a8a6e5bd34047166f1
ARG BUILD_DATE
ARG GIT_REVISION
LABEL org.opencontainers.image.source="https://github.com/platform-images/app-reference-implementations"
LABEL org.opencontainers.image.licenses="Apache-2.0"
LABEL org.opencontainers.image.created="${BUILD_DATE}"
LABEL org.opencontainers.image.revision="${GIT_REVISION}"
WORKDIR /app
COPY --from=builder /app/target/java-api-1.0.0.jar app.jar
EXPOSE 8080
CMD ["-jar", "/app/app.jar"]
