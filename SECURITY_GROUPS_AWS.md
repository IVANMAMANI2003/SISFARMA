# 🔒 Configuración de Security Groups para SiSFARMA

## 📋 Security Group para la Aplicación (EC2 - Aplicación)

### Reglas de Entrada (Inbound Rules)

| Tipo | Protocolo | Puerto | Origen | Descripción |
|------|-----------|--------|--------|-------------|
| SSH | TCP | 22 | Tu IP | Acceso SSH para administración |
| HTTP | TCP | 80 | 0.0.0.0/0 | Acceso web HTTP |
| HTTPS | TCP | 443 | 0.0.0.0/0 | Acceso web HTTPS (opcional) |
| Custom TCP | TCP | 8000 | 0.0.0.0/0 | Puerto de desarrollo (opcional) |

### Reglas de Salida (Outbound Rules)

| Tipo | Protocolo | Puerto | Destino | Descripción |
|------|-----------|--------|---------|-------------|
| All Traffic | All | All | 0.0.0.0/0 | Todo el tráfico de salida |

## 🗄️ Security Group para MySQL (EC2 - Base de Datos)

### Reglas de Entrada (Inbound Rules)

| Tipo | Protocolo | Puerto | Origen | Descripción |
|------|-----------|--------|--------|-------------|
| SSH | TCP | 22 | Tu IP | Acceso SSH para administración |
| MySQL | TCP | 3306 | IP de la instancia de aplicación | Acceso a MySQL desde la aplicación |

### Reglas de Salida (Outbound Rules)

| Tipo | Protocolo | Puerto | Destino | Descripción |
|------|-----------|--------|---------|-------------|
| All Traffic | All | All | 0.0.0.0/0 | Todo el tráfico de salida |

## 🛠️ Pasos para Configurar Security Groups

### 1. Crear Security Group para la Aplicación

1. **Acceder a AWS Console** → EC2 → Security Groups → Create Security Group
2. **Nombre**: `sisfarma-app-sg`
3. **Descripción**: `Security Group para la aplicación SiSFARMA`
4. **VPC**: Seleccionar la VPC por defecto
5. **Agregar reglas de entrada** según la tabla anterior
6. **Crear Security Group**

### 2. Crear Security Group para MySQL

1. **Acceder a AWS Console** → EC2 → Security Groups → Create Security Group
2. **Nombre**: `sisfarma-mysql-sg`
3. **Descripción**: `Security Group para MySQL de SiSFARMA`
4. **VPC**: Seleccionar la VPC por defecto
5. **Agregar reglas de entrada** según la tabla anterior
6. **Crear Security Group**

### 3. Asignar Security Groups a las Instancias

#### Para la Instancia de Aplicación:
1. **Seleccionar instancia** → Actions → Security → Change Security Groups
2. **Seleccionar**: `sisfarma-app-sg`
3. **Aplicar cambios**

#### Para la Instancia de MySQL:
1. **Seleccionar instancia** → Actions → Security → Change Security Groups
2. **Seleccionar**: `sisfarma-mysql-sg`
3. **Aplicar cambios**

## 🔐 Configuración de Seguridad Adicional

### 1. Configurar IP Elástica (Elastic IP)

#### Para la Instancia de Aplicación:
1. **EC2** → Elastic IPs → Allocate Elastic IP
2. **Asociar** a la instancia de aplicación
3. **Actualizar** el archivo `.env` con la nueva IP

#### Para la Instancia de MySQL:
1. **EC2** → Elastic IPs → Allocate Elastic IP
2. **Asociar** a la instancia de MySQL
3. **Actualizar** la configuración de la aplicación

### 2. Configurar Key Pairs

#### Crear Key Pair:
1. **EC2** → Key Pairs → Create Key Pair
2. **Nombre**: `sisfarma-key`
3. **Tipo**: RSA
4. **Formato**: .pem
5. **Descargar** y guardar en lugar seguro

#### Usar Key Pair existente:
- Asegúrate de tener acceso al archivo .pem
- Configurar permisos: `chmod 400 sisfarma-key.pem`

### 3. Configurar VPC (Opcional)

#### Crear VPC personalizada:
1. **VPC** → Create VPC
2. **Nombre**: `sisfarma-vpc`
3. **CIDR**: `10.0.0.0/16`
4. **Crear subredes**:
   - `10.0.1.0/24` (Aplicación)
   - `10.0.2.0/24` (MySQL)
5. **Configurar Route Tables**
6. **Configurar Internet Gateway**

## 🚨 Consideraciones de Seguridad

### 1. Restricciones de IP

#### Para SSH:
- **Solo tu IP**: `TU_IP_PUBLICA/32`
- **Rango de oficina**: `IP_OFICINA/24` (si es necesario)

#### Para MySQL:
- **Solo IP de aplicación**: `IP_INSTANCIA_APLICACION/32`
- **No abrir** a 0.0.0.0/0

### 2. Configuración de Firewall

#### En la instancia de aplicación:
```bash
sudo ufw enable
sudo ufw allow ssh
sudo ufw allow 80
sudo ufw allow 443
```

#### En la instancia de MySQL:
```bash
sudo ufw enable
sudo ufw allow ssh
sudo ufw allow from IP_INSTANCIA_APLICACION to any port 3306
```

### 3. Configuración de MySQL

#### Configurar bind-address:
```bash
# Editar /etc/mysql/mysql.conf.d/mysqld.cnf
bind-address = 0.0.0.0
```

#### Configurar usuario con IP específica:
```sql
CREATE USER 'sisfarma_user'@'IP_INSTANCIA_APLICACION' IDENTIFIED BY 'PASSWORD_SEGURA';
GRANT ALL PRIVILEGES ON sispharma.* TO 'sisfarma_user'@'IP_INSTANCIA_APLICACION';
FLUSH PRIVILEGES;
```

## 📊 Monitoreo de Seguridad

### 1. CloudTrail
- **Habilitar** para auditoría de API calls
- **Configurar** alertas para cambios en Security Groups

### 2. CloudWatch
- **Monitorear** métricas de las instancias
- **Configurar** alertas para tráfico anómalo

### 3. VPC Flow Logs
- **Habilitar** para monitorear tráfico de red
- **Analizar** logs para detectar patrones anómalos

## 🔄 Actualizaciones de Seguridad

### 1. Actualizar Security Groups
- **Revisar** regularmente las reglas
- **Eliminar** reglas innecesarias
- **Restringir** acceso cuando sea posible

### 2. Actualizar instancias
- **Aplicar** parches de seguridad
- **Actualizar** Docker y dependencias
- **Revisar** logs de seguridad

### 3. Rotar credenciales
- **Cambiar** contraseñas regularmente
- **Rotar** keys de acceso
- **Actualizar** certificados SSL

## ⚠️ Mejores Prácticas

1. **Principio de menor privilegio**: Solo dar acceso necesario
2. **Seguridad en capas**: Múltiples niveles de protección
3. **Monitoreo continuo**: Vigilar actividad sospechosa
4. **Backups seguros**: Proteger datos de respaldo
5. **Documentación**: Mantener registro de cambios

¡Con esta configuración tu aplicación SiSFARMA estará segura en AWS! 🔒
