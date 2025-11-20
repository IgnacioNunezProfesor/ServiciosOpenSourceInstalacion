
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
cd /opt
sudo git clone https://github.com/wekan/wekan.git
cd wekan
sudo npm install
```

## Paso 6: Configurar Wekan

Crear archivo `.env`:

```bash
sudo nano .env
```

Agregar:

```
MONGO_URL=mongodb://localhost:27017/wekan
ROOT_URL=http://localhost:3000
PORT=3000
```

## Paso 7: Iniciar Wekan

```bash
sudo npm start
```

Acceder en: `http://localhost:3000`
