# =========================================================
# STAGE 1: Compilación de la aplicación (Build Stage)
# =========================================================
FROM node:20-alpine AS build
WORKDIR /app

# Optimización de capas: Copiar archivos de configuración primero
COPY package*.json ./

# Instalación de dependencias de desarrollo y producción
RUN npm install

# Copiar el resto del código fuente del proyecto React
COPY . .

# Compilar el proyecto para generar los archivos estáticos de producción
RUN npm run build

# =========================================================
# STAGE 2: Entorno de ejecución seguro (Production Stage)
# =========================================================
FROM nginx:alpine AS runner

# Configuración de Seguridad No-Root: Permisos para directorios de Nginx
RUN touch /var/run/nginx.pid && \
    chown -R nginx:nginx /var/run/nginx.pid /var/cache/nginx /var/log/nginx /etc/nginx

# Copiar los archivos compilados desde la etapa anterior
# Nota: Vite/React por defecto compila en 'dist'. Si tu proyecto usa 'build', cambia /app/dist por /app/build
COPY --from=build /app/dist /usr/share/nginx/html

# Cambiar de forma explícita al usuario sin privilegios
USER nginx

# Puerto expuesto de manera interna en el contenedor
EXPOSE 80

# Comando para iniciar el servidor Nginx en primer plano
CMD ["nginx", "-g", "daemon off;"]