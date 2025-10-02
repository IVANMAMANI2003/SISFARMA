# 🔐 Comandos con SUDO para AWS - SiSFARMA

## ⚠️ IMPORTANTE: Siempre usar SUDO

Todos los comandos de Docker requieren permisos de administrador. **SIEMPRE** usa `sudo` antes de los comandos de Docker para evitar errores de permisos.

## 🚀 Comandos Esenciales con SUDO

### 1. Construcción y Ejecución

```bash
# Construir imagen
sudo docker build -t sisfarma-app .

# Ejecutar contenedor
sudo docker run -d \
  --name sisfarma-container \
  -p 80:80 \
  -e APP_KEY="$(php artisan key:generate --show)" \
  -e DB_HOST="3.93.60.179" \
  -e DB_PASSWORD="" \
  sisfarma-app

# Ver contenedores ejecutándose
sudo docker ps

# Ver todas las imágenes
sudo docker images
```

### 2. Gestión de Contenedores

```bash
# Detener contenedor
sudo docker stop sisfarma-container

# Eliminar contenedor
sudo docker rm sisfarma-container

# Reiniciar contenedor
sudo docker restart sisfarma-container

# Ver logs del contenedor
sudo docker logs sisfarma-container
sudo docker logs -f sisfarma-container  # En tiempo real
```

### 3. Ejecutar Comandos en el Contenedor

```bash
# Acceder al contenedor
sudo docker exec -it sisfarma-container bash

# Ejecutar migraciones
sudo docker exec -it sisfarma-container php artisan migrate --force

# Ejecutar seeders
sudo docker exec -it sisfarma-container php artisan db:seed --force

# Generar clave de aplicación
sudo docker exec -it sisfarma-container php artisan key:generate

# Limpiar cache
sudo docker exec -it sisfarma-container php artisan cache:clear
sudo docker exec -it sisfarma-container php artisan config:clear
sudo docker exec -it sisfarma-container php artisan route:clear
sudo docker exec -it sisfarma-container php artisan view:clear
```

### 4. Monitoreo y Diagnóstico

```bash
# Ver estadísticas del contenedor
sudo docker stats sisfarma-container

# Ver uso de recursos
sudo docker exec -it sisfarma-container top

# Verificar conectividad a MySQL
sudo docker exec -it sisfarma-container php artisan tinker
# En tinker ejecutar:
# DB::connection()->getPdo();

# Ver logs de Apache
sudo docker exec -it sisfarma-container tail -f /var/log/apache2/error.log
```

### 5. Limpieza y Mantenimiento

```bash
# Limpiar contenedores parados
sudo docker container prune

# Limpiar imágenes no utilizadas
sudo docker image prune

# Limpiar todo (¡CUIDADO!)
sudo docker system prune -a

# Ver espacio usado por Docker
sudo docker system df
```

### 6. Actualización de la Aplicación

```bash
# Detener contenedor actual
sudo docker stop sisfarma-container
sudo docker rm sisfarma-container

# Actualizar código
cd SISFARMA
git pull origin main

# Reconstruir imagen
sudo docker build -t sisfarma-app .

# Ejecutar nuevo contenedor
sudo docker run -d \
  --name sisfarma-container \
  -p 80:80 \
  -e APP_KEY="$(php artisan key:generate --show)" \
  -e DB_HOST="3.93.60.179" \
  -e DB_PASSWORD="" \
  sisfarma-app

# Verificar que esté funcionando
sudo docker ps
sudo docker logs sisfarma-container
```

### 7. Backup y Restauración

```bash
# Crear backup del contenedor
sudo docker commit sisfarma-container sisfarma-backup:$(date +%Y%m%d)

# Crear backup de la base de datos
sudo docker exec -it sisfarma-container mysqldump -h 3.93.60.179 -u root -p sispharma > backup_$(date +%Y%m%d_%H%M%S).sql
```

### 8. Configuración de Red

```bash
# Ver redes de Docker
sudo docker network ls

# Crear red personalizada
sudo docker network create sisfarma-network

# Ejecutar contenedor en red específica
sudo docker run -d \
  --name sisfarma-container \
  --network sisfarma-network \
  -p 80:80 \
  sisfarma-app
```

### 9. Variables de Entorno

```bash
# Ejecutar con archivo .env
sudo docker run -d \
  --name sisfarma-container \
  --env-file .env \
  -p 80:80 \
  sisfarma-app

# Ver variables de entorno del contenedor
sudo docker exec -it sisfarma-container env
```

### 10. Volúmenes y Persistencia

```bash
# Crear volumen para storage
sudo docker volume create sisfarma-storage

# Ejecutar con volumen montado
sudo docker run -d \
  --name sisfarma-container \
  -v sisfarma-storage:/var/www/html/storage \
  -p 80:80 \
  sisfarma-app

# Ver volúmenes
sudo docker volume ls
```

## 🔧 Comandos de Emergencia

### Si el contenedor no inicia:

```bash
# Ver logs de error
sudo docker logs sisfarma-container

# Verificar configuración
sudo docker inspect sisfarma-container

# Eliminar y recrear
sudo docker rm -f sisfarma-container
sudo docker run -d --name sisfarma-container -p 80:80 sisfarma-app
```

### Si hay problemas de permisos:

```bash
# Verificar permisos de Docker
sudo chmod 666 /var/run/docker.sock

# Agregar usuario al grupo docker
sudo usermod -aG docker ubuntu

# Reiniciar sesión
exit
# Volver a conectar
```

### Si hay problemas de red:

```bash
# Verificar puertos
sudo netstat -tlnp | grep :80

# Verificar conectividad
sudo docker exec -it sisfarma-container ping 3.93.60.179

# Verificar DNS
sudo docker exec -it sisfarma-container nslookup 3.93.60.179
```

## 📝 Notas Importantes

1. **SIEMPRE** usa `sudo` con comandos de Docker
2. **Verifica** que el contenedor esté ejecutándose con `sudo docker ps`
3. **Revisa logs** si hay problemas: `sudo docker logs sisfarma-container`
4. **Mantén** las imágenes actualizadas
5. **Haz backup** regularmente de la base de datos

## 🚨 Comandos Peligrosos (Usar con Cuidado)

```bash
# Eliminar TODAS las imágenes (¡CUIDADO!)
sudo docker rmi $(sudo docker images -q)

# Eliminar TODOS los contenedores (¡CUIDADO!)
sudo docker rm $(sudo docker ps -aq)

# Limpiar TODO el sistema Docker (¡CUIDADO!)
sudo docker system prune -a --volumes
```

¡Recuerda: **SUDO es tu amigo** para evitar problemas de permisos! 🔐
