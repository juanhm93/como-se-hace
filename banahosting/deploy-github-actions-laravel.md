# Deploy con GitHub Actions (proyecto Laravel)

Configuración de CI/CD para un proyecto **Laravel** en BanaHosting por FTP.  
Este archivo queda separado del flujo de PHP puro: [Deploy PHP](./deploy-github-actions-php.md).

**Antes**: termina [Configurar proyecto por FTP](./configurar-proyecto-ftp.md).

---

## 1. Secrets necesarios en GitHub

| Secret | Descripción |
|--------|-------------|
| `FTP_SERVER` | Host del servidor FTP |
| `FTP_USERNAME` | Usuario de la cuenta FTP |
| `FTP_PASSWORD` | Contraseña de la cuenta FTP |
| `DB_DATABASE` | Nombre de la BD (para el job de tests con MySQL) |
| `DB_USER` | Usuario de la BD |
| `DB_PASSWORD` | Contraseña de la BD |

> Los secrets de BD en este workflow se usan sobre todo para el servicio MySQL del runner de GitHub Actions (tests locales al CI), no necesariamente para la BD del hosting.

---

## 2. Archivo del workflow

Crea:

```text
.github/workflows/deploy.yml
```

Configuración base (referencia / trabajo en curso):

```yaml
name: Continuous Integration and Deployment

on:
  push:
    branches: "main"

jobs:
  laravel-tests:
    runs-on: ubuntu-latest

    services:
      mysql:
        image: mysql:8.0
        env:
          MYSQL_ROOT_PASSWORD: ${{ secrets.DB_PASSWORD }}
          MYSQL_DATABASE: ${{ secrets.DB_DATABASE }}
          MYSQL_USER: ${{ secrets.DB_USER }}
          MYSQL_PASSWORD: ${{ secrets.DB_PASSWORD }}
        ports:
          - 3306:3306
        options: --health-cmd="mysqladmin ping --password=root_password" --health-interval=10s --health-timeout=5s --health-retries=3

    steps:
      - uses: shivammathur/setup-php@15c43e89cdef867065b0213be354c2841860869e
        with:
          php-version: "8.2"
      - uses: actions/checkout@v3
      - name: Copy .env
        run: php -r "file_exists('.env') || copy('.env.example', '.env');"
      - name: Install Dependencies
        run: |
          composer install -q --no-ansi --no-interaction --no-scripts --no-progress --prefer-dist
      - name: Clear Config and Cache
        run: |
          php artisan config:clear
          php artisan cache:clear
          php artisan route:clear
      - name: Set Directory Permissions
        run: chmod -R 777 storage bootstrap/cache
      - name: Install Node.js
        uses: actions/setup-node@v3
        with:
          node-version: "18"
      - name: Install npm Dependencies
        run: |
          npm install -g npm@latest
          npm cache clean --force
      - name: Build Frontend Assets
        run: npm run build
      - name: Upload Artifact
        uses: actions/upload-artifact@v3
        with:
          name: dist
          path: public/
      - name: Deploy to Server
        uses: SamKirkland/FTP-Deploy-Action@v4.3.5
        with:
          server: ${{ secrets.FTP_SERVER }}
          username: ${{ secrets.FTP_USERNAME }}
          password: ${{ secrets.FTP_PASSWORD }}
          server-dir: /
```

---

## 3. Migraciones (pendiente / no logrado)

Se intentó agregar pasos para correr migraciones en el flujo (o vía SSH / artisan remoto). **Por ahora no se logró que funcione** de forma estable en este setup con solo FTP.

Ideas a probar más adelante (sin marcarlas como listas):

- SSH + `php artisan migrate --force` en el servidor (requiere acceso SSH, no solo FTP)
- Script post-deploy en el hosting
- Endpoint / comando protegido que ejecute migraciones (solo si se controla bien la seguridad)

Hasta que eso quede resuelto, las migraciones se corren **a mano** en el servidor (Terminal de cPanel, SSH, o local apuntando a la BD remota con cuidado).

---

## 4. Qué hace este workflow (resumen)

1. Levanta MySQL en el runner (servicio del job).
2. Instala PHP 8.2, Composer y dependencias.
3. Limpia config/cache/rutas de Artisan.
4. Instala Node, hace `npm run build`.
5. Sube artefactos de `public/` (artifact opcional).
6. Despliega el proyecto por FTP a BanaHosting.

---

## Notas de prueba (rellenar después de experimentar)

> Usa esta sección cuando pruebes algo y falle, cambie, o falte un paso.

- **Fecha**:
- **Repo / subdominio**:
- **Qué funcionó**:
- **Qué no funcionó**:
  - Migraciones automáticas: no logradas (ver sección 3)
- **Pasos extra que hiciste**:
- **Ajustes al `health-cmd` de MySQL / secrets de BD**:
- **Enlaces útiles**:
