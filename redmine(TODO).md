# Instalación de Redmine en Ubuntu 24.04

## Requisitos previos
- Ubuntu 24.04 LTS
- Acceso root o sudo
- Conexión a Internet

## 1. Actualizar el sistema
```bash
sudo apt update
sudo apt upgrade -y
```

## 2. Instalar dependencias
```bash
sudo apt install -y build-essential libssl-dev libreadline-dev zlib1g-dev \
libsqlite3-dev libxml2-dev libxslt1-dev libffi-dev git curl \
libmysqlclient-dev imagemagick libmagickwand-dev mysql-server mysql-client

# Instalar rbenv (gestor Ruby)
git clone https://github.com/rbenv/rbenv.git ~/.rbenv
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(rbenv init -)"' >> ~/.bashrc
source ~/.bashrc

# Instalar ruby-build
git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build

# Instalar Ruby compatible con Redmine
rbenv install 3.1.3
rbenv global 3.1.3
rbenv rehash
```

## 3. Configurar MySQL
```bash
sudo mysql_secure_installation
mysql -u root -p
CREATE DATABASE redmine CHARACTER SET utf8mb4;
CREATE USER 'redmine'@'localhost' IDENTIFIED BY 'tu_contraseña';
GRANT ALL PRIVILEGES ON redmine.* TO 'redmine'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

## 4. Instalar Redmine
```bash
cd /opt
sudo wget https://www.redmine.org/releases/redmine-5.1.1.tar.gz
sudo tar xvf redmine-5.1.1.tar.gz
sudo mv redmine-5.1.1 redmine
sudo chown -R $USER:$USER /opt/redmine
sudo chown -R www-data:www-data /opt/redmine
```

## 5. Configurar Redmine
```bash
cd redmine
cp config/database.yml.example config/database.yml
```

Editar config/database.yml:
```yaml
production:
    adapter: mysql2
    database: redmine
    host: localhost
    username: redmine
    password: "tu_contraseña"
```

## 6. Instalar gemas y configurar la base de datos
```bash
gem install bundler
rbenv rehash
bundle config set --local without 'development test'
bundle install
bundle install
bundle exec rake generate_secret_token
RAILS_ENV=production bundle exec rake db:migrate
RAILS_ENV=production bundle exec rake redmine:load_default_data
```

## 7. Configurar Passenger y Nginx
```bash
sudo apt install -y apache2 libapache2-mod-passenger
sudo nano /etc/apache2/sites-available/redmine.conf
Contenido:
<VirtualHost *:80>
    ServerName redmine.local
    DocumentRoot /opt/redmine/public
    <Directory /opt/redmine/public>
        AllowOverride all
        Options -MultiViews
        Require all granted
    </Directory>
</VirtualHost>
Activación:
sudo a2ensite redmine.conf
sudo systemctl restart apache2
```
## 8. Configurar Firewall
```bash
sudo ufw allow 80
sudo ufw allow 443
sudo ufw enable
sudo ufw status
```

## 9. Iniciar servicios
```bash
sudo systemctl start nginx
sudo systemctl enable nginx
```

## 10. Acceder a Redmine
- Abrir navegador web
- Acceder a http://localhost
- Usuario por defecto: admin
- Contraseña por defecto: admin

## Notas de seguridad
- Cambiar la contraseña del administrador inmediatamente
- Configurar SSL/TLS para conexiones seguras
- Mantener el sistema actualizado regularmente