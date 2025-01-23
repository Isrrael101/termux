# Tutorial de Instalación y Configuración de MariaDB en Termux

## 1. Preparación Inicial
```bash
# Actualizar repositorios
pkg update
pkg upgrade

# Instalar MariaDB
pkg install mariadb
```

## 2. Configuración Inicial de MariaDB
```bash
# Instalar la base de datos inicial
mysql_install_db

# Iniciar el servidor MariaDB
mysqld_safe -u root &

# Esperar unos segundos a que el servidor inicie completamente
sleep 5
```

## 3. Configuración del archivo my.cnf
```bash
# Crear/editar el archivo de configuración
echo "[mysqld]
character-set-server = utf8mb3
collation-server = utf8mb3_general_ci
bind-address = 0.0.0.0
skip-networking = 0
skip-character-set-client-handshake

[client]
default-character-set = utf8mb3

[mysql]
default-character-set = utf8mb3" > $PREFIX/etc/my.cnf
```

## 4. Acceso y Configuración de Seguridad
```bash
# Conectarse a MariaDB
mariadb -u root

# Dentro de MariaDB, ejecutar:
SET GLOBAL character_set_server = 'utf8mb3';
SET GLOBAL collation_server = 'utf8mb3_general_ci';

# Para permitir conexiones remotas:
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'tu_contraseña';
FLUSH PRIVILEGES;

# Salir de MariaDB
quit;
```

## 5. Comandos Útiles de Administración
```bash
# Detener el servidor
pkill mysqld
pkill mariadb

# Reiniciar el servidor
mysqld_safe -u root &

# Ver estado del servidor
ps aux | grep mysql
ps aux | grep mariadb

# Ver logs
cat /data/data/com.termux/files/usr/var/lib/mysql/localhost.err
```

## 6. Ejemplo de Código Python para Conectarse
```python
import mysql.connector

try:
    # Conexión básica
    conexion = mysql.connector.connect(
        host="localhost",  # O tu IP para conexiones remotas
        user="root",
        password="tu_contraseña",
        charset='utf8mb3'
    )
    
    cursor = conexion.cursor()
    
    # Crear base de datos de prueba
    cursor.execute("CREATE DATABASE IF NOT EXISTS db_prueba CHARACTER SET utf8mb3 COLLATE utf8mb3_general_ci")
    cursor.execute("USE db_prueba")
    
    # Crear tabla de ejemplo
    cursor.execute("""
        CREATE TABLE IF NOT EXISTS usuarios (
            id INT AUTO_INCREMENT PRIMARY KEY,
            nombre VARCHAR(100) CHARACTER SET utf8mb3 COLLATE utf8mb3_general_ci,
            edad INT
        ) ENGINE=InnoDB DEFAULT CHARACTER SET utf8mb3 COLLATE utf8mb3_general_ci
    """)
    
    print("Base de datos y tabla creadas exitosamente")

except mysql.connector.Error as error:
    print(f"Error: {error}")

finally:
    if 'conexion' in locals() and conexion.is_connected():
        cursor.close()
        conexion.close()
        print("Conexión cerrada")
```

## 7. Solución de Problemas Comunes

### Error de Collation
Si encuentras el error "Unknown collation: 'utf8mb4_0900_ai_ci'", asegúrate de:
1. Usar utf8mb3 en lugar de utf8mb4
2. Tener la configuración correcta en my.cnf
3. Reiniciar el servidor después de cambiar la configuración

### Error de Conexión
Si no puedes conectarte:
1. Verifica que el servidor esté corriendo con `ps aux | grep mysql`
2. Asegúrate de usar las credenciales correctas
3. Reinicia el servidor si es necesario

### Error de Permisos
Si tienes problemas de permisos:
1. Verifica los permisos del usuario
2. Asegúrate de que la base de datos existe
3. Verifica que el usuario tiene acceso desde la IP que estás usando

## 8. Para Conexiones Remotas

1. Encuentra tu IP:
```bash
ip addr
```

2. Configura el servidor para aceptar conexiones remotas:
- Asegúrate de que bind-address = 0.0.0.0 está en my.cnf
- Otorga permisos al usuario para conectarse desde cualquier IP
- El puerto por defecto es 3306

3. Usa la IP de tu dispositivo en lugar de localhost en el código Python

## 9. Mantenimiento

### Backup de Base de Datos
```bash
mysqldump -u root nombre_base > backup.sql
```

### Restaurar Backup
```bash
mariadb -u root nombre_base < backup.sql
```

### Ver Tamaño de Bases de Datos
```sql
SELECT table_schema "Database Name",
ROUND(SUM(data_length + index_length) / 1024 / 1024, 1) "Size in MB"
FROM information_schema.tables
GROUP BY table_schema;
```

## 10. Consideraciones de Seguridad

1. Siempre usa contraseñas fuertes
2. Limita los permisos de usuarios según sea necesario
3. Regularmente actualiza MariaDB
4. Haz backups periódicos
5. Monitorea los logs por actividad sospechosa