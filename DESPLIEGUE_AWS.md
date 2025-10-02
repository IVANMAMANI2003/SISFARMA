# 🚀 Despliegue de SiSFARMA en AWS con Docker

## 📋 Arquitectura Propuesta

```
┌─────────────────┐    ┌─────────────────┐
│   EC2 Instance  │    │   EC2 Instance  │
│   (Aplicación)  │    │   (MySQL)       │
│                 │    │                 │
│  ┌─────────────┐│    │  ┌─────────────┐│
│  │   Docker    ││    │  │   MySQL     ││
│  │   Container ││◄───┤  │   Server    ││
│  │   Laravel   ││    │  │             ││
│  └─────────────┘│    │  └─────────────┘│
└─────────────────┘    └─────────────────┘
        │                       │
        └───────────────────────┘
              Security Group
```

## 🛠️ Paso 1: Preparar la Aplicación

### 1.1 Configurar variables de entorno para producción

Crea un archivo `.env.production`:

```env
APP_NAME=SiSFARMA
APP_ENV=production
APP_KEY=base64:TU_APP_KEY_AQUI
APP_DEBUG=false
APP_URL=http://TU_IP_PUBLICA

APP_LOCALE=es
APP_FALLBACK_LOCALE=es
APP_FAKER_LOCALE=es_ES

APP_MAINTENANCE_DRIVER=file

PHP_CLI_SERVER_WORKERS=4

BCRYPT_ROUNDS=12

LOG_CHANNEL=stack
LOG_STACK=single
LOG_DEPRECATIONS_CHANNEL=null
LOG_LEVEL=error

# Configuración de MySQL (IP de la instancia de base de datos)
DB_CONNECTION=mysql
DB_HOST=IP_DE_LA_INSTANCIA_MYSQL
DB_PORT=3306
DB_DATABASE=sispharma
DB_USERNAME=root
DB_PASSWORD=TU_PASSWORD_SEGURA

SESSION_DRIVER=database
SESSION_LIFETIME=120
SESSION_ENCRYPT=false
SESSION_PATH=/
SESSION_DOMAIN=null

BROADCAST_CONNECTION=log
FILESYSTEM_DISK=local
QUEUE_CONNECTION=database

CACHE_STORE=database

MEMCACHED_HOST=127.0.0.1

REDIS_CLIENT=phpredis
REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379

MAIL_MAILER=log
MAIL_SCHEME=null
MAIL_HOST=127.0.0.1
MAIL_PORT=2525
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_FROM_ADDRESS="hello@example.com"
MAIL_FROM_NAME="${APP_NAME}"

AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=
AWS_USE_PATH_STYLE_ENDPOINT=false

VITE_APP_NAME="${APP_NAME}"
```

### 1.2 Subir código a GitHub

```bash
git add .
git commit -m "Preparar aplicación para despliegue en AWS"
git push origin main
```

## 🖥️ Paso 2: Configurar Instancia EC2 para la Aplicación

### 2.1 Crear instancia EC2

1. **Acceder a AWS Console** → EC2 → Launch Instance
2. **AMI**: Ubuntu Server 22.04 LTS
3. **Instance Type**: t3.medium (mínimo recomendado)
4. **Key Pair**: Crear o seleccionar una existente
5. **Security Group**: Crear nuevo con las siguientes reglas:
   - SSH (22) - Tu IP
   - HTTP (80) - 0.0.0.0/0
   - HTTPS (443) - 0.0.0.0/0
   - Custom TCP (8000) - 0.0.0.0/0 (para testing)

### 2.2 Conectar a la instancia

```bash
ssh -i "tu-key.pem" ubuntu@IP_PUBLICA_APLICACION
```

### 2.3 Instalar Docker en la instancia

```bash
# Actualizar sistema
sudo apt update && sudo apt upgrade -y

# Instalar Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Agregar usuario ubuntu al grupo docker
sudo usermod -aG docker ubuntu

# Instalar Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/download/v2.20.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Reiniciar sesión
exit
```

### 2.4 Reconectar y clonar repositorio

```bash
ssh -i "tu-key.pem" ubuntu@IP_PUBLICA_APLICACION

# Clonar repositorio
git clone https://github.com/IVANMAMANI2003/SISFARMA.git
cd SISFARMA

# Crear archivo .env para producción
cp .env.production .env
```

### 2.5 Construir y ejecutar contenedor

```bash
# Construir imagen
sudo docker build -t sisfarma-app .

# Ejecutar contenedor
sudo docker run -d \
  --name sisfarma-container \
  -p 80:80 \
  -e APP_KEY="$(php artisan key:generate --show)" \
  -e DB_HOST="IP_DE_LA_INSTANCIA_MYSQL" \
  -e DB_PASSWORD="TU_PASSWORD_SEGURA" \
  sisfarma-app
```

## 🗄️ Paso 3: Configurar Instancia EC2 para MySQL

### 3.1 Crear segunda instancia EC2

1. **AMI**: Ubuntu Server 22.04 LTS
2. **Instance Type**: t3.small (suficiente para MySQL)
3. **Key Pair**: Misma que la anterior
4. **Security Group**: Crear nuevo con las siguientes reglas:
   - SSH (22) - Tu IP
   - MySQL (3306) - IP de la instancia de aplicación

### 3.2 Conectar a la instancia MySQL

```bash
ssh -i "tu-key.pem" ubuntu@IP_PUBLICA_MYSQL
```

### 3.3 Instalar MySQL

```bash
# Actualizar sistema
sudo apt update && sudo apt upgrade -y

# Instalar MySQL
sudo apt install mysql-server -y

# Configurar MySQL
sudo mysql_secure_installation

# Iniciar y habilitar MySQL
sudo systemctl start mysql
sudo systemctl enable mysql
```

### 3.4 Configurar MySQL para conexión remota

```bash
# Acceder a MySQL
sudo mysql -u root -p

# Crear base de datos
CREATE DATABASE sispharma CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

# Crear usuario para la aplicación
CREATE USER 'sisfarma_user'@'%' IDENTIFIED BY 'TU_PASSWORD_SEGURA';
GRANT ALL PRIVILEGES ON sispharma.* TO 'sisfarma_user'@'%';
FLUSH PRIVILEGES;

# Configurar MySQL para aceptar conexiones remotas
exit
```

### 3.5 Editar configuración de MySQL

```bash
# Editar archivo de configuración
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf

# Cambiar la línea:
# bind-address = 127.0.0.1
# Por:
# bind-address = 0.0.0.0

# Reiniciar MySQL
sudo systemctl restart mysql
```

## 🔧 Paso 4: Configurar la Aplicación

### 4.1 Ejecutar migraciones en la instancia de aplicación

```bash
# Conectar a la instancia de aplicación
ssh -i "tu-key.pem" ubuntu@IP_PUBLICA_APLICACION

# Ejecutar migraciones
sudo docker exec -it sisfarma-container php artisan migrate --force

# Ejecutar seeders
sudo docker exec -it sisfarma-container php artisan db:seed --force
```

### 4.2 Configurar dominio (opcional)

Si tienes un dominio, puedes configurar un Load Balancer o usar Route 53.

## 🔒 Paso 5: Configuración de Seguridad

### 5.1 Configurar Security Groups

**Security Group para Aplicación:**
- SSH (22): Tu IP
- HTTP (80): 0.0.0.0/0
- HTTPS (443): 0.0.0.0/0

**Security Group para MySQL:**
- SSH (22): Tu IP
- MySQL (3306): IP de la instancia de aplicación

### 5.2 Configurar SSL (opcional)

Puedes usar Let's Encrypt con Certbot para SSL gratuito.

## 📊 Paso 6: Monitoreo y Logs

### 6.1 Ver logs de la aplicación

```bash
# Ver logs del contenedor
docker logs sisfarma-container

# Ver logs en tiempo real
docker logs -f sisfarma-container
```

### 6.2 Verificar estado de la aplicación

```bash
# Verificar que el contenedor esté ejecutándose
sudo docker ps

# Verificar conectividad a MySQL
sudo docker exec -it sisfarma-container php artisan tinker
# En tinker:
# DB::connection()->getPdo();
```

## 🚀 Paso 7: Acceso a la Aplicación

Una vez configurado todo:

1. **URL de la aplicación**: `http://IP_PUBLICA_APLICACION`
2. **Credenciales de acceso**:
   - Email: `ivanyomm.2003@gmail.com`
   - Password: `12345678`

## ⚠️ IMPORTANTE: Uso de SUDO

**TODOS los comandos de Docker requieren `sudo`** para evitar problemas de permisos. Siempre usa:

```bash
sudo docker build -t sisfarma-app .
sudo docker run -d --name sisfarma-container -p 80:80 sisfarma-app
sudo docker exec -it sisfarma-container php artisan migrate --force
```

Ver archivo `COMANDOS_SUDO_AWS.md` para lista completa de comandos con `sudo`.

## 🔄 Paso 8: Actualizaciones

Para actualizar la aplicación:

```bash
# Conectar a la instancia de aplicación
ssh -i "tu-key.pem" ubuntu@IP_PUBLICA_APLICACION

# Detener contenedor actual
sudo docker stop sisfarma-container
sudo docker rm sisfarma-container

# Actualizar código
cd SISFARMA
git pull origin main

# Reconstruir y ejecutar
sudo docker build -t sisfarma-app .
sudo docker run -d \
  --name sisfarma-container \
  -p 80:80 \
  -e APP_KEY="$(php artisan key:generate --show)" \
  -e DB_HOST="IP_DE_LA_INSTANCIA_MYSQL" \
  -e DB_PASSWORD="TU_PASSWORD_SEGURA" \
  sisfarma-app
```

## 💰 Costos Estimados

- **Instancia de Aplicación (t3.medium)**: ~$30/mes
- **Instancia de MySQL (t3.small)**: ~$15/mes
- **Total aproximado**: ~$45/mes

## ⚠️ Consideraciones Importantes

1. **Backups**: Configura backups automáticos de la base de datos
2. **Monitoreo**: Usa CloudWatch para monitorear las instancias
3. **Escalabilidad**: Considera usar Auto Scaling Groups para la aplicación
4. **Seguridad**: Mantén las instancias actualizadas y usa Security Groups restrictivos
5. **Logs**: Configura rotación de logs para evitar llenar el disco

## 🆘 Solución de Problemas

### Error de conexión a MySQL
- Verifica Security Groups
- Confirma que MySQL esté ejecutándose
- Revisa configuración de bind-address

### Error 500 en la aplicación
- Revisa logs: `docker logs sisfarma-container`
- Verifica variables de entorno
- Confirma que las migraciones se ejecutaron

### Contenedor no inicia
- Verifica que la imagen se construyó correctamente
- Revisa logs: `docker logs sisfarma-container`
- Confirma que el puerto 80 esté disponible

¡Tu aplicación SiSFARMA estará funcionando en AWS con Docker! 🎉
