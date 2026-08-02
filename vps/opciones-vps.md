# Opciones de VPS (compra y pruebas gratis)

Guía base para elegir un VPS. Precios y créditos cambian; verifica siempre en la web del proveedor antes de comprar.

---

## Qué mirar al elegir

| Criterio | Por qué importa |
| --- | --- |
| RAM / CPU | Un proyecto Python chico suele ir bien con 1–2 GB; APIs + DB necesitan más |
| Disco (SSD/NVMe) | Logs, venv, Docker e imágenes ocupan rápido |
| Transferencia / mes | Tráfico saliente; free tiers suelen tener límite |
| Región | Latencia hacia tus usuarios (EU, US, LatAm…) |
| Facilidad | Panel, docs, SSH listo, backups |
| Facturación | Por hora vs mensual; fácil de apagar = menos sorpresas |

**Recomendación mínima para aprender / un proyecto Python simple:** 1 vCPU, 1–2 GB RAM, Ubuntu 22.04 o 24.04 LTS.

---

## Opciones de pago (buena relación calidad/precio)

### Hetzner Cloud
- **Ideal para:** mejor precio/rendimiento en Europa
- **Entrada aproximada:** ~€3.79/mes (ej. CX22: 2 vCPU, 4 GB)
- **Pros:** barato, rápido, panel claro
- **Contras:** menos presencia en LatAm/US; verificación de cuenta a veces estricta
- **Web:** https://www.hetzner.com/cloud

### Contabo
- **Ideal para:** mucha RAM/disco por poco dinero
- **Entrada aproximada:** ~€5–6/mes en planes básicos
- **Pros:** specs altas al precio
- **Contras:** rendimiento/IO más variable; soporte más lento
- **Web:** https://contabo.com

### DigitalOcean
- **Ideal para:** principiantes y docs excelentes
- **Entrada aproximada:** desde ~$6/mes (1 vCPU, 1 GB)
- **Pros:** Droplets fáciles, tutoriales, snapshots
- **Contras:** más caro por recurso que Hetzner/Contabo
- **Web:** https://www.digitalocean.com

### Vultr / Linode (Akamai)
- **Ideal para:** muchas regiones y facturación por hora
- **Entrada aproximada:** desde ~$5–6/mes
- **Pros:** red global, fácil de crear/destruir
- **Contras:** precio medio-alto vs Hetzner
- **Web:** https://www.vultr.com · https://www.linode.com

### Hostinger / proveedores “all-in-one”
- **Ideal para:** panel amigable y menos terminal al inicio
- **Contras:** menos control fino; a veces upsells
- Útil si quieres algo simple; para aprender despliegue “real”, mejor un cloud clásico (DO/Hetzner/Vultr).

---

## Gratis / prueba (para experimentar)

### Oracle Cloud — Always Free
- **Tipo:** gratis permanente (Always Free), no solo trial
- **Aprox.:** hasta ~4 OCPU ARM + 24 GB RAM + ~200 GB storage (sujeto a cuotas actuales)
- **Pros:** generoso de verdad para labs y proyectos personales
- **Contras:** a veces no hay capacidad (`Out of host capacity`); setup de red/SSH más fricción; instancias ARM (no x86)
- **Web:** https://www.oracle.com/cloud/free/
- **Tip:** si falla crear la instancia, prueba otra región o vuelve a intentar más tarde

### DigitalOcean — crédito de bienvenida
- **Tipo:** crédito temporal para usuarios nuevos (suele ser ~$200 / 60 días; confirma en su sitio)
- **Pros:** panel muy cómodo para el primer VPS
- **Contras:** al acabar el crédito cobras; no es forever-free
- **Tip:** apaga/destruye el Droplet cuando no lo uses

### Vultr — créditos / promos nuevos usuarios
- **Tipo:** crédito promocional (varía)
- **Pros:** muchas ubicaciones, cobro por hora
- **Contras:** igual: al acabar el crédito, paga

### AWS / Google Cloud — free tier
- **Tipo:** Always Free limitado + créditos de prueba (12 meses en algunos productos AWS)
- **Pros:** aprender cloud “grande”
- **Contras:** facturación fácil de disparar si dejas recursos; más complejo que un VPS simple
- **Tip:** pon alertas de billing el día 1

### Google Cloud — e2-micro Always Free
- **Tipo:** instancia pequeña forever-free en regiones concretas
- **Pros:** gratis de verdad para pruebas mínimas
- **Contras:** muy poca RAM/CPU; no sirve para cargas serias

---

## Cómo elegir rápido

| Tu caso | Opción sugerida |
| --- | --- |
| Quiero probar sin gastar (o casi) | Oracle Always Free **o** crédito DO/Vultr |
| Quiero barato y serio a largo plazo | **Hetzner** |
| Quiero mucha RAM barata | **Contabo** |
| Es mi primer VPS y quiero docs claras | **DigitalOcean** |
| Proyecto Python chico + Nginx | 1–2 GB RAM alcanza para empezar |

---

## Checklist al crear el VPS

1. SO: **Ubuntu 24.04 LTS** (o 22.04)
2. Guardar IP, usuario (`root` o el que den) y clave SSH / password
3. Abrir solo puertos necesarios (22 SSH; luego 80/443 si hay web)
4. Crear usuario no-root y deshabilitar login root por password (cuando te sientas cómodo)
5. Anotar región y plan (para comparar costo/rendimiento después)

Siguiente guía: [configurar-proyecto-python.md](./configurar-proyecto-python.md)

---

## Notas de prueba

> Espacio para lo que falló, pasos extra o cambios de precio/créditos.

- Fecha:
- Proveedor probado:
- Qué funcionó:
- Qué no funcionó / paso extra:
- Costo real al final del mes:
