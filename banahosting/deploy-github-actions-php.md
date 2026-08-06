# Deploy con GitHub Actions (proyecto PHP)

Configuración de CI/CD para un proyecto **solo PHP** en BanaHosting, desplegando por FTP al push a `main`.

**Antes**: termina [Configurar proyecto por FTP](./configurar-proyecto-ftp.md) (subdominio, carpeta, cuenta FTP y secrets).

---

## 1. Secrets necesarios en GitHub

| Secret | Descripción |
|--------|-------------|
| `FTP_SERVER` | Host del servidor FTP |
| `FTP_USERNAME` | Usuario de la cuenta FTP |
| `FTP_PASSWORD` | Contraseña de la cuenta FTP |

---

## 2. Archivo del workflow

Crea el archivo en el repo del proyecto:

```text
.github/workflows/deploy.yml
```

Contenido base (PHP, sin Laravel / Composer / npm):

```yaml
name: Continuous Integration and Deployment

on:
  push:
    branches: ["main"]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Deploy to Server
        uses: SamKirkland/FTP-Deploy-Action@v4.3.5
        with:
          server: ${{ secrets.FTP_SERVER }}
          username: ${{ secrets.FTP_USERNAME }}
          password: ${{ secrets.FTP_PASSWORD }}
          server-dir: /
```

Notas:

- `server-dir: /` asume que la cuenta FTP ya apunta a la carpeta del proyecto (`developer/mi-proyecto`). No hace falta subir a otra ruta.
- Si el proyecto tiene archivos que **no** deben subirse (`.git`, `.env`, tests, etc.), agrega un `.ftp-deploy-sync-state.json` / exclusiones según la acción, o un archivo `.git-ftp-ignore` / `exclude` del action. Ejemplo con exclusiones:

```yaml
      - name: Deploy to Server
        uses: SamKirkland/FTP-Deploy-Action@v4.3.5
        with:
          server: ${{ secrets.FTP_SERVER }}
          username: ${{ secrets.FTP_USERNAME }}
          password: ${{ secrets.FTP_PASSWORD }}
          server-dir: /
          exclude: |
            **/.git*
            **/.git*/**
            **/.env
            **/.env.*
            **/node_modules/**
            **/tests/**
```

---

## 3. Flujo esperado

1. Haces push a `main`.
2. GitHub Actions corre el job `deploy`.
3. Los archivos del repo se suben por FTP a la carpeta del proyecto en BanaHosting.
4. Abres el subdominio y verificas que el PHP responda.

---

## 4. Variantes útiles (opcionales)

### Con PHP lint antes de subir

```yaml
      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: "8.2"

      - name: PHP syntax check
        run: find . -name "*.php" -not -path "./vendor/*" -print0 | xargs -0 -n1 php -l
```

### Si usas Composer (PHP con dependencias, pero no Laravel)

Agrega un step de `composer install --no-dev` **antes** del deploy y asegúrate de que `vendor/` se suba (o se genere en el servidor; en FTP puro suele subirse lo generado en el runner).

---

## Notas de prueba (rellenar después de experimentar)

> Usa esta sección cuando pruebes algo y falle, cambie, o falte un paso.

- **Fecha**:
- **Repo / subdominio**:
- **Qué funcionó**:
- **Qué no funcionó**:
- **Exclusiones que hiciste falta**:
- **Enlaces útiles**:
