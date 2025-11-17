# Guía de Instalación de Moodle en Ubuntu 24.04

## Requisitos Previos

Antes de comenzar, asegúrate de tener los siguientes requisitos:

- Un servidor Ubuntu 22.04.
- Acceso a la terminal con privilegios de sudo.
- Un servidor web (Apache o Nginx).
- PHP y extensiones necesarias.
- Base de datos (MySQL, PostgreSQL, MariaDB ...).

## Paso 1: Actualizar el Sistema

```bash
sudo apt update && sudo apt upgrade -y
```

## Paso 2: Instalar Apache

```bash
sudo apt install apache2 -y
```

## Paso 3: Instalar PHP y Extensiones

```bash
sudo apt install php php-cli php-fpm php-mysql php-xml php-mbstring php-zip php-curl -y
```

## Paso 4: Instalar MySQL

```bash
sudo apt install mysql-server -y
```

### Configurar MySQL

```bash
sudo mysql_secure_installation
```

## Paso 5: Crear Base de Datos para Moodle

```bash
sudo mysql -u root -p
```

```sql
CREATE DATABASE moodle DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'moodleuser'@'localhost' IDENTIFIED BY 'your_password';
GRANT ALL PRIVILEGES ON moodle.* TO 'moodleuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

## Paso 6: Descargar Moodle

```bash
cd /var/www/html
sudo git clone git://git.moodle.org/moodle.git
```

## Paso 7: Configurar Moodle

```bash
cd moodle
sudo git branch -r
sudo git checkout MOODLE_401_STABLE
```

## Paso 8: Configurar Permisos

```bash
sudo chown -R www-data:www-data /var/www/html/moodle
sudo chmod -R 755 /var/www/html/moodle
```

## Paso 9: Configurar Apache

Crea un archivo de configuración para Moodle:

```bash
sudo nano /etc/apache2/sites-available/moodle.conf
```

Agrega lo siguiente:

```apache
<VirtualHost *:80>
    ServerAdmin admin@example.com
    DocumentRoot /var/www/html/moodle
    ServerName your_domain.com

    <Directory /var/www/html/moodle>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/moodle_error.log
    CustomLog ${APACHE_LOG_DIR}/moodle_access.log combined
</VirtualHost>
```

Habilita el sitio y el módulo de reescritura:

```bash
sudo a2ensite moodle.conf
sudo a2enmod rewrite
sudo systemctl restart apache2
```

## Paso 10: Completar la Instalación

Accede a `http://your_domain.com` en tu navegador y sigue las instrucciones en pantalla para completar la instalación de Moodle.

## Conclusión

¡Moodle debería estar instalado y funcionando en tu servidor Ubuntu 24.04! Asegúrate de seguir las mejores prácticas de seguridad y realizar copias de seguridad periódicas.