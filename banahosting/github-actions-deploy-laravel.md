# GitHub Actions — Deploy Laravel por FTP (Banahosting)

Workflow de referencia para proyectos **Laravel**: tests, build de frontend, subida de `public/` por FTP.

**Antes**: hosting y cuenta FTP listos → [configurar-proyecto-ftp.md](./configurar-proyecto-ftp.md).

---

## Archivo: `.github/workflows/deploy.yml`

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
          node-version: '18'
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

## Secretos necesarios en GitHub

| Secret | Uso |
|--------|-----|
| `FTP_SERVER` | Host FTP (Banahosting) |
| `FTP_USERNAME` | Usuario de la cuenta FTP |
| `FTP_PASSWORD` | Contraseña FTP |
| `DB_PASSWORD` | MySQL en el job de tests |
| `DB_DATABASE` | Nombre de la base de datos de prueba |
| `DB_USER` | Usuario MySQL de prueba |

---

## Ajustes habituales

### `server-dir`

En el ejemplo está en `/` porque la cuenta FTP ya tiene como raíz la carpeta del proyecto (`/developer/mi-proyecto`). Si la raíz FTP es otra, cambia `server-dir` al subdirectorio correcto.

### Qué se sube

Este workflow sube el contenido de `public/` después del build. En un deploy Laravel completo en hosting compartido a veces hace falta:

- Subir también el resto del proyecto (fuera de `public/`) en otro job o con `local-dir` distinto
- O tener en el servidor la estructura Laravel con `public` como document root del subdominio

Documenta aquí la estructura que terminó funcionando en tu caso.

### Migraciones en el servidor

Por ahora **no está resuelto** en este repo. Ver intentos y notas en [github-actions-migraciones.md](./github-actions-migraciones.md).

---

## Notas de prueba (rellenar después de experimentar)

- **Fecha**:
- **Repositorio**:
- **Versión PHP en hosting**:
- **¿Corrieron los tests en CI?**:
- **¿Qué carpeta quedó como document root?**:
- **Qué funcionó**:
- **Qué no funcionó**:
