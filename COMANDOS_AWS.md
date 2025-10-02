# 🚀 Comandos Útiles para Despliegue en AWS

## 📋 Comandos para la Instancia de Aplicación

### Conectar a la instancia
```bash
ssh -i "tu-key.pem" ubuntu@IP_PUBLICA_APLICACION
```

### Construir imagen Docker
```bash
sudo docker build -t sisfarma-app .
```

### Ejecutar contenedor
```bash
sudo docker run -d \
  --name sisfarma-container \
  -p 80:80 \
  -e APP_KEY="$(php artisan key:generate --show)" \
  -e DB_HOST="IP_DE_LA_INSTANCIA_MYSQL" \
  -e DB_PASSWORD="TU_PASSWORD_SEGURA" \
  sisfarma-app
```

### Ver logs del contenedor
```bash
sudo docker logs sisfarma-container
sudo docker logs -f sisfarma-container  # En tiempo real
```

### Ejecutar migraciones
```bash
sudo docker exec -it sisfarma-container php artisan migrate --force
```

### Ejecutar seeders
```bash
sudo docker exec -it sisfarma-container php artisan db:seed --force
```

### Acceder al contenedor
```bash
sudo docker exec -it sisfarma-container bash
```

### Detener contenedor
```bash
sudo docker stop sisfarma-container
sudo docker rm sisfarma-container
```

### Ver contenedores ejecutándose
```bash
sudo docker ps
```

### Ver todas las imágenes
```bash
sudo docker images
```

## 🗄️ Comandos para la Instancia de MySQL

### Conectar a la instancia
```bash
ssh -i "tu-key.pem" ubuntu@IP_PUBLICA_MYSQL
```

### Acceder a MySQL
```bash
sudo mysql -u root -p
```

### Crear base de datos
```sql
CREATE DATABASE sispharma CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

### Crear usuario
```sql
CREATE USER 'sisfarma_user'@'%' IDENTIFIED BY 'TU_PASSWORD_SEGURA';
GRANT ALL PRIVILEGES ON sispharma.* TO 'sisfarma_user'@'%';
FLUSH PRIVILEGES;
```

### Verificar usuarios
```sql
SELECT User, Host FROM mysql.user;
```

### Verificar base de datos
```sql
SHOW DATABASES;
```

### Verificar permisos
```sql
SHOW GRANTS FOR 'sisfarma_user'@'%';
```

## 🔧 Comandos de Mantenimiento

### Actualizar aplicación
```bash
# En la instancia de aplicación
cd SISFARMA
git pull origin main
sudo docker stop sisfarma-container
sudo docker rm sisfarma-container
sudo docker build -t sisfarma-app .
sudo docker run -d --name sisfarma-container -p 80:80 sisfarma-app
```

### Backup de base de datos
```bash
# En la instancia de MySQL
mysqldump -u root -p sispharma > backup_$(date +%Y%m%d_%H%M%S).sql
```

### Restaurar backup
```bash
# En la instancia de MySQL
mysql -u root -p sispharma < backup_20241002_180000.sql
```

### Verificar espacio en disco
```bash
df -h
```

### Verificar memoria
```bash
free -h
```

### Verificar procesos
```bash
top
htop  # Si está instalado
```

## 🔍 Comandos de Diagnóstico

### Verificar conectividad entre instancias
```bash
# Desde la instancia de aplicación
telnet IP_DE_LA_INSTANCIA_MYSQL 3306
```

### Verificar logs del sistema
```bash
sudo journalctl -u mysql
sudo journalctl -u apache2
```

### Verificar configuración de red
```bash
netstat -tlnp
ss -tlnp
```

### Verificar DNS
```bash
nslookup IP_DE_LA_INSTANCIA_MYSQL
```

## 🚨 Comandos de Emergencia

### Reiniciar MySQL
```bash
sudo systemctl restart mysql
```

### Reiniciar aplicación
```bash
sudo docker restart sisfarma-container
```

### Verificar estado de servicios
```bash
sudo systemctl status mysql
sudo docker ps
```

### Limpiar contenedores
```bash
sudo docker system prune -a
```

### Limpiar imágenes
```bash
sudo docker image prune -a
```

## 📊 Comandos de Monitoreo

### Ver uso de CPU
```bash
top -p $(pgrep -d',' -f "sisfarma")
```

### Ver uso de memoria
```bash
sudo docker stats sisfarma-container
```

### Ver logs de acceso
```bash
sudo docker logs sisfarma-container | grep "GET\|POST"
```

### Verificar puertos abiertos
```bash
sudo netstat -tlnp | grep :80
sudo netstat -tlnp | grep :3306
```

## 🔐 Comandos de Seguridad

### Verificar Security Groups
```bash
# Desde AWS CLI (si está instalado)
aws ec2 describe-security-groups --group-ids sg-xxxxxxxxx
```

### Verificar conexiones activas
```bash
sudo netstat -an | grep :3306
sudo netstat -an | grep :80
```

### Verificar logs de autenticación
```bash
sudo tail -f /var/log/auth.log
```

## 📝 Comandos de Configuración

### Configurar timezone
```bash
sudo timedatectl set-timezone America/Lima
```

### Configurar swap (si es necesario)
```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

### Configurar firewall (UFW)
```bash
sudo ufw enable
sudo ufw allow ssh
sudo ufw allow 80
sudo ufw allow 443
sudo ufw allow from IP_DE_LA_INSTANCIA_APLICACION to any port 3306
```

## 🎯 Comandos de Testing

### Probar conectividad HTTP
```bash
curl -I http://IP_PUBLICA_APLICACION
```

### Probar conectividad MySQL
```bash
mysql -h IP_DE_LA_INSTANCIA_MYSQL -u sisfarma_user -p
```

### Probar aplicación completa
```bash
curl -X POST http://IP_PUBLICA_APLICACION/login \
  -H "Content-Type: application/json" \
  -d '{"email":"ivanyomm.2003@gmail.com","password":"12345678"}'
```

¡Estos comandos te ayudarán a gestionar tu aplicación SiSFARMA en AWS! 🚀
