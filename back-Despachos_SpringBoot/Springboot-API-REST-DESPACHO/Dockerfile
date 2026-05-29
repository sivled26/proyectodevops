# Etapa 1: Build con Maven
FROM maven:3.9.6-eclipse-temurin-17 AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

# Etapa 2: Runtime con OpenJDK
FROM eclipse-temurin:17-jre-alpine

WORKDIR /app
COPY --from=build /app/target/Springboot-API-REST-DESPACHO-0.0.1-SNAPSHOT.jar app.jar

# Exponer el puerto configurado en application.properties
EXPOSE 8081

# Variables de entorno para la BD (se inyectan en docker-compose o ECS)
ENV DB_ENDPOINT=localhost \
    DB_PORT=3306 \
    DB_NAME=rednorte_db \
    DB_USERNAME=root \
    DB_PASSWORD=secret

CMD ["java", "-jar", "app.jar"]
