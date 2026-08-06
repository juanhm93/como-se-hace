# Configurar un proyecto en BanaHosting por FTP

Guía paso a paso para dejar un proyecto (PHP u otro) listo en BanaHosting: subdominio, carpeta en el servidor, cuenta FTP y datos para deploy (GitHub Actions u otro cliente).

**Después de esto**:

- Proyecto solo PHP → [Deploy con GitHub Actions (PHP)](./deploy-github-actions-php.md)
- Proyecto Laravel → [Deploy con GitHub Actions (Laravel)](./deploy-github-actions-laravel.md)

---

## 0. Lo que vas a tener al final

- Un **subdominio** apuntando a la carpeta del proyecto
- Una **carpeta** del proyecto dentro de `/developer` en el cPanel
- Una **cuenta FTP** con acceso solo a esa carpeta
- Los datos (usuario, clave, servidor) listos como secrets / variables para el deploy

---

## 1. Crear el subdominio (pasos en el hosting)

En la cuenta de BanaHosting / cPanel:

1. Entra al **cPanel**.
2. Busca la sección de **Subdominios** (o Domains → Subdomains, según la versión del panel).
3. Crea el subdominio, por ejemplo: `mi-proyecto.tudominio.com`.
4. Anota el nombre del subdominio: lo vas a usar como dominio de la cuenta FTP y como URL del sitio.

> Tip: deja claro a qué carpeta del servidor queda asociado el subdominio. En el siguiente paso la carpeta del proyecto vive bajo `/developer`.

---

## 2. Crear la carpeta del proyecto en `/developer`

En el **Administrador de archivos** del cPanel:

1. Abre la carpeta principal de la cuenta (home del cPanel).
2. Entra a la carpeta **`developer`** (si no existe, créala).
3. Dentro de `developer`, crea una carpeta con el **nombre del proyecto**, por ejemplo:

```text
/home/TU_USUARIO/developer/mi-proyecto
```

4. Esa carpeta es el destino del FTP y, en la práctica, donde vive el código que se despliega.

> Ajusta el path exacto si tu cPanel muestra otra ruta base; lo importante es: **carpeta principal → `developer` → nombre del proyecto**.

---

## 3. Crear la cuenta FTP

En cPanel → **Cuentas FTP** (FTP Accounts):

1. Crea una cuenta nueva.
2. Completa:
   - **Nombre / usuario**: un identificador claro (puede ir ligado al proyecto).
   - **Dominio**: el **subdominio** que creaste en el paso 1.
   - **Contraseña**: una clave segura (guárdala; la vas a poner en secrets).
   - **Directorio / Directory**: la carpeta del proyecto bajo `developer`, por ejemplo:

```text
developer/mi-proyecto
```

3. Guarda / crea la cuenta.
4. Abre **Configurar cliente de FTP** (o “Configure FTP Client”) para ver la data exacta que te da el hosting:

| Dato | Dónde se usa |
|------|----------------|
| Servidor / Host FTP | `FTP_SERVER` / `FTP_API_FIELD_SERVER` |
| Usuario | `FTP_USERNAME` / `FTP_API_FIELD_USERNAME` |
| Contraseña | `FTP_PASSWORD` / `FTP_API_FIELD_PASSWORD` |
| Puerto | suele ser `21` (FTP) o el que indique el panel |

> Es importante revisar esa pantalla: el usuario a veces viene con el dominio completo (`usuario@subdominio...`) y el host puede ser el dominio o un hostname del servidor.

---

## 4. Variables / secrets para el deploy

Usa estos nombres al configurar GitHub Secrets (o variables de entorno locales). Los valores salen de **Configurar cliente de FTP**.

```env
FTP_API_FIELD_USERNAME=
FTP_API_FIELD_PASSWORD=
FTP_API_FIELD_SERVER=
```

En GitHub Actions suele mapearse así (nombres usados en los workflows de este repo):

| Secret en GitHub | Valor |
|------------------|--------|
| `FTP_USERNAME` | usuario FTP |
| `FTP_PASSWORD` | contraseña FTP |
| `FTP_SERVER` | host / servidor FTP |

Cómo cargarlos en el repo:

1. Repo en GitHub → **Settings** → **Secrets and variables** → **Actions**.
2. **New repository secret** por cada uno.
3. Pega los valores sin espacios de más.

---

## 5. Checklist rápido

- [ ] Subdominio creado
- [ ] Carpeta `developer/nombre-del-proyecto` creada
- [ ] Cuenta FTP apuntando a esa carpeta
- [ ] Datos tomados de “Configurar cliente de FTP”
- [ ] Secrets cargados en GitHub
- [ ] Workflow de deploy agregado al repo (PHP o Laravel)

---

## Notas de prueba (rellenar después de experimentar)

> Usa esta sección cuando pruebes algo y falle, cambie, o falte un paso.

- **Fecha**:
- **Cuenta / dominio**:
- **Qué funcionó**:
- **Qué no funcionó**:
- **Pasos extra que hiciste**:
- **Path real de la carpeta en cPanel**:
- **Formato exacto del usuario FTP** (con o sin `@dominio`):
- **Enlaces útiles**:
