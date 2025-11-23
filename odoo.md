# Instalación de Odoo en Ubuntu 24.04

## Requisitos previos
- Ubuntu 24.04 LTS
- Acceso root o privilegios sudo
- Conexión a Internet

## 1. Actualizar el sistema
```bash
sudo apt update
sudo apt upgrade -y
```

## 2. Instalar dependencias
```bash
sudo apt install git python3-pip python3-dev python3-venv python3-wheel libxslt1-dev libzip-dev libldap2-dev libsasl2-dev python3-setuptools node-less libjpeg-dev zlib1g-dev libpq-dev libxml2-dev libxslt1-dev libldap2-dev libtiff5-dev libjpeg8-dev libopenjp2-7-dev liblcms2-dev libwebp-dev libharfbuzz-dev libfribidi-dev libxcb1-dev wget -y
```
## 3. Instalar nodejs y npm
```bash
sudo apt -y install nodejs npm
```
## 4. intalacion de wkhtmltopdf para reportes automanticos en pdf
```bash
sudo wget http://security.ubuntu.com/ubuntu/pool/universe/w/wkhtmltopdf/wkhtmltopdf_0.12.6-2build2_amd64.deb
dpkg -i wkhtmltopdf_0.12.6-2build2_amd64.deb
apt-get install -f
dpkg -i wkhtmltopdf_0.12.6-2build2_amd64.deb
wkhtmltopdf --version
```
## 5. Crear usuario Odoo
```bash
sudo useradd -m -d /opt/odoo -U -r -s /bin/bash odoo
```

## 6. Instalar PostgreSQL
```bash
sudo apt install postgresql -y
sudo systemctl start postgresql
sudo systemctl enable postgresql
sudo systemctl status postgresql
sudo su - postgres -c "createuser -s odoo"
```

## 7. Instalar Odoo desde GitHub
```bash
sudo git clone --depth 1 --branch 17.0 https://www.github.com/odoo/odoo /opt/odoo/odoo
sudo chown -R odoo:odoo /opt/odoo
```

## 8. Crear entorno virtual Python
```bash
sudo su - odoo -c "python3 -m venv /opt/odoo/venv"
sudo su - odoo -c "/opt/odoo/venv/bin/pip install -r /opt/odoo/odoo/requirements.txt"
```

## 9. Configurar Odoo
```bash
sudo mkdir /etc/odoo
sudo cp /opt/odoo/odoo/debian/odoo.conf /etc/odoo/odoo.conf
sudo chown odoo:odoo /etc/odoo/odoo.conf
```

## 10. Crear servicio systemd
```bash
sudo tee /etc/systemd/system/odoo.service << EOF
[Unit]
Description=Odoo
After=network.target

[Service]
Type=simple
User=odoo
Group=odoo
ExecStart=/opt/odoo/venv/bin/python3 /opt/odoo/odoo/odoo-bin -c /etc/odoo/odoo.conf
Restart=always

[Install]
WantedBy=multi-user.target
EOF
```

## 11. Iniciar Odoo
```bash
sudo systemctl daemon-reload
sudo systemctl start odoo
sudo systemctl enable odoo
```

## 12. Acceder a Odoo
Abrir navegador web y acceder a:
```
http://localhost:8069
```

## Notas adicionales
- Puerto por defecto: 8069
- Usuario admin por defecto: admin
- La primera vez que acceda deberá crear la contraseña de administrador
- Los logs se pueden consultar con: `sudo journalctl -u odoo -f`