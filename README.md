# como-se-hace

Repositorio de guías en Markdown: cómo se hacen procesos de configuración que ya se probaron (o se van a probar), para no olvidar los pasos.

## Cómo está organizado

- **Una carpeta = una categoría** (ej. `vps/`)
- **Un archivo `.md` = una cosa concreta a hacer**
- Cada guía tiene una parte **no técnica** (objetivo y camino explicado de forma didáctica) y una parte **técnica** (pasos, comandos y configuración)
- Al final de cada guía hay una sección **Notas de prueba** para anotar qué falló, qué faltó o qué se cambió al experimentarlo
- Para una guía nueva, parte de la [plantilla](./PLANTILLA.md)

## Categorías

### [vps/](./vps/)

| Guía | Descripción |
|------|-------------|
| [Opciones de VPS](./vps/opciones-vps.md) | Proveedores de pago y opciones free / de prueba |
| [Configurar proyecto Python](./vps/configurar-proyecto-python.md) | Subir y dejar corriendo una app Python en un VPS |

### [banahosting/](./banahosting/)

| Guía | Descripción |
|------|-------------|
| [Configurar proyecto por FTP](./banahosting/configurar-proyecto-ftp.md) | Subdominio, carpeta en `/developer`, cuenta FTP y secrets |
| [Deploy GitHub Actions (PHP)](./banahosting/deploy-github-actions-php.md) | Workflow de deploy FTP para proyecto solo PHP |
| [Deploy GitHub Actions (Laravel)](./banahosting/deploy-github-actions-laravel.md) | Workflow Laravel (referencia; migraciones aún pendientes) |

### [aws/](./aws/)

| Guía | Descripción |
|------|-------------|
| [Deploy sitio estático (S3 + CloudFront)](./aws/deploy-sitio-estatico-cloudfront.md) | Build local, subida a S3 e invalidación de CloudFront |

### [claude-config/](./claude-config/)

| Guía | Descripción |
|------|-------------|
| [Prompts de configuración inicial](./claude-config/prompts-configuracion-inicial.md) | Analizar la arquitectura, crear el `CLAUDE.md`, levantar el backend y analizar el impacto de un feature |
