# Configurar un proyecto Python en un VPS

Guía base: servidor Ubuntu limpio → app Python corriendo (y opcionalmente detrás de Nginx).  
Ajusta nombres de usuario, rutas y puertos a tu caso.

**Relacionado:** [opciones-vps.md](./opciones-vps.md)

---

## Antes de empezar

Necesitas:

- IP del VPS
- Acceso SSH (usuario + clave o password)
- Tu código (GitHub/GitLab o subirlo con `scp`/`rsync`)
- Saber cómo arranca tu app (`uvicorn`, `gunicorn`, `flask run`, script, etc.)

Ejemplo usado en esta guía:

- Usuario: `deploy`
- App: FastAPI/Flask con Gunicorn o Uvicorn en el puerto `8000`
- Ruta: `/home/deploy/mi-app`

---

## 1. Conectar y actualizar el sistema

```bash
ssh root@TU_IP
# o: ssh usuario@TU_IP

apt update && apt upgrade -y
```

Si entras como `root`, crea un usuario de trabajo:

```bash
adduser deploy
usermod -aG sudo deploy
# Copia tu clave SSH a deploy (recomendado) antes de cerrar sesión root
```

Luego entra como ese usuario:

```bash
ssh deploy@TU_IP
```

---

## 2. Firewall básico (UFW)

```bash
sudo ufw allow OpenSSH
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
sudo ufw status
```

No abras el puerto de la app (`8000`) a internet si vas a usar Nginx como proxy.

---

## 3. Instalar Python y herramientas

```bash
sudo apt install -y python3 python3-pip python3-venv git curl
python3 --version
```

Opcional (build de algunas librerías):

```bash
sudo apt install -y build-essential libssl-dev libffi-dev
```

---

## 4. Clonar / subir el proyecto

Con Git:

```bash
cd ~
git clone https://github.com/TU_USUARIO/TU_REPO.git mi-app
cd mi-app
```

Sin Git (desde tu PC):

```bash
rsync -avz --exclude '.venv' --exclude '__pycache__' ./mi-app/ deploy@TU_IP:~/mi-app/
```

---

## 5. Entorno virtual e dependencias

```bash
cd ~/mi-app
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
```

Si no tienes `requirements.txt` aún:

```bash
pip freeze > requirements.txt
```

Variables de entorno (ejemplo):

```bash
nano ~/mi-app/.env
```

```env
# ejemplo
APP_ENV=production
SECRET_KEY=cambia-esto
DATABASE_URL=sqlite:///./app.db
```

**Importante:** no subas `.env` al repo. Añádelo a `.gitignore`.

---

## 6. Probar que arranca a mano

Ajusta al tipo de proyecto:

**FastAPI / Starlette (Uvicorn):**

```bash
source .venv/bin/activate
uvicorn main:app --host 127.0.0.1 --port 8000
```

**Flask (con Gunicorn):**

```bash
pip install gunicorn
gunicorn -b 127.0.0.1:8000 'app:app'
```

**Script simple:**

```bash
python main.py
```

Desde otra terminal en el VPS:

```bash
curl http://127.0.0.1:8000/
```

Si responde, la app está bien. Corta con `Ctrl+C`.

---

## 7. Servicio systemd (que reinicie solo)

Crea el unit file:

```bash
sudo nano /etc/systemd/system/mi-app.service
```

Ejemplo para Uvicorn:

```ini
[Unit]
Description=Mi app Python
After=network.target

[Service]
User=deploy
Group=deploy
WorkingDirectory=/home/deploy/mi-app
Environment="PATH=/home/deploy/mi-app/.venv/bin"
EnvironmentFile=/home/deploy/mi-app/.env
ExecStart=/home/deploy/mi-app/.venv/bin/uvicorn main:app --host 127.0.0.1 --port 8000
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Ejemplo para Gunicorn + Flask:

```ini
ExecStart=/home/deploy/mi-app/.venv/bin/gunicorn -b 127.0.0.1:8000 'app:app'
```

Activar:

```bash
sudo systemctl daemon-reload
sudo systemctl enable mi-app
sudo systemctl start mi-app
sudo systemctl status mi-app
```

Logs:

```bash
journalctl -u mi-app -f
```

---

## 8. Nginx como reverse proxy (opcional pero recomendado)

```bash
sudo apt install -y nginx
sudo nano /etc/nginx/sites-available/mi-app
```

```nginx
server {
    listen 80;
    server_name TU_DOMINIO_O_IP;

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
sudo rm -f /etc/nginx/sites-enabled/default
sudo nginx -t
sudo systemctl reload nginx
```

Prueba en el navegador: `http://TU_IP`

### HTTPS con Let's Encrypt (si tienes dominio)

```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d tudominio.com
```

---

## 9. Actualizar el proyecto después

```bash
cd ~/mi-app
git pull
source .venv/bin/activate
pip install -r requirements.txt
sudo systemctl restart mi-app
```

---

## Problemas frecuentes

| Síntoma | Qué revisar |
| --- | --- |
| `Permission denied` en SSH | Clave, usuario, `~/.ssh/authorized_keys` |
| App cae al cerrar la terminal | Falta systemd (o usaste solo `python` en foreground) |
| `502 Bad Gateway` en Nginx | App no está en `:8000`; `systemctl status mi-app` |
| `ModuleNotFoundError` | Activaste el venv / `PATH` del service apunta al venv |
| Puerto no responde desde fuera | Firewall; o estás bindeando solo a `127.0.0.1` (correcto si usas Nginx) |
| ARM (Oracle) y paquete falla | Algunas wheels no existen para aarch64; puede hacer falta compilar |

---

## Checklist rápido

- [ ] Ubuntu actualizado
- [ ] Usuario no-root + SSH
- [ ] UFW (22/80/443)
- [ ] Python + venv + `requirements.txt`
- [ ] App responde en `127.0.0.1:PUERTO`
- [ ] systemd enable + start
- [ ] Nginx proxy (opcional)
- [ ] HTTPS si hay dominio (opcional)

---

## Notas de prueba

> Anota aquí lo que cambió en tu caso (versión de Python, framework, errores raros, pasos extra).

- Fecha:
- Distro / versión:
- Framework (Flask, FastAPI, Django, otro):
- Cómo arranca la app:
- Qué funcionó:
- Qué no funcionó / paso adicional:
- Comandos o configs finales que quedaron:
