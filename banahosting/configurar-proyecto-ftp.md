# Configurar un proyecto por FTP en Banahosting

Guía base para dejar un proyecto publicado en Banahosting usando **subdominio + carpeta en cPanel + cuenta FTP + deploy automático** (GitHub Actions).

**Relacionado**:
- Deploy Laravel → [github-actions-deploy-laravel.md](./github-actions-deploy-laravel.md)
- Deploy PHP puro → [github-actions-deploy-php.md](./github-actions-deploy-php.md)
- Migraciones remotas (pendiente) → [github-actions-migraciones.md](./github-actions-migraciones.md)

---

## 0. Lo que vas a tener al final

- Un **subdominio** apuntando al hosting (ej. `mi-proyecto.tudominio.com`)
- Una **carpeta del proyecto** dentro de `/developer` en el File Manager de cPanel
- Una **cuenta FTP** que apunta exactamente a esa carpeta
- Los **datos de conexión FTP** anotados (servidor, usuario, contraseña, directorio)
- (Opcional) **GitHub Actions** subiendo el código al servidor en cada push a `main`

---

## 1. Crear subdominio en la cuenta Banahosting

> **PASOS EN HOSTING** — desde cPanel de Banahosting.

1. Entra a **cPanel** de tu cuenta Banahosting.
2. Busca la sección **Dominios** (o **Subdominios** / **Subdomains**).
3. Crea un subdominio nuevo:
   - **Subdominio**: el nombre del proyecto (ej. `mi-proyecto`)
   - **Dominio**: tu dominio principal
   - **Document Root**: anota la ruta que cPanel asigne (suele ser algo como `public_html/mi-proyecto` o similar)
4. Guarda y espera a que el DNS propague si acabas de crear el registro.

**Nota**: En este flujo la carpeta de trabajo del proyecto va en `/developer`, no necesariamente en el document root por defecto del subdominio. Si el sitio no carga después del deploy, revisa que el subdominio o un enlace simbólico apunte a la carpeta correcta dentro de `/developer`.

---

## 2. Crear carpeta del proyecto en File Manager

1. En cPanel, abre **File Manager** (Administrador de archivos).
2. Navega a la carpeta principal de tu cuenta y entra en **`developer`**.
   - Ruta típica: `/developer` (dentro del home del usuario de cPanel)
3. Crea una carpeta con el **nombre del proyecto** (ej. `mi-proyecto`).
4. Dentro de esa carpeta es donde el FTP va a subir los archivos.

Estructura esperada:

```
/developer/
  └── mi-proyecto/     ← carpeta del proyecto (raíz del deploy FTP)
```

---

## 3. Crear cuenta FTP

1. En cPanel, ve a **Cuentas FTP** (FTP Accounts).
2. Crea una cuenta nueva con:
   - **Usuario**: suele ser algo como `usuario@tudominio.com` o el formato que muestre Banahosting
   - **Contraseña**: genera una segura y guárdala
   - **Directorio / Directorio raíz**: debe apuntar a la carpeta del proyecto creada en el paso anterior
     - Ejemplo: `/developer/mi-proyecto`
   - **Cuota**: según necesites (o ilimitada si el plan lo permite)
3. Guarda la cuenta.

### Verificar datos en "Configurar cliente de FTP"

En la misma pantalla de cuentas FTP, Banahosting suele mostrar un bloque **Configurar cliente de FTP** con:

| Campo | Ejemplo | Dónde se usa |
|-------|---------|--------------|
| **Servidor / Host** | `ftp.tudominio.com` o IP | `FTP_SERVER` en GitHub Secrets |
| **Usuario** | `usuario@tudominio.com` | `FTP_USERNAME` |
| **Contraseña** | la que definiste | `FTP_PASSWORD` |
| **Puerto** | `21` (FTP) o `22` (SFTP, si está disponible) | según el action que uses |
| **Directorio** | `/developer/mi-proyecto` | `server-dir` en el workflow |

> Es importante revisar **Configurar cliente de FTP**: ahí aparece la data real de tu cuenta, no asumas valores genéricos.

### Probar conexión manual (recomendado)

Antes de configurar GitHub Actions, prueba con FileZilla o el cliente FTP de tu preferencia:

- Host: el servidor FTP
- Usuario y contraseña de la cuenta creada
- Puerto: 21 (o el que indique cPanel)
- Directorio remoto: la carpeta del proyecto

Si puedes subir un `index.php` o `index.html` de prueba y verlo en el navegador, el hosting está listo para el deploy automático.

---

## 4. Variables de entorno y secretos

### En el proyecto (referencia local / `.env`)

Si tu aplicación o algún script necesita leer credenciales FTP:

```env
FTP_API_FIELD_USERNAME=
FTP_API_FIELD_PASSWORD=
FTP_API_FIELD_SERVER=
```

### En GitHub (para GitHub Actions)

En el repositorio: **Settings → Secrets and variables → Actions → New repository secret**

| Secret | Valor |
|--------|--------|
| `FTP_SERVER` | Host del paso "Configurar cliente de FTP" |
| `FTP_USERNAME` | Usuario FTP |
| `FTP_PASSWORD` | Contraseña FTP |

Para proyectos Laravel con tests en CI, además suelen hacer falta secretos de base de datos (`DB_PASSWORD`, `DB_DATABASE`, `DB_USER`, etc.) — ver [github-actions-deploy-laravel.md](./github-actions-deploy-laravel.md).

---

## 5. Deploy automático con GitHub Actions

Elige la guía según el tipo de proyecto:

| Tipo | Archivo |
|------|---------|
| Laravel (PHP + Composer + npm build) | [github-actions-deploy-laravel.md](./github-actions-deploy-laravel.md) |
| PHP puro (sin framework) | [github-actions-deploy-php.md](./github-actions-deploy-php.md) |

Coloca el workflow en tu repo en:

```
.github/workflows/deploy.yml
```

---

## 6. Checklist rápido

- [ ] Subdominio creado en cPanel
- [ ] Carpeta `/developer/<nombre-proyecto>` creada
- [ ] Cuenta FTP apuntando a esa carpeta
- [ ] Conexión FTP probada manualmente
- [ ] Secretos `FTP_SERVER`, `FTP_USERNAME`, `FTP_PASSWORD` en GitHub
- [ ] Workflow `deploy.yml` en el repo
- [ ] Push a `main` y revisar pestaña **Actions** en GitHub

---

## Problemas frecuentes

| Síntoma | Qué revisar |
|---------|-------------|
| FTP conecta pero no sube archivos | Directorio raíz de la cuenta FTP; permisos de la carpeta en cPanel |
| Sitio en blanco o 403 | ¿Hay `index.php` / `index.html` en la raíz del deploy? ¿El subdominio apunta al directorio correcto? |
| Deploy falla en GitHub Actions | Secretos mal escritos; servidor FTP incorrecto; firewall del hosting bloqueando la IP de GitHub |
| Laravel: assets rotos | ¿Corrió `npm run build`? ¿Se subió la carpeta `public/` completa? |
| Cambios no se ven | Caché del navegador; caché de LiteSpeed/Cloudflare si está activo |

---

## Notas de prueba (rellenar después de experimentar)

> Cuando algo no coincida con esta guía, anótalo aquí para la próxima vez.

- **Fecha**:
- **Dominio / subdominio**:
- **Ruta real de la carpeta en cPanel**:
- **Usuario FTP**:
- **Qué funcionó**:
- **Qué no funcionó**:
- **Pasos extra**:
- **Ajustes al document root del subdominio**:
