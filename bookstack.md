# Instalación de BookStack en Ubuntu Server 24.04

Requisitos mínimos: Ubuntu 24.04, acceso root o usuario con sudo, dominio (recomendado).

## 1) Actualizar sistema
```bash
sudo apt update && sudo apt upgrade -y
```

## 2) Instalar dependencias
```bash
sudo apt install -y nginx mariadb-server git composer \
 php8.3-fpm php8.3-mysql php8.3-xml php8.3-mbstring php8.3-curl \
 php8.3-gd php8.3-zip php8.3-bcmath php8.3-intl unzip \
 certbot python3-certbot-nginx supervisor
```

(Ajustar versión de PHP si es distinta en tu repositorio)

## 3) Configurar base de datos (MariaDB)
```bash
sudo mysql -u root
```
Dentro del prompt de MariaDB:
```sql
CREATE DATABASE bookstack CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'bookstack'@'localhost' IDENTIFIED BY 'TuPasswordSeguro';
GRANT ALL PRIVILEGES ON bookstack.* TO 'bookstack'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

## 4) Descargar BookStack y dependencias PHP
```bash
sudo mkdir -p /var/www
cd /var/www
sudo git clone https://github.com/BookStackApp/BookStack.git --branch=release --single-branch bookstack
cd bookstack
sudo cp .env.example .env
sudo composer install --no-dev --optimize-autoloader
```

Editar `.env` (APP_URL, DB_CONNECTION, DB_HOST, DB_DATABASE, DB_USERNAME, DB_PASSWORD). Ejemplo:
```
APP_URL=https://tu-dominio.com
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_DATABASE=bookstack
DB_USERNAME=bookstack
DB_PASSWORD=TuPasswordSeguro
```

Generar clave y migrar:
```bash
sudo php artisan key:generate --force
sudo php artisan migrate --seed --force
```

Ajustar permisos:
```bash
sudo chown -R www-data:www-data /var/www/bookstack
sudo chmod 640 /var/www/bookstack/.env
sudo find /var/www/bookstack -type f -exec chmod 640 {} \;
sudo find /var/www/bookstack -type d -exec chmod 750 {} \;
```

## 5) Configurar Nginx (ejemplo)
Crear `/etc/nginx/sites-available/bookstack`:
```nginx
server {
    listen 80;
    server_name tu-dominio.com;

    root /var/www/bookstack/public;
    index index.php;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass unix:/run/php/php8.3-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param HTTP_PROXY ""; # evita problemas
    }

    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
        try_files $uri =404;
        expires 1M;
        access_log off;
    }
}
```
Habilitar y recargar:
```bash
sudo rm /etc/nginx/sites-enabled/default
sudo ln -s /etc/nginx/sites-available/bookstack /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx
```

## 6) Habilitar HTTPS (Let’s Encrypt)
```bash
sudo certbot --nginx -d tu-dominio.com -d www.tu-dominio.com
```

## 7) Configurar worker (opcional, recomendado para emails/colas)
Crear `/etc/supervisor/conf.d/bookstack-worker.conf`:
```
[program:bookstack-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /var/www/bookstack/artisan queue:work --sleep=3 --tries=3 --timeout=90
user=www-data
numprocs=1
autostart=true
autorestart=true
redirect_stderr=true
stdout_logfile=/var/log/bookstack-worker.log
```
Recargar supervisor:
```bash
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start bookstack-worker:*
```

## 8) Pruebas y solución de problemas
- Acceder a https://tu-dominio.com
- Logs: /var/www/bookstack/storage/logs/laravel.log, /var/log/nginx/error.log, /var/log/php8.3-fpm.log
- Comprobar permisos y configuración de `.env`.

Fin.