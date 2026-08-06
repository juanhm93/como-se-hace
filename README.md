# como-se-hace

Repositorio de guías en Markdown: cómo se hacen procesos de configuración que ya se probaron (o se van a probar), para no olvidar los pasos.

## Cómo está organizado

- **Una carpeta = una categoría** (ej. `vps/`)
- **Un archivo `.md` = una cosa concreta a hacer**
- Al final de cada guía hay una sección **Notas de prueba** para anotar qué falló, qué faltó o qué se cambió al experimentarlo

## Categorías

### [vps/](./vps/)

| Guía | Descripción |
|------|-------------|
| [Opciones de VPS](./vps/opciones-vps.md) | Proveedores de pago y opciones free / de prueba |
| [Configurar proyecto Python](./vps/configurar-proyecto-python.md) | Subir y dejar corriendo una app Python en un VPS |

### [banahosting/](./banahosting/)

| Guía | Descripción |
|------|-------------|
| [Configurar proyecto por FTP](./banahosting/configurar-proyecto-ftp.md) | Subdominio, carpeta en cPanel, cuenta FTP y secretos |
| [Deploy Laravel (GitHub Actions)](./banahosting/github-actions-deploy-laravel.md) | Workflow `deploy.yml` para proyectos Laravel |
| [Deploy PHP puro (GitHub Actions)](./banahosting/github-actions-deploy-php.md) | Workflow `deploy.yml` para proyectos solo PHP |
| [Migraciones en servidor](./banahosting/github-actions-migraciones.md) | Intentos de migraciones automáticas (pendiente) |
