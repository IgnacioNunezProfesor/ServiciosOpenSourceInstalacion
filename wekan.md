
# Instalación de Wekan en Ubuntu Server 24.04

## Requisitos Previos

- Ubuntu Server 24.04 LTS
- Usuario con permisos sudo
- Conexión a internet

## Paso 1: Actualizar el Sistema

```bash
sudo apt update
sudo apt upgrade -y
```

## Paso 2: Instalar Dependencias

```bash
sudo apt install -y curl wget git build-essential gnupg2 ca-certificates lsb-release snapd
```

## Paso 3: Instalar Node.js y npm
### Instalación de Node.js (OPCIONAL — sólo si vas a compilar Wekan manualmente)
### Si vas a instalar Wekan mediante Snap, puedes omitir este paso.
### Para instalar Node.js 20:

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
source ~/.bashrc
nvm install 14.21.3
nvm use 14.21.3
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs
node -v
npm -v
```

## Paso 4: Instalar MongoDB
### Añadir clave GPG y repo oficial de MongoDB (Ubuntu 24.04 "noble")

```bash
sudo apt install -y mongodb-org
sudo apt install -y mongodb
sudo systemctl enable mongod
sudo systemctl start mongod

mongod --version
mongo --version
curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc \
  | gpg --dearmor \
  | sudo tee /usr/share/keyrings/mongodb-archive-keyring.gpg > /dev/null

echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-archive-keyring.gpg ] https://repo.mongodb.org/apt/ubuntu noble/mongodb-org/7.0 multiverse" \
  | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list

sudo apt update
sudo apt install -y mongodb-org

sudo systemctl enable --now mongod
sudo systemctl status mongod --no-pager
```

## Paso 5: Descargar e Instalar Wekan
### Instalación recomendada de Wekan (Snap)
### Snap instala WeKan como un servicio gestionado y con actualizaciones automáticas.

```bash
sudo mkdir -p /opt/wekan
sudo unzip wekan-8.16-amd64.zip -d /opt/wekan
cd /opt/wekan/bundle/programs/server
npm install
```

## Paso 6: Configurar Wekan

Crear usuario de servicio:
```bash
sudo adduser --system --no-create-home wekan
sudo chown -R wekan:wekan /opt/wekan
```

Crear archivo `.env`:
sudo snap install wekan --channel=latest/candidate
cd /opt
sudo wget https://releases.wekan.team/wekan-latest.zip
sudo apt install -y unzip
sudo unzip wekan-latest.zip -d wekan
cd wekan
sudo npm install --production
```

## Paso 6: Configurar Wekan
### Configurar URL y puerto (ejemplo)

```bash
sudo snap set wekan root-url="http://TU_IP_O_DOMINIO"
```

### Para servir en puerto 80 (HTTP) (si quieres usar 80 en lugar de 3000)

```
MONGO_URL=mongodb://localhost:27017/wekan
ROOT_URL=http://<tu-IP>:3000
PORT=3000
NODE_ENV=production
```bash
sudo snap set wekan port='80'
```

## Paso 7: Iniciar Wekan
### Comprobar estado del servicio snap

```bash
sudo snap services wekan
```

## Paso 8: Configurar Wekan como servicio systemd

Crear archivo systemd:

```bash
sudo nano /etc/systemd/system/wekan.service
```

Agregar:
```bash
[Unit]
Description=Wekan Kanban Server
Acceder en: `http://localhost:3000`

## Paso 8: Verificación y firewall

### Verificar que Wekan está corriendo (snap o manual)
```bash
ss -tulpn | grep -E '3000|80|443' || sudo snap services wekan
```

### Si usas UFW y ejecutas Wekan en 3000:
```bash
sudo ufw allow 3000/tcp
```

### Si usas proxy inverso (Nginx) o snap configurado para puerto 80/443:
```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

# Acceso:
### Si accedes desde la misma máquina: http://localhost:3000 (o http://localhost si configuraste puerto 80)
### Desde otra máquina: http://IP_DEL_SERVIDOR:3000  (o http://IP_DEL_SERVIDOR si usas puerto 80)
```MONGO_URL=mongodb://localhost:27017/wekan
MONGO_OPLOG_URL=mongodb://localhost:27017/local?replicaSet=rs0
ROOT_URL=http://localhost:8080
PORT=8080
MAIL_URL=smtp://localhost
```
## Paso 7: ACTIVAR REPLICASET

```bash
sudo systemctl stop mongod
sudo mongod --dbpath /var/lib/mongodb --replSet rs0 --bind_ip_all &
sleep 5
mongosh --eval "rs.initiate()"
sudo systemctl start mongod
```

## Paso 8: Iniciar Wekan

```bash
sudo nano /etc/systemd/system/wekan.service

Contenido:
[Unit]
Description=Wekan Server
After=network.target mongod.service

[Service]
Type=simple
User=wekan
Group=wekan
WorkingDirectory=/opt/wekan/bundle

ExecStart=/usr/bin/node main.js

Environment="PORT=3000"
Environment="ROOT_URL=http://<tu-IP>:3000"
Environment="MONGO_URL=mongodb://localhost:27017/wekan"
Environment="NODE_ENV=production"

Restart=always
RestartSec=10
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=wekan

[Install]
WantedBy=multi-user.target
```
Ejecutamos:
```bash
sudo systemctl daemon-reload
sudo systemctl enable wekan
sudo systemctl start wekan
sudo systemctl status wekan
```

Acceder en: `http://<tu-IP>:3000`
EnvironmentFile=/opt/wekan/.env
ExecStart=/usr/bin/node /opt/wekan/main.js
Restart=always

[Install]
WantedBy=multi-user.target

Habilitar:
sudo systemctl daemon-reload
sudo systemctl enable --now wekan
```
## Paso 9: Configuración mínima de Nginx
```bash
server {
    listen 80;
    server_name tu_dominio.com;

    location / {
        proxy_pass http://127.0.0.1:8080/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```
Acceder en: `http://localhost:8080`
