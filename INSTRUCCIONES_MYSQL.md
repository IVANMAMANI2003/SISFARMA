# 🏥 SiSFARMA - Configuración con MySQL

## 📋 Requisitos Previos

1. **MySQL** instalado y ejecutándose
2. **PHP** 8.1 o superior
3. **Composer** instalado
4. **Node.js** y **npm** (opcional, para assets)

## 🚀 Pasos de Configuración

### 1. Crear el archivo .env

Crea un archivo `.env` en la raíz del proyecto con este contenido:

```env
APP_NAME=SiSFARMA
APP_ENV=local
APP_KEY=base64:TU_APP_KEY_AQUI
APP_DEBUG=true
APP_URL=http://localhost

APP_LOCALE=es
APP_FALLBACK_LOCALE=es
APP_FAKER_LOCALE=es_ES

APP_MAINTENANCE_DRIVER=file

PHP_CLI_SERVER_WORKERS=4

BCRYPT_ROUNDS=12

LOG_CHANNEL=stack
LOG_STACK=single
LOG_DEPRECATIONS_CHANNEL=null
LOG_LEVEL=debug

# Configuración de MySQL
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=sispharma
DB_USERNAME=root
DB_PASSWORD=tu_password_aqui

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

### 2. Generar la APP_KEY

```bash
php artisan key:generate
```

### 3. Crear la base de datos

Conecta a MySQL y ejecuta:

```sql
CREATE DATABASE sispharma CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

### 4. Instalar dependencias

```bash
composer install
npm install
```

### 5. Ejecutar migraciones

```bash
php artisan migrate
```

### 6. Poblar la base de datos (opcional)

```bash
php artisan db:seed
```

### 7. Compilar assets (opcional)

```bash
npm run build
# o para desarrollo:
npm run dev
```

### 8. Iniciar el servidor

```bash
php artisan serve
```

## 🔧 Configuración Automática

### Windows
Ejecuta el archivo `configurar_mysql.bat`:

```cmd
configurar_mysql.bat
```

### Linux/Mac
Ejecuta el archivo `configurar_mysql.sh`:

```bash
./configurar_mysql.sh
```

## 👤 Usuario por Defecto

Después de ejecutar los seeders, tendrás un usuario administrador:

- **Email:** ivanyomm.2003@gmail.com
- **Password:** 12345678

## 🗄️ Estructura de la Base de Datos

El sistema incluye las siguientes tablas:

- `users` - Usuarios del sistema
- `suppliers` - Proveedores
- `categories` - Categorías de productos
- `products` - Productos
- `clients` - Clientes
- `purchases` - Compras
- `sales` - Ventas
- `details` - Detalles de ventas
- `cache` - Cache del sistema
- `jobs` - Cola de trabajos
- `migrations` - Control de migraciones

## 🐛 Solución de Problemas

### Error de conexión a MySQL
- Verifica que MySQL esté ejecutándose
- Confirma la contraseña en el archivo `.env`
- Asegúrate de que el puerto 3306 esté disponible

### Error de permisos
- Verifica que el usuario MySQL tenga permisos para crear bases de datos
- En Windows, ejecuta el terminal como administrador

### Error de migraciones
- Verifica que la base de datos `sispharma` exista
- Revisa los logs en `storage/logs/laravel.log`

## 📞 Soporte

Si encuentras algún problema, revisa:
1. Los logs en `storage/logs/laravel.log`
2. La configuración de MySQL
3. Los permisos de archivos y carpetas

¡Tu sistema de farmacia está listo para usar! 🎉
