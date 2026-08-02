# Opciones de VPS (pago y prueba gratis)

Guía base para elegir un VPS. Precios y ofertas cambian; verifica siempre en la web del proveedor antes de comprar.

## Qué mirar antes de elegir

- **RAM**: mínimo 1 GB para un proyecto Python chico; 2 GB si usas Docker, bases de datos o varios servicios.
- **CPU**: 1 vCPU alcanza para APIs simples; 2+ si hay workers, scrapers o mucho tráfico.
- **Disco**: SSD preferible. 20–40 GB suele bastar al inicio.
- **Ancho de banda / transferencia**: revisa el límite mensual.
- **Ubicación**: elige el datacenter más cerca de tus usuarios (o de ti, si es solo lab).
- **Sistema operativo**: Ubuntu 22.04/24.04 LTS es lo más cómodo para empezar.
- **IPv4**: confirma que incluya IP pública (algunos planes baratos ya no la dan gratis).

---

## Proveedores de pago (buenos para empezar)

| Proveedor | Ideal para | Notas |
|-----------|------------|--------|
| **DigitalOcean** | Principiantes | Droplets claros, docs excelentes, panel simple. |
| **Linode / Akamai** | Uso general | Buen rendimiento/precio, facturación simple. |
| **Vultr** | Flexibilidad | Muchas ubicaciones; hay planes cloud y bare metal. |
| **Hetzner** | Mejor precio/rendimiento (Europa) | Muy barato y potente; UI menos “amigable” al inicio. |
| **Contabo** | Mucha RAM/disco barato | Recursos generosos; red/soporte más variables. |
| **OVHcloud** | Europa / control | Planes VPS clásicos; a veces hay stock limitado. |
| **AWS Lightsail / Azure / GCP** | Si ya usas la nube grande | Más caro a largo plazo, pero créditos de prueba útiles. |

### Rangos orientativos (pueden variar)

- **~3–6 USD/mes**: 1 vCPU, 1 GB RAM — lab / bot / API chica.
- **~6–12 USD/mes**: 1–2 vCPU, 2 GB RAM — app Python + DB liviana.
- **~12–25 USD/mes**: 2+ vCPU, 4 GB RAM — varios servicios o más tráfico.

---

## Opciones free / de prueba

Útiles para aprender sin pagar. **No las uses como producción seria** (límites, sleep, o caducan).

| Opción | Qué ofrece | Limitaciones típicas |
|--------|------------|----------------------|
| **Oracle Cloud Free Tier** | VMs Always Free (ARM Ampere, etc.) | Cupo por región; a veces hay que insistir al crear. |
| **Google Cloud Trial** | Crédito ~300 USD / ~90 días | Pide tarjeta; cobra si te pasas del crédito. |
| **AWS Free Tier** | Crédito / instancias free por tiempo limitado | No es VPS “para siempre”; vigila el billing. |
| **Azure Free** | Crédito inicial + servicios free | Igual: configura alertas de gasto. |
| **Railway / Render / Fly.io** | Deploy de apps sin administrar mucho el VPS | Free tier limitado (sleep, horas, recursos). No es VPS clásico. |
| **GitHub Codespaces / Gitpod** | Entorno de desarrollo en la nube | No es un VPS público 24/7 para tu app. |

### Tips para no llevarte sorpresas

1. Activa **alertas de facturación** el día 1.
2. Borra recursos de prueba cuando termines (discos, IPs, snapshots también cuestan).
3. En free tiers de cloud grande, un “olvido” puede generar cargo.
4. Si solo quieres practicar SSH + Python, un VPS barato (~4–6 USD) suele ser más simple que pelear con cuotas free.

---

## Recomendación práctica (punto de partida)

1. Si quieres **aprender rápido con panel simple**: DigitalOcean o Vultr (pago bajo).
2. Si quieres **máximo hardware por poco dinero**: Hetzner.
3. Si quieres **probar gratis primero**: Oracle Free Tier o crédito de GCP/AWS/Azure.
4. Sistema operativo sugerido: **Ubuntu 24.04 LTS**.

Cuando elijas proveedor, sigue con: [Configurar un proyecto Python en un VPS](./configurar-proyecto-python.md).

---

## Notas de prueba (rellenar después de experimentar)

> Usa esta sección cuando pruebes algo y falle, cambie, o falte un paso.

- **Fecha**:
- **Proveedor / plan**:
- **Qué funcionó**:
- **Qué no funcionó**:
- **Pasos extra que hiciste**:
- **Enlaces útiles**:
