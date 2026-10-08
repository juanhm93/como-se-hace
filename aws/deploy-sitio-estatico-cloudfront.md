# Deploy de un sitio estático en AWS (S3 + CloudFront)

**Antes**: tener creados el bucket de S3 con el sitio y la distribución de CloudFront que lo sirve (esta guía no cubre su creación).
**Después**: Ninguna (posible siguiente paso: automatizar este flujo con GitHub Actions).

---

## Objetivo

**Qué se quiere lograr**: que los cambios hechos en el proyecto (textos, estilos, funcionalidades) queden publicados en el sitio web y que todos los visitantes vean la versión nueva, no una anterior.

**Por qué importa**: el sitio no se publica directamente desde el código que escribimos. Primero hay que "empaquetarlo", luego guardarlo donde vive el sitio y, por último, avisarle al sistema que lo distribuye que hay una versión nueva. Si se salta un paso, el sitio no cambia o muestra una mezcla de versión vieja y nueva.

**Para quién / cuándo usar esta guía**: cada vez que haya cambios en un sitio estático (por ejemplo React, Vue o Vite) alojado en S3 y servido con CloudFront, y se quiera publicar a mano desde la computadora local.

---

## El camino en pocas palabras

1. **Compilar (build)**: el código del proyecto se transforma en archivos listos para el navegador: un `index.html` y una carpeta `assets/` con el JavaScript, el CSS y las imágenes optimizadas. Es como pasar un borrador a la versión final impresa.
2. **Subir a S3**: esos archivos se copian al bucket de S3, que es el "almacén" donde vive el sitio. En este momento la versión nueva ya está guardada en AWS, pero los visitantes todavía pueden seguir viendo la anterior.
3. **Invalidar en CloudFront**: CloudFront guarda copias del sitio en servidores repartidos por el mundo para que cargue rápido. La invalidación le dice: "las copias que tienes ya no sirven, tíralas y ve a buscar las nuevas al almacén". Desde ese momento todos reciben la versión nueva.

**Analogía**: S3 es la bodega central de una cadena de tiendas y CloudFront son las sucursales, cada una con su propio stock en estantería para atender rápido. Si llega un producto nuevo a la bodega, las sucursales siguen vendiendo lo que tienen en la estantería. La invalidación es la orden de retirar ese stock viejo para que cada sucursal pida el nuevo a la bodega.

### Conceptos clave

| Concepto | Qué es (en simple) |
|----------|--------------------|
| Sitio estático | Sitio compuesto solo por archivos (HTML, CSS, JS, imágenes); no necesita un servidor ejecutando código. |
| Build / compilar | Proceso que convierte el código del proyecto en archivos optimizados para el navegador. |
| S3 | Servicio de AWS para guardar archivos. Un **bucket** es una "carpeta raíz" dentro de S3. |
| CloudFront | Red de distribución de contenido (CDN) de AWS: entrega el sitio desde el servidor más cercano al visitante. |
| Origen (origin) | El lugar del que CloudFront toma los archivos originales; en este caso, el bucket de S3. |
| Caché | Copia temporal de un archivo que CloudFront guarda para no pedírselo al origen cada vez. |
| TTL | Tiempo que una copia en caché se considera válida antes de que CloudFront vuelva a consultar al origen. |
| Invalidación | Orden manual para que CloudFront descarte las copias en caché de ciertas rutas antes de que venza su TTL. |

### Por qué hace falta la invalidación (explicación técnica)

CloudFront mantiene copias en caché de los archivos en sus **edge locations** (servidores distribuidos geográficamente). Cada copia tiene un **TTL**; con la configuración por defecto suele ser de 24 horas. Mientras el TTL no vence, CloudFront responde con la copia que tiene y **no consulta a S3**, aunque en S3 ya exista un archivo nuevo con el mismo nombre.

Por eso, técnicamente, el despliegue ocurre al **subir los archivos a S3**. La invalidación no "despliega": **purga la caché** de CloudFront para las rutas indicadas. La siguiente petición a esas rutas es un *cache miss*: CloudFront va al origen (S3), obtiene la versión nueva, la sirve y la vuelve a guardar en caché.

Un detalle importante del build (Vite y similares): los archivos de `assets/` llevan un **hash** en el nombre (ej. `index-4f9a2c1b.js`), que cambia en cada build cuando cambia el contenido. Esos archivos nunca chocan con sus versiones viejas en caché porque tienen otro nombre. El que **siempre se llama igual es `index.html`**, y es justamente el que indica qué assets cargar. Si CloudFront sigue sirviendo el `index.html` viejo, el navegador sigue pidiendo los assets viejos, y por eso la invalidación es imprescindible.

---

## Lo que vas a tener al final

- Los archivos nuevos compilados en el bucket de S3
- La caché de CloudFront invalidada
- El sitio público mostrando la versión nueva

---

## Requisitos

- Node.js y npm instalados, con las dependencias del proyecto (`npm install`)
- Acceso a la cuenta de AWS, por consola web o con **AWS CLI v2** configurada (`aws configure` o un perfil)
- Datos a mano:
  - **Nombre del bucket** de S3 (ej. `mi-sitio-prod`)
  - **ID de la distribución** de CloudFront (ej. `E1ABCDEF2GHIJK`). Dónde verlo: `CloudFront` > `Distributions` > columna `ID`
- Permisos IAM mínimos para el usuario que despliega:
  - `s3:ListBucket`, `s3:PutObject`, `s3:DeleteObject` sobre el bucket
  - `cloudfront:CreateInvalidation`, `cloudfront:GetInvalidation` sobre la distribución

---

## 1. Compilar el proyecto

Para qué: generar la versión final del sitio que se va a publicar.

```bash
npm run build
```

La salida queda en una carpeta del proyecto: en Vite es `dist/` y en Create React App es `build/`. En esta guía se usa `dist/`.

**Cómo verificar**: la carpeta contiene `index.html` y `assets/`.

```bash
ls dist
# index.html  assets/
```

---

## 2. Subir los archivos a S3

Para qué: guardar la versión nueva en el origen del que CloudFront toma los archivos.

### Opción A: por terminal (AWS CLI, recomendado)

```bash
aws s3 sync dist/ s3://MI_BUCKET --delete
```

- `sync` sube solo los archivos nuevos o modificados.
- `--delete` borra del bucket los archivos que ya no existen en `dist/` (por ejemplo, los assets con hashes viejos). Así el bucket no acumula basura de builds anteriores.
- Ojo con la barra final en `dist/`: se suben los **contenidos** de la carpeta, no la carpeta en sí.

> Si se prefiere no borrar nada (más seguro mientras se aprende), quita `--delete`. Los assets viejos se quedarán en el bucket sin afectar al sitio.

### Opción B: por la consola web de AWS (visual)

**Ruta**: `S3` > `Buckets` > `MI_BUCKET` > pestaña `Objects` > botón `Upload` > `Add files` / `Add folder` > botón `Upload`

1. En el buscador superior de la consola, escribe **S3** y entra al servicio.
2. Menú izquierdo: **Buckets**, y haz clic en el nombre del bucket del sitio.
3. Pestaña **Objects**: botón **Upload**.
4. Arrastra el **contenido** de `dist/` (el `index.html` y la carpeta `assets/`), o usa **Add files** y **Add folder**. No subas la carpeta `dist` completa.
5. Al final de la página, botón **Upload**. Espera el mensaje *Upload succeeded* y pulsa **Close**.

> Si la consola está en español, los nombres cambian: `Upload` = **Cargar**, `Objects` = **Objetos**.

**Cómo verificar**: `index.html` y `assets/` quedan en la raíz del bucket.

```bash
aws s3 ls s3://MI_BUCKET/
#                            PRE assets/
# 2026-10-08 10:00:00       512 index.html
```

---

## 3. Crear la invalidación en CloudFront

Para qué: que CloudFront descarte las copias en caché y empiece a servir los archivos nuevos de S3.

### Opción A: por terminal (AWS CLI)

```bash
aws cloudfront create-invalidation \
  --distribution-id MI_DISTRIBUTION_ID \
  --paths "/*"
```

- `"/*"` invalida todas las rutas del sitio. Va entre comillas para que la terminal no interprete el `*`.
- La respuesta incluye un `Id` de invalidación y `"Status": "InProgress"`.

Para seguir el estado:

```bash
aws cloudfront get-invalidation \
  --distribution-id MI_DISTRIBUTION_ID \
  --id ID_DE_LA_INVALIDACION
```

Cuando el estado pasa a `Completed`, ya está propagada (suele tardar de segundos a pocos minutos).

### Opción B: por la consola web de AWS (visual)

**Ruta**: `CloudFront` > `Distributions` > `MI_DISTRIBUTION_ID` > pestaña `Invalidations` > botón `Create invalidation` > escribir `/*` > botón `Create invalidation`

1. En el buscador superior de la consola, escribe **CloudFront** y entra al servicio.
2. Menú izquierdo: **Distributions**, y haz clic en el **ID** de la distribución del sitio.
3. Pestaña **Invalidations**: botón **Create invalidation**.
4. En el campo **Add object paths** escribe `/*`.
5. Botón **Create invalidation**.
6. Vuelve a la pestaña **Invalidations** y espera a que el estado pase de *In progress* a *Completed*.

> Si la consola está en español: `Invalidations` = **Invalidaciones**, `Create invalidation` = **Crear invalidación**.

> **Costo**: las primeras 1.000 rutas invalidadas al mes son gratis. `/*` cuenta como **una** sola ruta, así que para deploys manuales normales no tiene costo.

**Cómo verificar**: abrir el sitio en una ventana de incógnito (para evitar la caché del navegador) y comprobar que se ven los cambios. También se puede revisar desde la terminal:

```bash
curl -I https://MI_DOMINIO
# x-cache: Miss from cloudfront  -> CloudFront fue a buscar el archivo a S3 (versión nueva)
# x-cache: Hit from cloudfront   -> respondió desde su caché
```

---

## Resumen de comandos

```bash
npm run build
aws s3 sync dist/ s3://MI_BUCKET --delete
aws cloudfront create-invalidation --distribution-id MI_DISTRIBUTION_ID --paths "/*"
```

---

## Checklist rápido

- [ ] `npm run build` sin errores
- [ ] `dist/` contiene `index.html` y `assets/`
- [ ] Archivos subidos a la **raíz** del bucket
- [ ] Invalidación `/*` creada
- [ ] Invalidación en estado `Completed`
- [ ] Sitio verificado en incógnito

---

## Problemas frecuentes

| Síntoma | Qué revisar |
|---------|-------------|
| El sitio sigue mostrando la versión vieja | ¿Se creó la invalidación? ¿Ya está `Completed`? Probar en incógnito o con recarga forzada (`Cmd/Ctrl + Shift + R`): puede ser la caché del navegador, no la de CloudFront. |
| Página en blanco y errores 404 en `/assets/...` | Se subió la carpeta `dist` completa y quedó `s3://MI_BUCKET/dist/index.html`. Los archivos deben estar en la raíz del bucket. |
| `AccessDenied` al subir o invalidar | Permisos IAM del usuario o perfil de la CLI (ver Requisitos). Comprobar el perfil activo con `aws sts get-caller-identity`. |
| Error 403/404 al recargar una ruta interna (ej. `/contacto`) | Es configuración de la distribución, no del deploy: en una SPA hay que definir *Custom error responses* que devuelvan `/index.html` con código 200. |
| `aws: command not found` | Instalar AWS CLI v2 y ejecutar `aws configure`. |

---

## Notas de prueba (rellenar después de experimentar)

> Usa esta sección cuando pruebes algo y falle, cambie, o falte un paso.

- **Fecha**:
- **Entorno** (framework / carpeta de build / región del bucket):
- **Qué funcionó**:
- **Qué no funcionó**:
- **Pasos extra que hiciste**:
- **Tiempo que tardó la invalidación**:
- **Enlaces útiles**:
