
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
sudo apt install -y curl wget git build-essential
```

## Paso 3: Instalar Node.js y npm

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
source ~/.bashrc
nvm install 14.21.3
nvm use 14.21.3
```

## Paso 4: Instalar MongoDB

```bash
sudo apt install -y mongodb-org
sudo apt install -y mongodb
sudo systemctl enable mongod
sudo systemctl start mongod

mongod --version
mongo --version
```

## Paso 5: Descargar e Instalar Wekan

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

```bash
sudo nano .env
```

Agregar:

```
MONGO_URL=mongodb://localhost:27017/wekan
ROOT_URL=http://<tu-IP>:3000
PORT=3000
NODE_ENV=production
```

## Paso 7: Iniciar Wekan

```bash
sudo npm start
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
