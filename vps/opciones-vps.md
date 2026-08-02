# Opciones de VPS para comprar y pruebas gratuitas

> **Notas personales:** Usa esta sección para anotar qué probaste, qué te funcionó y qué no.
>
> - Fecha de prueba:
> - Proveedor elegido:
> - Observaciones:

---

## ¿Qué es un VPS?

Un **VPS (Virtual Private Server)** es un servidor virtual con recursos dedicados (CPU, RAM, disco) donde tú tienes control total: instalas lo que quieras, configuras el sistema y despliegas tus proyectos.

**Ideal para:** APIs en Python, bots, sitios web, bases de datos, pruebas de despliegue, aprender Linux en un entorno real.

---

## Proveedores recomendados (de pago)

### Económicos / para empezar

| Proveedor | Precio aprox. | RAM | Disco | Notas |
|-----------|---------------|-----|-------|-------|
| [Hetzner](https://www.hetzner.com/cloud) | ~4–5 €/mes | 2–4 GB | 20–40 GB SSD | Muy buena relación precio/rendimiento. Datacenters en Europa. |
| [DigitalOcean](https://www.digitalocean.com) | ~6 $/mes | 1 GB | 25 GB SSD | Interfaz simple, mucha documentación. Droplets fáciles de crear. |
| [Vultr](https://www.vultr.com) | ~6 $/mes | 1 GB | 25 GB SSD | Similar a DigitalOcean. Muchas ubicaciones. |
| [Linode (Akamai)](https://www.linode.com) | ~5 $/mes | 1 GB | 25 GB SSD | Estable, buena documentación. |
| [Contabo](https://contabo.com) | ~5 €/mes | 4–8 GB | 50+ GB | Muy barato por recursos, a veces soporte más lento. |

### Otros populares

| Proveedor | Precio aprox. | Notas |
|-----------|---------------|-------|
| [OVH](https://www.ovhcloud.com) | ~3–6 €/mes | Europeo, buenas opciones VPS. |
| [Scaleway](https://www.scaleway.com) | ~3–8 €/mes | Francés, precios competitivos. |
| [AWS Lightsail](https://aws.amazon.com/lightsail/) | ~5 $/mes | Entrada sencilla a AWS. |
| [Google Cloud Compute](https://cloud.google.com/compute) | Variable | Más complejo; suele usarse con crédito gratuito. |

---

## Opciones gratuitas o de prueba

### Créditos para nuevos usuarios

| Proveedor | Oferta | Duración / condiciones |
|-----------|--------|------------------------|
| **DigitalOcean** | 200 $ de crédito | Con referido o promociones (suele durar 60 días). Requiere tarjeta. |
| **Google Cloud** | 300 $ de crédito | 90 días para nuevos usuarios. Requiere tarjeta. |
| **AWS** | Capa gratuita + créditos | EC2 t2.micro gratis 12 meses (limitado). Lightsail a veces tiene promos. |
| **Oracle Cloud** | Instancias Always Free | **Gratis permanente** (con límites): 2 VMs ARM o AMD pequeñas. Requiere tarjeta para verificación. |
| **Azure** | 200 $ de crédito | 30 días para nuevos usuarios. |

### Siempre gratis (con limitaciones)

| Servicio | Qué ofrece | Limitaciones |
|----------|------------|--------------|
| **Oracle Cloud Always Free** | 2 VMs (hasta 4 OCPU ARM + 24 GB RAM total) | Proceso de registro algo estricto; puede rechazar tarjetas virtuales. |
| **Fly.io** | Apps pequeñas | No es VPS clásico, pero sirve para desplegar apps. Tiene tier gratuito. |
| **Railway** | Proyectos pequeños | Crédito mensual limitado, no es VPS tradicional. |
| **Render** | Web services | Tier gratuito con sleep tras inactividad. |

### Para practicar sin gastar (local)

| Opción | Descripción |
|--------|-------------|
| **Máquina virtual local** | VirtualBox + Ubuntu Server. Simula un VPS en tu PC. |
| **WSL2 (Windows)** | Linux dentro de Windows. Útil para practicar comandos. |
| **Docker** | Contenedores locales. No sustituye un VPS real para aprender despliegue. |

---

## Qué tener en cuenta al elegir

1. **Ubicación del datacenter:** Elige uno cercano a tus usuarios (o a ti) para menor latencia.
2. **Sistema operativo:** Casi todos ofrecen **Ubuntu 22.04 LTS** o **24.04 LTS** — buena opción por defecto.
3. **RAM mínima recomendada:** 1 GB para algo muy básico; **2 GB** para un proyecto Python con base de datos pequeña.
4. **Backups:** Algunos cobran extra; conviene activarlos o hacer backups manuales.
5. **IPv4:** Algunos proveedores cobran IP extra; verifica que incluya una pública.

---

## Recomendación para empezar

| Objetivo | Sugerencia |
|----------|------------|
| Aprender sin gastar | Oracle Cloud Always Free o VM local con VirtualBox |
| Proyecto real barato | Hetzner o Contabo |
| Máxima documentación en español/inglés | DigitalOcean (tutoriales excelentes) |
| Probar antes de pagar | Crédito de DigitalOcean o Google Cloud |

---

## Checklist antes de comprar

- [ ] Definir presupuesto mensual
- [ ] Elegir ubicación del servidor
- [ ] Verificar que acepta tu método de pago
- [ ] Revisar si incluye IPv4 pública
- [ ] Anotar usuario, contraseña y IP en un gestor de contraseñas
- [ ] Activar autenticación por SSH (clave, no solo contraseña)

---

## Mis pruebas y decisiones

<!-- Escribe aquí tus experiencias: -->

### Proveedores que probé

| Proveedor | Fecha | Resultado | Notas |
|-----------|-------|-----------|-------|
| | | | |

### Proveedor final elegido

- **Nombre:**
- **Plan:**
- **Precio:**
- **Por qué lo elegí:**
