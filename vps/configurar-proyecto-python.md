# Configurar un proyecto Python en un VPS

Guía base para subir y dejar corriendo un proyecto Python en un VPS (Ubuntu). Ajústala según tu app (Flask, FastAPI, Django, script, bot, etc.).

**Antes**: elige y crea el VPS → [Opciones de VPS](./opciones-vps.md).

---

## 0. Lo que vas a tener al final

- Acceso por SSH
- Python + entorno virtual
- Código del proyecto en el servidor
- Dependencias instaladas
- (Opcional) servicio systemd para que arranque solo
- (Opcional) Nginx como proxy inverso + HTTPS

---

## 1. Conectarte al VPS

Desde tu máquina:

```bash
ssh root@IP_DEL_VPS
# o, si creaste un usuario:
ssh usuario@IP_DEL_VPS
```

Si usas llave SSH:

```bash
ssh -i ~/.ssh/tu_llave usuario@IP_DEL_VPS
```

---

## 2. Actualizar el sistema y paquetes básicos

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y git curl ufw build-essential
```

Firewall básico (ajusta si usas otros puertos):

```bash
sudo ufw allow OpenSSH
sudo ufw allow 80
sudo ufw allow 443
sudo ufw enable
sudo ufw status
```

---

## 3. Crear un usuario (recomendado si entraste como root)

```bash
adduser deploy
usermod -aG sudo deploy
# Copia tu llave SSH a /home/deploy/.ssh/authorized_keys
su - deploy
```

---

## 4. Instalar Python

En Ubuntu reciente suele venir Python 3. Si no:

```bash
sudo apt install -y python3 python3-pip python3-venv
python3 --version
```

---

## 5. Traer el proyecto

### Opción A — con Git (recomendado)

```bash
mkdir -p ~/apps
cd ~/apps
git clone https://github.com/TU_USUARIO/TU_REPO.git
cd TU_REPO
```

### Opción B — subir archivos con `scp` / `rsync`

Desde tu PC:

```bash
scp -r ./mi-proyecto usuario@IP_DEL_VPS:~/apps/
# o
rsync -avz ./mi-proyecto usuario@IP_DEL_VPS:~/apps/
```

---

## 6. Entorno virtual e instalar dependencias

```bash
cd ~/apps/TU_REPO
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
```

Si no tienes `requirements.txt` aún:

```bash
pip freeze > requirements.txt
```

---

## 7. Variables de entorno

No subas secretos al repo. En el VPS:

```bash
nano ~/apps/TU_REPO/.env
```

Ejemplo:

```env
APP_ENV=production
SECRET_KEY=cambia-esto
DATABASE_URL=sqlite:///./app.db
PORT=8000
```

Carga el `.env` desde tu app (p. ej. `python-dotenv`) o exporta en el servicio systemd.

Asegura permisos:

```bash
chmod 600 ~/apps/TU_REPO/.env
```

---

## 8. Probar que corre

Ejemplos según el tipo de app:

```bash
# Script
python main.py

# FastAPI / Starlette con Uvicorn
uvicorn app.main:app --host 0.0.0.0 --port 8000

# Flask con Gunicorn
gunicorn -b 0.0.0.0:8000 "app:create_app()"

# Django
python manage.py migrate
gunicorn -b 0.0.0.0:8000 mi_proyecto.wsgi:application
```

Prueba desde tu navegador o con:

```bash
curl http://IP_DEL_VPS:8000
```

Si no abre, revisa `ufw` y que el proceso escuche en `0.0.0.0` (no solo `127.0.0.1`).

---

## 9. Dejarlo corriendo con systemd

Crea un servicio para que sobreviva al cierre de SSH y al reinicio:

```bash
sudo nano /etc/systemd/system/mi-app.service
```

Contenido ejemplo (FastAPI/Uvicorn):

```ini
[Unit]
Description=Mi app Python
After=network.target

[Service]
User=deploy
Group=deploy
WorkingDirectory=/home/deploy/apps/TU_REPO
Environment="PATH=/home/deploy/apps/TU_REPO/.venv/bin"
EnvironmentFile=/home/deploy/apps/TU_REPO/.env
ExecStart=/home/deploy/apps/TU_REPO/.venv/bin/uvicorn app.main:app --host 127.0.0.1 --port 8000
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Activar:

```bash
sudo systemctl daemon-reload
sudo systemctl enable mi-app
sudo systemctl start mi-app
sudo systemctl status mi-app
sudo journalctl -u mi-app -f
```

---

## 10. (Opcional) Nginx como proxy + dominio

```bash
sudo apt install -y nginx
sudo nano /etc/nginx/sites-available/mi-app
```

```nginx
server {
    listen 80;
    server_name tu-dominio.com;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

```bash
sudo ln -s /etc/nginx/sites-available/mi-app /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

HTTPS con Let’s Encrypt:

```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d tu-dominio.com
```

---

## 11. Checklist rápido de actualización

Cuando cambies código:

```bash
cd ~/apps/TU_REPO
git pull
source .venv/bin/activate
pip install -r requirements.txt
sudo systemctl restart mi-app
sudo systemctl status mi-app
```

---

## Problemas frecuentes

| Síntoma | Qué revisar |
|---------|-------------|
| `Permission denied (publickey)` | Llave SSH, usuario, `authorized_keys`, permisos `~/.ssh` (700) y `authorized_keys` (600). |
| No abre el puerto | `ufw`, security group del proveedor, app escuchando en `0.0.0.0` o detrás de Nginx. |
| `ModuleNotFoundError` | ¿Activaste el venv? ¿Instalaste `requirements.txt` en ese venv? |
| Servicio cae al instante | `journalctl -u mi-app -n 50` — suele ser ruta mal, `.env` faltante o comando `ExecStart` incorrecto. |
| 502 Bad Gateway en Nginx | La app no está arriba o el puerto en `proxy_pass` no coincide. |

---

## Notas de prueba (rellenar después de experimentar)

> Cuando algo no coincida con esta guía, anótalo aquí para la próxima vez.

- **Fecha**:
- **Proveedor / OS**:
- **Tipo de proyecto** (Flask / FastAPI / Django / script):
- **Qué funcionó**:
- **Qué no funcionó**:
- **Pasos extra**:
- **Comandos finales que sí quedaron**:
