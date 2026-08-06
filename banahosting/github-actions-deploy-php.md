# GitHub Actions — Deploy PHP puro por FTP (Banahosting)

Workflow de referencia para proyectos **solo PHP** (sin Laravel, sin Composer obligatorio): checkout del repo y subida por FTP.

**Antes**: hosting y cuenta FTP listos → [configurar-proyecto-ftp.md](./configurar-proyecto-ftp.md).

---

## Archivo: `.github/workflows/deploy.yml`

```yaml
name: Deploy PHP to Banahosting

on:
  push:
    branches: "main"

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Deploy to Server via FTP
        uses: SamKirkland/FTP-Deploy-Action@v4.3.5
        with:
          server: ${{ secrets.FTP_SERVER }}
          username: ${{ secrets.FTP_USERNAME }}
          password: ${{ secrets.FTP_PASSWORD }}
          server-dir: /
          # Opcional: excluir archivos que no deben ir al servidor
          # exclude: |
          #   **/.git*
          #   **/.github/**
          #   **/README.md
```

---

## Secretos en GitHub

| Secret | Valor |
|--------|--------|
| `FTP_SERVER` | Host de "Configurar cliente de FTP" en cPanel |
| `FTP_USERNAME` | Usuario FTP |
| `FTP_PASSWORD` | Contraseña FTP |

---

## Estructura típica del proyecto PHP

El contenido que debe quedar en la raíz FTP (`/developer/mi-proyecto/`) suele ser:

```
mi-proyecto/
  index.php
  assets/
  includes/
  config.php      ← no subir secretos reales al repo; usar .env en servidor si aplica
```

Asegúrate de que `index.php` (o el entry point) esté en la **raíz** de lo que sube el FTP, no dentro de una subcarpeta extra del repo.

---

## Ajustes según el caso

### Subir solo una carpeta del repo

Si el código público está en `public/` o `dist/`:

```yaml
with:
  local-dir: ./public/
  server-dir: /
```

### Excluir archivos de desarrollo

```yaml
exclude: |
  **/.git*
  **/.github/**
  **/node_modules/**
  **/*.md
```

### PHP con Composer (sin Laravel)

Si el proyecto usa Composer pero no es Laravel:

```yaml
steps:
  - uses: actions/checkout@v4
  - uses: shivammathur/setup-php@v2
    with:
      php-version: "8.2"
  - run: composer install --no-dev --optimize-autoloader
  - uses: SamKirkland/FTP-Deploy-Action@v4.3.5
    with:
      server: ${{ secrets.FTP_SERVER }}
      username: ${{ secrets.FTP_USERNAME }}
      password: ${{ secrets.FTP_PASSWORD }}
      server-dir: /
```

---

## Notas de prueba (rellenar después de experimentar)

- **Fecha**:
- **Repositorio**:
- **Ruta FTP real**:
- **¿`local-dir` necesario?**:
- **Qué funcionó**:
- **Qué no funcionó**:
- **Archivos excluidos del deploy**:
