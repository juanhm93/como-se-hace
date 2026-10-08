# CLAUDE.md

Guía para trabajar en este repositorio con Claude Code.

## Qué es este proyecto

`como-se-hace` es un repositorio **solo de documentación** en Markdown. No hay código de aplicación, dependencias, build ni tests. Cada guía documenta cómo realizar una acción de **configuración** (VPS, hosting, CI/CD, nube, flujos con IA, etc.) que ya se probó o se va a probar, para no olvidar los pasos.

Cada guía tiene dos partes y ambas son obligatorias:

- **No técnica**: qué se quiere lograr, por qué importa y el camino explicado de forma didáctica, entendible para alguien sin conocimientos técnicos.
- **Técnica**: pasos concretos, comandos, archivos de configuración, verificación y solución de problemas.

## Estructura

```text
README.md        Índice general: una tabla por categoría con todas las guías
PLANTILLA.md     Plantilla base para cualquier guía nueva
<categoria>/     Una carpeta por categoría, en minúsculas (vps/, banahosting/, aws/, cursor/)
  <accion>.md    Un archivo por acción concreta, en kebab-case (ej. deploy-sitio-estatico-cloudfront.md)
```

## Cómo crear o editar una guía

1. Parte siempre de `PLANTILLA.md` y respeta su orden de secciones:
   1. Título
   2. Enlaces **Antes / Después**
   3. **Objetivo**
   4. **El camino en pocas palabras** (con *Conceptos clave*)
   5. **Lo que vas a tener al final**
   6. **Requisitos**
   7. Pasos numerados
   8. **Checklist rápido**
   9. **Problemas frecuentes**
   10. **Notas de prueba**
2. Registra la guía en la tabla de su categoría del `README.md` (crea la sección de la categoría si no existe).
3. Enlaza las guías relacionadas con **Antes / Después** usando rutas relativas.

### Parte no técnica

- Explica el resultado visto desde afuera y el problema que resuelve, sin jerga.
- Cuenta las etapas como una historia corta; usa una **analogía** cuando ayude (ej. S3 = bodega central, CloudFront = sucursales).
- Define los términos en la tabla *Conceptos clave*, con una línea por término.
- Si el usuario explica algo con sus palabras, **mejora el lenguaje técnico**: corrige con tacto los conceptos imprecisos y explica el mecanismo real. Ejemplo: una invalidación de CloudFront no "despliega", purga la caché; el deploy ocurre al subir a S3.

### Parte técnica

- Cada paso empieza con **"Para qué:"**, una línea que lo conecta con la etapa del camino.
- Cada paso termina con **"Cómo verificar"**, que dice qué se debe ver si salió bien.
- Usa placeholders en MAYÚSCULAS para los datos propios (`MI_BUCKET`, `IP_DEL_VPS`, `TU_USUARIO`). Nunca pongas credenciales reales.
- **Si un paso se puede hacer por terminal y por un panel web** (consola de AWS, cPanel, GitHub, etc.), documenta las dos opciones:
  - `### Opción A: por terminal`, con los comandos.
  - `### Opción B: por la interfaz web (visual)`, que empieza con una línea **Ruta** de clics separada por `>` y debajo el detalle numerado:

    ```text
    **Ruta**: `CloudFront` > `Distributions` > `ID` > pestaña `Invalidations` > botón `Create invalidation`
    ```

  - Si el panel puede estar en otro idioma, anota la equivalencia de los nombres de los botones.
- Indica costos, límites o riesgos cuando existan (ej. borrados con `--delete`, cargos de la nube).
- Marca con claridad lo que **no está resuelto o probado** en lugar de presentarlo como funcional (ej. migraciones de Laravel en `banahosting/deploy-github-actions-laravel.md`).

## Estilo

- Escribe en **español**, con tono directo y cercano (tuteo).
- Usa títulos con `##` y separadores `---` entre secciones, tablas para comparaciones y datos, y bloques de código con lenguaje (`bash`, `yaml`, `nginx`, `env`, `text`).
- Las guías deben ser autocontenidas: quien las lea no debería necesitar otra fuente para completar la acción.

## Flujo de trabajo

- No hagas commit ni push salvo que se pida explícitamente.
- Al terminar, resume qué se creó o cambió e indica los **supuestos** que el usuario debe validar (ej. carpeta de build `dist/` vs `build/`, nombres de botones de la consola).
- Las secciones **Notas de prueba** las llena el usuario después de probar; no inventes su contenido.

## Pendientes conocidos

- Migrar las guías existentes (`vps/`, `banahosting/`, `cursor/`) al formato de `PLANTILLA.md`, añadiendo la parte no técnica y las opciones visuales con ruta de clics.
- `cursor/planificar-prds.md`:
  - Le falta el título `#` y la sección *Notas de prueba*.
  - No está registrada en el README.
  - Tiene restos de copiar y pegar (`None`) y los prompts no están en bloques de código.
- `banahosting/deploy-github-actions-laravel.md`:
  - Usa `actions/upload-artifact@v3`, que está deprecado.
  - El `health-cmd` usa `root_password` en lugar del secret.
  - Ejecuta `npm run build` sin instalar dependencias antes (falta `npm ci`).
- `banahosting/deploy-github-actions-php.md`: presenta `.ftp-deploy-sync-state.json` como mecanismo de exclusión, pero es el archivo de estado del action.
