# syntax=docker/dockerfile:1
# Build stage — Maven available
FROM ghcr.io/platform-images/openjdk-base:1.0.0@sha256:39dda08361dcc9d9253a7e9904e92438511b5e15de3e633ac5edc3dcbda81871 AS builder
USER root
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline -q
COPY src/ ./src/
RUN mvn package -DskipTests -q

# Runtime stage — no shell, no Maven
FROM ghcr.io/platform-images/openjdk-base:1.0.0-distroless@sha256:9d9a8a2fb49ca6539426a16a47a9eaa149ce855f2c96fe2e3a103a40e2f93dc3
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
