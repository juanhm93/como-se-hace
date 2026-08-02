# Configurar un proyecto Python en un VPS

> **Notas personales:** Anota aquí los pasos que te funcionaron, errores que tuviste y pasos adicionales que descubriste.
>
> - Fecha:
> - VPS (proveedor / IP):
> - Versión de Python:
> - Observaciones:

---

## Requisitos previos

- Un VPS con **Ubuntu 22.04 o 24.04** (recomendado)
- IP pública del servidor
- Acceso SSH (usuario `root` o usuario con sudo)
- Tu proyecto Python (local o en GitHub/GitLab)

---

## 1. Conectarse al VPS por SSH

Desde tu máquina local:

```bash
ssh root@TU_IP_DEL_VPS
```

Si usas clave SSH:

```bash
ssh -i ~/.ssh/tu_clave root@TU_IP_DEL_VPS
```

**Primera vez:** El sistema pregunta si confías en el host — escribe `yes`.

### (Opcional) Crear un usuario no-root

```bash
adduser deploy
usermod -aG sudo deploy
rsync --archive --chown=deploy:deploy ~/.ssh /home/deploy
```

Luego conéctate con: `ssh deploy@TU_IP_DEL_VPS`

---

## 2. Actualizar el sistema

```bash
sudo apt update && sudo apt upgrade -y
```

---

## 3. Instalar Python y herramientas básicas

Ubuntu 22.04/24.04 ya trae Python 3. Verifica:

```bash
python3 --version
```

Instala pip, venv y utilidades:

```bash
sudo apt install -y python3-pip python3-venv git curl build-essential
```

---

## 4. Subir el proyecto al VPS

### Opción A: Clonar desde Git (recomendado)

```bash
cd ~
git clone https://github.com/TU_USUARIO/TU_PROYECTO.git
cd TU_PROYECTO
```

### Opción B: Copiar con SCP desde tu PC

Desde tu máquina local:

```bash
scp -r ./mi-proyecto deploy@TU_IP_DEL_VPS:~/
```

---

## 5. Crear entorno virtual e instalar dependencias

```bash
cd ~/TU_PROYECTO
python3 -m venv venv
source venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
```

Si no tienes `requirements.txt`, créalo en local:

```bash
pip freeze > requirements.txt
```

### Probar que funciona

```bash
python main.py
# o
python -m tu_paquete
# o para FastAPI/Flask:
uvicorn main:app --host 0.0.0.0 --port 8000
```

---

## 6. Variables de entorno

Crea un archivo `.env` en el proyecto (no lo subas a Git):

```bash
nano ~/TU_PROYECTO/.env
```

Ejemplo:

```env
DATABASE_URL=postgresql://user:pass@localhost/midb
SECRET_KEY=tu-clave-secreta-muy-larga
DEBUG=False
```

Carga las variables en tu app con `python-dotenv` o la librería que uses.

---

## 7. Ejecutar la app en segundo plano (systemd)

Para que la app arranque sola al reiniciar el servidor.

### Crear el servicio

```bash
sudo nano /etc/systemd/system/mi-app.service
```

Contenido de ejemplo (ajusta rutas y comando):

```ini
[Unit]
Description=Mi aplicación Python
After=network.target

[Service]
User=deploy
Group=deploy
WorkingDirectory=/home/deploy/TU_PROYECTO
Environment="PATH=/home/deploy/TU_PROYECTO/venv/bin"
ExecStart=/home/deploy/TU_PROYECTO/venv/bin/uvicorn main:app --host 127.0.0.1 --port 8000
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

> **Nota:** Si usas Flask: `ExecStart=.../venv/bin/gunicorn -w 4 -b 127.0.0.1:8000 app:app`
> Si es un script: `ExecStart=.../venv/bin/python main.py`

### Activar el servicio

```bash
sudo systemctl daemon-reload
sudo systemctl enable mi-app
sudo systemctl start mi-app
sudo systemctl status mi-app
```

Ver logs:

```bash
sudo journalctl -u mi-app -f
```

---

## 8. (Opcional) Nginx como proxy inverso

Permite acceder por el puerto 80/443 y usar dominio con HTTPS.

### Instalar Nginx

```bash
sudo apt install -y nginx
```

### Configurar sitio

```bash
sudo nano /etc/nginx/sites-available/mi-app
```

```nginx
server {
    listen 80;
    server_name tudominio.com www.tudominio.com;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Activar y recargar:

```bash
sudo ln -s /etc/nginx/sites-available/mi-app /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

### HTTPS con Let's Encrypt (Certbot)

```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d tudominio.com -d www.tudominio.com
```

---

## 9. Firewall básico

```bash
sudo ufw allow OpenSSH
sudo ufw allow 'Nginx Full'
sudo ufw enable
sudo ufw status
```

> Si no usas Nginx y expones el puerto directo: `sudo ufw allow 8000`

---

## 10. Actualizar el proyecto (despliegues futuros)

```bash
cd ~/TU_PROYECTO
git pull
source venv/bin/activate
pip install -r requirements.txt
sudo systemctl restart mi-app
```

---

## Estructura típica en el VPS

```
/home/deploy/
└── TU_PROYECTO/
    ├── venv/
    ├── .env
    ├── requirements.txt
    ├── main.py
    └── ...
```

---

## Problemas comunes

| Problema | Posible solución |
|----------|------------------|
| `Permission denied` al conectar SSH | Verificar clave SSH o contraseña; revisar `~/.ssh/authorized_keys` |
| `pip install` falla por paquetes C | `sudo apt install build-essential python3-dev libpq-dev` (ajusta según librería) |
| App no accesible desde fuera | Revisar firewall (`ufw`), que la app escuche en `0.0.0.0` o usar Nginx |
| `systemctl` falla al iniciar | `sudo journalctl -u mi-app -n 50` para ver el error |
| Puerto 80 ocupado | `sudo lsof -i :80` o usar otro puerto |

---

## Checklist de despliegue

- [ ] SSH configurado con clave (sin contraseña root)
- [ ] Sistema actualizado
- [ ] Python + venv + dependencias instaladas
- [ ] Variables de entorno configuradas
- [ ] App corre manualmente
- [ ] Servicio systemd creado y activo
- [ ] Nginx configurado (si aplica)
- [ ] HTTPS activo (si aplica)
- [ ] Firewall configurado

---

## Mis notas y pasos adicionales

<!-- Escribe aquí lo que descubriste al probar: -->

### Lo que funcionó tal cual

-

### Pasos adicionales que tuve que hacer

-

### Errores que encontré y cómo los resolví

| Error | Solución |
|-------|----------|
| | |

### Comandos útiles que guardo

```bash
# Ejemplo: reiniciar app
sudo systemctl restart mi-app
```
