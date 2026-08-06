# GitHub Actions — Migraciones en servidor (pendiente)

> **Estado**: por ahora **no funcionó** / no se logró automatizar migraciones contra el servidor Banahosting desde GitHub Actions.

Esta guía es un lugar para documentar intentos y la solución cuando se consiga.

**Contexto**: en proyectos Laravel (u otros con migraciones), el deploy por FTP solo sube archivos; **no ejecuta** `php artisan migrate` en el hosting. Eso requiere otro mecanismo (SSH, cron, panel, script remoto, etc.).

---

## Por qué es difícil en hosting compartido

| Enfoque | Limitación en Banahosting / cPanel |
|---------|-------------------------------------|
| SSH + `artisan migrate` | Muchos planes compartidos no dan SSH o lo limitan |
| Ejecutar PHP remoto vía HTTP | Riesgo de seguridad si se expone un endpoint de migración |
| FTP-Deploy-Action | Solo transfiere archivos |
| GitHub Actions + MySQL del hosting | El runner de GitHub no alcanza la DB del hosting sin IP permitida / túnel |

---

## Ideas a probar (sin validar aún)

1. **Cron en cPanel** que ejecute migraciones en horario fijo o tras detectar cambios (complejo).
2. **SSH** si el plan lo incluye:
   ```bash
   cd /developer/mi-proyecto
   php artisan migrate --force
   ```
3. **Script de deploy manual** post-FTP desde tu máquina con acceso SSH.
4. **Whitelist de IP** de GitHub Actions en MySQL remoto (frágil: las IPs de GitHub cambian).
5. **Panel Banahosting / phpMyAdmin**: migraciones manuales o import SQL (no automatizado).

---

## Borrador de workflow (no probado — no usar en producción tal cual)

```yaml
# NO FUNCIONÓ — guardar aquí lo que se intentó
# jobs:
#   migrate:
#     needs: deploy
#     runs-on: ubuntu-latest
#     steps:
#       - name: Run migrations remotely
#         run: |
#           curl -X POST "https://mi-proyecto.tudominio.com/deploy/migrate?token=${{ secrets.MIGRATE_TOKEN }}"
```

Si algún día se usa un endpoint HTTP para migrar, debe estar **muy protegido** (token, IP, desactivar tras uso).

---

## Notas de prueba (rellenar cuando se reintente)

- **Fecha**:
- **Qué se intentó**:
- **Error exacto**:
- **¿Hay SSH en el plan?**:
- **¿Dónde corre MySQL?** (mismo servidor / remoto):
- **Próximo enfoque a probar**:
