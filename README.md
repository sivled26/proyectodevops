# Reporte Técnico: Contenedorización, Orquestación y Despliegue de Infraestructura Cloud

Este documento detalla los fundamentos conceptuales y el procedimiento metodológico secuencial implementado para la contenerización, distribución y posterior despliegue de la aplicación. La solución automatiza el aprovisionamiento de servidores utilizando metodologías DevOps y servicios Cloud de AWS.

---

## 1. Fundamentos Teóricos (Definiciones Académicas)

* *Contenedorización*: Forma de virtualización a nivel de sistema operativo (Kernel). Permite aislar procesos garantizando que una aplicación se ejecute con sus librerías y dependencias exactas sin importar las configuraciones del hardware anfitrión.
* *Dockerfile*: Manifiesto de configuración secuencial y declarativo utilizado para compilar de forma reproducible una imagen base inmutable de un servicio.
* *Docker Compose*: Orquestador diseñado para definir y ejecutar aplicaciones multi-contenedor. Gestiona el ciclo de vida de los servicios, almacenamiento (volúmenes) y redes virtuales compartidas en un único archivo YAML.
* *Docker Hub*: Registro público/privado administrado en la nube que actúa como repositorio centralizado de imágenes de contenedores para su posterior distribución global.
* *GitHub Secrets*: Almacén cifrado de variables de entorno y credenciales integradas en repositorios que mitiga la fuga de accesos, contraseñas o tokens confidenciales en el código fuente.
* *AWS EC2 (Elastic Compute Cloud)*: Servicio de infraestructura como servicio (IaaS) de Amazon Web Services que provee capacidad de cómputo seguro a través de servidores virtuales (instancias) en la nube.
* *Security Groups*: Firewalls lógicos con estado a nivel de instancia que controlan de manera estricta el filtrado del tráfico de red entrante y saliente (Inbound/Outbound Rules).

---

## 2. Desglose del Paso a Paso del Ciclo de Despliegue

### Fase 1: Control de Versiones y Resguardo de Secretos
1. Inicialización del control de versiones git y sincronización con el servidor remoto de GitHub:
```bash
git init
git add .
git commit -m "infra: dockerfile and docker-compose orchestration setup"
git branch -M main
git remote add origin [https://github.com/tu-usuario/tu-repositorio.git](https://github.com/tu-usuario/tu-repositorio.git)
git push -u origin main
Almacenamiento seguro de secretos en el panel de configuración de GitHub (Settings > Secrets and Variables > Actions) para evitar exponer credenciales en texto plano dentro del código:

DOCKERHUB_USERNAME: Identificador del usuario de Docker Hub.

DOCKERHUB_TOKEN: Token de acceso personal (PAT) de Docker Hub.

AWS_SSH_KEY: Clave de acceso criptográfica codificada.

Fase 2: Compilación y Distribución a Docker Hub
Proceso de empaquetado del artefacto inmutable y su publicación en el registro remoto para la posterior extracción desde la nube:

Bash
# Autenticación segura en el registro centralizado
docker login -u TU_USUARIO_DOCKERHUB

# Compilación de la imagen asignando el tag reglamentario
docker build -t TU_USUARIO_DOCKERHUB/mi-aplicacion-web:latest .

# Distribución de la imagen hacia los servidores de Docker Hub
docker push TU_USUARIO_DOCKERHUB/mi-aplicacion-web:latest
Fase 3: Aprovisionamiento de Infraestructura Cloud (AWS EC2)
Se levantó una instancia virtual EC2 seleccionando una AMI basada en Ubuntu Server 22.04 LTS.

Se configuraron las reglas de entrada del Security Group adjunto al servidor virtual:

Puerto 22 (SSH): Permitido para la administración por consola de comandos.

Puerto 80 (HTTP): Abierto al tráfico de red global (0.0.0.0/0) para exponer la interfaz web del proyecto.

Conexión remota al servidor de AWS a través de un cliente SSH externo:

Bash
chmod 400 mi-clave-aws.pem
ssh -i "mi-clave-aws.pem" ubuntu@IP_PUBLICA_DE_TU_INSTANCIA
Fase 4: Configuración Interna del Servidor y Despliegue (Run)
Una vez dentro del prompt del servidor Ubuntu en la nube de AWS, se ejecutó de forma secuencial el siguiente bloque consolidado de comandos para preparar el entorno y correr el proyecto:

Bash
# 1. Actualización de los índices de paquetes y parches del sistema operativo
sudo apt-get update -y && sudo apt-get upgrade -y

# 2. Instalación de herramientas mandatorias del entorno: Git, Motor Docker y Docker Compose
sudo apt-get install git docker.io docker-compose -y

# 3. Asegurar el arranque automático y activación del servicio de Docker con el sistema
sudo systemctl enable docker
sudo systemctl start docker

# 4. Modificación de privilegios para permitir ejecutar comandos de Docker sin anteponer 'sudo'
sudo usermod -aG docker $USER
newgrp docker

# 5. Clonación de la base de código que contiene los manifiestos de infraestructura
git clone [https://github.com/tu-usuario/tu-repositorio.git](https://github.com/tu-usuario/tu-repositorio.git)
cd tu-repositorio

# 6. Inyección dinámica del archivo de variables de entorno locales de producción
echo "DOCKERHUB_USERNAME=tu_usuario_dockerhub" >> .env
echo "DB_USER=admin_user" >> .env
echo "DB_PASSWORD=clave_encriptada_segura" >> .env

# 7. Autenticación contra el registro remoto para la descarga de imágenes
docker login

# 8. Despliegue final ininterrumpido en segundo plano del ecosistema multi-contenedor
docker-compose up -d --build
