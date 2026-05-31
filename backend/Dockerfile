# syntax=docker/dockerfile:1

# ---------- Stage 1: build ----------
# Resolve dependencies first (cached layer) so source-only changes don't re-download Maven deps.
FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /app

# Dependencies first — this layer is reused unless pom.xml changes.
COPY pom.xml .
RUN mvn -B dependency:go-offline

# Then the source, then build.
COPY src ./src
RUN mvn -B clean package -DskipTests

# ---------- Stage 2: runtime ----------
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app

# Create a dedicated non-root user/group and run as it.
RUN addgroup -S app && adduser -S -G app app

# Copy only the built artifact from the build stage.
COPY --from=build --chown=app:app /app/target/*.jar app.jar

USER app
EXPOSE 8080



# Adjust the endpoint if /api-docs is secured; this matches devops/scripts/deploy.sh.
HEALTHCHECK --interval=30s --timeout=5s --start-period=40s --retries=3 \
  CMD wget -qO- http://localhost:8080/api-docs >/dev/null 2>&1 || exit 1

# exec form so the JVM is PID 1 and receives stop signals.
ENTRYPOINT ["sh", "-c", "exec java $JAVA_OPTS -jar /app/app.jar"]
