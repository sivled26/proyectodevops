# ===== STAGE 1: BUILD =====
# Usamos alpine en la compilación para mantener el proceso ligero
FROM maven:3.9.6-eclipse-temurin-17-alpine AS build

WORKDIR /app

# Copiamos primero el pom.xml para aprovechar la caché de capas de Docker
COPY pom.xml .
RUN mvn dependency:go-offline -B

# Copiamos el resto del código fuente y compilamos
COPY . .
RUN mvn clean package -DskipTests

# ===== STAGE 2: RUNTIME =====
# Imagen ultra-liviana de producción (Optimización de capas exigida en la pauta)
FROM eclipse-temurin:17-jre-alpine

WORKDIR /app

# Crear grupo y usuario no-root en formato Alpine (Seguridad de menor privilegio)
RUN addgroup -g 1001 -S springgroup && \
    adduser -u 1001 -S springuser -G springgroup

# Copiamos el archivo .jar generado en la etapa de build
COPY --from=build /app/target/*.jar app.jar

# Asignamos la propiedad del directorio al usuario seguro
RUN chown -R springuser:springgroup /app

# Cambiamos al usuario no-root
USER springuser

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]