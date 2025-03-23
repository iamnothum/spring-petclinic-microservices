# Use official OpenJDK base image
FROM openjdk:17-jdk-slim

# Set working directory
WORKDIR /app

# Copy built JAR file from Jenkins workspace into the container
COPY target/*.jar app.jar

# Copy SQL files from JAR to /docker-entrypoint-initdb.d/ for MySQL initialization
RUN mkdir /docker-entrypoint-initdb.d && \
    jar -xvf app.jar -C /docker-entrypoint-initdb.d db/mysql/schema.sql && \
    jar -xvf app.jar -C /docker-entrypoint-initdb.d db/mysql/data.sql

# Expose service port
EXPOSE 8080

# Set environment variables for database connection (Kubernetes will override these)
ENV DATABASE_URL=jdbc:mysql://mysql:3306/petclinic
ENV DATABASE_USER=root
ENV DATABASE_PASSWORD=petclinic

# Run the Spring Boot application
CMD ["java", "-jar", "app.jar"]
