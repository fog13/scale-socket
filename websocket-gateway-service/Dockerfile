# ---- Build Stage ----
# Use a Maven image with JDK 17 to build the application
FROM maven:3.9.6-eclipse-temurin-17 AS build

# Set the working directory inside the container for the build stage
WORKDIR /build

# Copy all POM files first, maintaining the multi-module directory structure relative to /build
# This allows Maven to resolve parent/sibling relationships correctly.
COPY pom.xml .
COPY common-library/pom.xml ./common-library/
COPY websocket-gateway-service/pom.xml ./websocket-gateway-service/
COPY message-processor-service/pom.xml ./message-processor-service/

# Download dependencies for all modules based on the POMs copied above.
# Running from the root (/build) lets Maven understand the whole project structure.
RUN mvn dependency:go-offline -B

# Copy source code for all modules, maintaining structure
# Needed for compilation if modules depend on each other's code (not just JARs)
# Copy common library source (uncomment if common-library has code beyond simple DTOs used during compile)
# COPY common-library/src ./common-library/src
# Copy service sources
COPY websocket-gateway-service/src ./websocket-gateway-service/src
COPY message-processor-service/src ./message-processor-service/src

# Build specifically the websocket-gateway-service JAR, including required dependencies (-am)
RUN mvn package -DskipTests -pl websocket-gateway-service -am

# ---- Package Stage ----
# Use a slim JRE image for the final application runtime for smaller size
FROM eclipse-temurin:17-jre-jammy

# Set the working directory for the runtime stage
WORKDIR /app

# Copy only the built JAR file from this module's target directory in the build stage
COPY --from=build /build/websocket-gateway-service/target/websocket-gateway-service-*.jar app.jar

# Expose the port the application runs on (informational purpose for Docker)
EXPOSE 8080

# Command to run the application when the container starts
ENTRYPOINT ["java", "-jar", "/app/app.jar"]