# Instalación completa de Nextcloud en Ubuntu 24.04

Guía concisa para instalar Nextcloud en un servidor Ubuntu 24.04 con Apache, MariaDB, PHP-FPM, HTTPS y optimizaciones básicas.

## Resumen de pasos
1. Actualizar sistema
2. Instalar Apache, MariaDB, PHP y extensiones
3. Configurar base de datos
4. Descargar y desplegar Nextcloud
5. Ajustar permisos y configuración Apache/PHP
6. Activar HTTPS con Certbot
7. (Opcional) Redis para caching y bloqueo de archivos
8. Configurar tareas cron y optimizaciones PHP

---

## 1) Actualizar sistema
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y software-properties-common curl unzip wget
```

## 2) Instalar Apache, MariaDB y PHP (PHP 8.3 en 24.04)
```bash
sudo apt install -y apache2 mariadb-server libapache2-mod-fcgid
sudo apt install -y php8.3-fpm php8.3-cli php8.3-mysql php8.3-xml php8.3-gd \
    php8.3-curl php8.3-zip php8.3-mbstring php8.3-intl php8.3-bz2 php8.3-imagick \
    php8.3-common php8.3-opcache php8.3-gmp php8.3-redis
```
Habilitar módulos de Apache:
```bash
sudo a2enmod proxy proxy_fcgi setenvif rewrite headers env dir mime ssl
sudo systemctl restart apache2
```

## 3) Asegurar y crear base de datos MariaDB
Ejecutar el asistente de seguridad:
```bash
sudo mysql_secure_installation
sudo mysql -u root -p
```
Crear la base de datos y usuario (reemplazar contraseña segura):
```sql
CREATE DATABASE nextcloud CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
CREATE USER 'ncuser'@'localhost' IDENTIFIED BY 'TuPasswordSegura';
GRANT ALL PRIVILEGES ON nextcloud.* TO 'ncuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

## 4) Descargar y desplegar Nextcloud
```bash
cd /tmp
wget https://download.nextcloud.com/server/releases/latest.zip
unzip latest.zip
sudo mv nextcloud /var/www/nextcloud
```

## 5) Permisos y configuración de Apache
Instalar acl (recomendado) y ajustar permisos:
```bash
sudo apt install -y acl
sudo chown -R www-data:www-data /var/www/nextcloud
sudo find /var/www/nextcloud/ -type d -exec chmod 755 {} \;
sudo find /var/www/nextcloud/ -type f -exec chmod 640 {} \;
# Opcional (mejor para despliegues multiusuario):
sudo setfacl -R -m u:www-data:rwx -m u:$USER:rwx /var/www/nextcloud
sudo setfacl -dR -m u:www-data:rwx -m u:$USER:rwx /var/www/nextcloud
```

Crear archivo de sitio Apache (`/etc/apache2/sites-available/nextcloud.conf`):
```apache
<VirtualHost *:80>
        ServerName your.domain.tld
        DocumentRoot /var/www/nextcloud

        <Directory /var/www/nextcloud/>
            Require all granted
            AllowOverride All
            Options FollowSymlinks MultiViews
            <IfModule mod_dav.c>
                Dav off
            </IfModule>
        </Directory>

        <FilesMatch "\.php$">
            SetHandler "proxy:unix:/run/php/php8.3-fpm.sock|fcgi://localhost/"
        </FilesMatch>

        ErrorLog ${APACHE_LOG_DIR}/nextcloud_error.log
        CustomLog ${APACHE_LOG_DIR}/nextcloud_access.log combined
</VirtualHost>
```
Habilitar sitio y reiniciar:
```bash
sudo a2ensite nextcloud.conf
sudo systemctl reload apache2
```

## 6) Configurar PHP opcional (php.ini vs pool)
Ajustes recomendados en /etc/php/8.3/fpm/php.ini (valores ejemplo):
- memory_limit = 512M
- upload_max_filesize = 1024M
- post_max_size = 1024M
- max_execution_time = 360
- opcache.enable=1 y configuración de opcache (ver docs Nextcloud)

Reiniciar PHP-FPM:
```bash
sudo systemctl restart php8.3-fpm
```

## 7) Finalizar instalación vía web o CLI
Abrir en navegador http://your.domain.tld y completar:
- Crear cuenta administrador
- Configurar datos de almacenamiento (por defecto data/ dentro de Nextcloud)
- Poner datos de DB (nextcloud / ncuser / TuPasswordSegura)
O usar occ (desde /var/www/nextcloud):
```bash
cd /var/www/nextcloud
sudo -u www-data php occ maintenance:install \
  --admin-user=admin --admin-pass='AdminPass' \
  --database='mysql' --database-name='nextcloud' \
  --database-user='ncuser' --database-pass='TuPasswordSegura'
```

## 8) Activar HTTPS (Certbot)
Instalar Certbot y obtener certificado:
```bash
sudo snap install core; sudo snap refresh core
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot --apache -d your.domain.tld
```
Verificar renovación: `sudo certbot renew --dry-run`

## 9) Cron para tareas de Nextcloud
Agregar cron para el usuario www-data:
```bash
sudo crontab -u www-data -e
# Añadir línea:
*/15 * * * * php -f /var/www/nextcloud/cron.php
```
En config.php (opcional) establecer 'cron' como modo background.

## 10) (Opcional) Redis para caching y file locking
Instalar redis:
```bash
sudo apt install -y redis-server php8.3-redis
sudo systemctl enable --now redis
```
Añadir a /var/www/nextcloud/config/config.php:
```php
'memcache.local' => '\\OC\\Memcache\\Redis',
'redis' => [
    'host' => 'localhost',
    'port' => 6379,
    // 'password' => 'tu_password_redis'  // si aplica
],
'memcache.locking' => '\\OC\\Memcache\\Redis',
```
Asegurar que la extensión php-redis está activa y reiniciar php-fpm.

## 11) Recomendaciones finales
- Mantener sistema y Nextcloud actualizados.
- Hacer backups regulares (base de datos + carpeta data + config).
- Revisar requisitos oficiales de Nextcloud antes de producción.
- Activar seguridad adicional (fail2ban, firewall UFW).

---

Si necesitas un archivo completo listo para pegar en `/C:/Users/profe/OneDrive/ServiciosOpenSourceInstalacion/nextcloud.md` con todos los bloques de comandos y el vhost configurado, indícalo y lo genero.