# Configurar Mailpit para probar el envío de correos en local (Laravel)

**Antes**: Ninguna.
**Después**: Ninguna.

---

## Objetivo

**Qué se quiere lograr**: que todos los correos que envía un proyecto en tu computador lleguen a una bandeja de prueba local, donde puedes verlos tal como los recibiría un usuario, sin que ninguno salga a internet.

**Por qué importa**:

- Los sandbox en la nube, como Mailtrap en su plan gratis, tienen un **límite de envíos**. Cuando lo alcanzas, dejas de poder probar hasta que se reinicia.
- Evitas mandarle por accidente un correo de prueba a un cliente o a un usuario real.
- Puedes enviar todos los correos que quieras, a cualquier dirección, y revisarlos al instante.

**Para quién / cuándo usar esta guía**: cuando desarrollas en local un proyecto que envía correos (registro, recuperar contraseña, notificaciones, facturas...) y quieres revisar su contenido y su diseño. **No** sirve para producción: Mailpit no entrega correos de verdad, solo los atrapa.

---

## El camino en pocas palabras

1. **Levantar Mailpit**: un pequeño programa que corre en tu computador y hace de "servidor de correo" falso. Lo arrancas con Docker o lo instalas con Homebrew.
2. **Apuntar el proyecto a Mailpit**: en el `.env` del proyecto le dices a Laravel que, en vez de mandar los correos a Mailtrap, Gmail o SES, se los entregue a Mailpit.
3. **Reiniciar el proyecto**: los procesos que ya estaban corriendo leyeron la configuración vieja al arrancar. Hay que reiniciarlos para que tomen la nueva.
4. **Enviar y revisar**: disparas un correo desde el proyecto y lo abres en la bandeja web de Mailpit (`http://localhost:8025`).

**Analogía**: Mailpit es una **oficina de correos de mentira** dentro de tu casa. Tu proyecto le entrega las cartas creyendo que las va a enviar, pero la oficina no las despacha: las deja en una vitrina para que tú las leas. Por eso puedes escribirle a quien sea; nada sale de la casa.

### Conceptos clave

| Concepto | Qué es (en simple) |
|----------|--------------------|
| SMTP | El "idioma" que usan los programas para entregarse correos entre sí. |
| Mailpit | Servidor SMTP falso que recibe los correos y los muestra en una página web, sin enviarlos. |
| Mailtrap | Servicio en la nube que hace lo mismo que Mailpit, pero con límite de envíos en el plan gratis. |
| Puerto 1025 | La "puerta" por la que Mailpit recibe los correos (SMTP). |
| Puerto 8025 | La "puerta" de la bandeja web de Mailpit, la que abres en el navegador. |
| Docker | Programa que ejecuta aplicaciones dentro de "cajas" aisladas (contenedores) sin instalarlas en tu sistema. |
| `.env` | Archivo del proyecto con la configuración de tu entorno (credenciales, servidores, puertos). |
| Mailer | En Laravel, el "transporte" que se usa para enviar: `smtp`, `log`, `ses`, etc. |

---

## Lo que vas a tener al final

- Mailpit corriendo en tu computador y arrancando solo cada vez que abres Docker Desktop.
- Tu proyecto Laravel enviando todos los correos a Mailpit.
- La bandeja de prueba en `http://localhost:8025`, con los correos renderizados (HTML, texto, cabeceras y adjuntos).
- El bloque de Mailtrap comentado en el `.env`, listo para volver a usarlo cuando quieras.

---

## Requisitos

- macOS con **Docker Desktop** instalado (opción principal) o **Homebrew** (opción sin Docker).
- Un proyecto **Laravel** en local con PHP funcionando (por ejemplo, con Laravel Herd).
- Los puertos **1025** y **8025** libres en tu computador.

---

## 1. Arrancar Docker Desktop

Para qué: Mailpit corre dentro de Docker, así que el "motor" de Docker tiene que estar encendido antes de crear el contenedor.

### Opción A: por terminal

```bash
open -a Docker
```

Espera uno o dos minutos y comprueba que el motor responde:

```bash
docker info --format '{{.ServerVersion}}'
```

### Opción B: por la interfaz (visual)

**Ruta**: `Aplicaciones` > `Docker` > esperar el estado `Engine running`

1. Abre **Docker Desktop** desde Aplicaciones o Spotlight.
2. Si aparece alguna ventana (aceptar términos, actualización, iniciar sesión), atiéndela. **Mientras no lo hagas, el motor no arranca**, aunque `open -a Docker` no muestre ningún error.
3. Espera a ver **Engine running** abajo a la izquierda.

> Recomendado: en **Settings** > **General**, activa **Start Docker Desktop when you sign in to your computer**. Así Mailpit queda disponible al encender el Mac, sin abrir nada a mano.

**Cómo verificar**: `docker info` devuelve un número de versión, sin errores de conexión, y Docker Desktop muestra **Engine running**.

---

## 2. Levantar el contenedor de Mailpit

Para qué: dejar la "oficina de correos de mentira" funcionando en segundo plano y que se vuelva a encender sola.

### Opción A: por terminal

```bash
docker run -d \
  --name mailpit \
  --restart unless-stopped \
  -p 1025:1025 \
  -p 8025:8025 \
  axllent/mailpit
```

Qué hace cada parte:

| Parte | Para qué sirve |
|-------|----------------|
| `-d` | Lo deja corriendo en segundo plano. |
| `--name mailpit` | Le pone un nombre fijo para poder pararlo, arrancarlo o borrarlo fácilmente. |
| `--restart unless-stopped` | Lo vuelve a levantar cada vez que arranca Docker, salvo que lo hayas parado tú. |
| `-p 1025:1025` | Expone el puerto SMTP, por donde Laravel entrega los correos. |
| `-p 8025:8025` | Expone la bandeja web. |
| `axllent/mailpit` | La imagen oficial de Mailpit. |

> **Persistencia opcional**: por defecto, los correos se pierden si borras el contenedor (`docker rm`). Si quieres conservarlos, añade un volumen y la base de datos:
>
> ```bash
> docker run -d --name mailpit --restart unless-stopped \
>   -p 1025:1025 -p 8025:8025 \
>   -v mailpit-data:/data -e MP_DATABASE=/data/mailpit.db \
>   axllent/mailpit
> ```

### Opción B: por la interfaz de Docker Desktop (visual)

**Ruta**: barra de búsqueda `Search` > `axllent/mailpit` > botón `Run` > `Optional settings`

1. En Docker Desktop, busca **axllent/mailpit** en la barra superior y pulsa **Run** (la primera vez descarga la imagen).
2. Despliega **Optional settings**:
   - **Container name**: `mailpit`
   - **Ports**: host `1025` para el puerto `1025/tcp` y host `8025` para el puerto `8025/tcp`.
3. Pulsa **Run**.

> Desde la interfaz no se puede configurar `--restart unless-stopped`. Si quieres que arranque solo, usa la opción A.

**Cómo verificar**:

```bash
docker ps --filter name=mailpit
```

Debe aparecer el contenedor con estado `Up` y los puertos `0.0.0.0:1025->1025/tcp, 0.0.0.0:8025->8025/tcp`. Al abrir `http://localhost:8025` debes ver la bandeja de Mailpit vacía.

### Alternativa sin Docker: Homebrew

Si Docker te da problemas, Mailpit se instala como un programa normal y funciona igual, con el mismo `.env`:

```bash
brew install mailpit
brew services start mailpit   # lo deja corriendo y arrancando con el sistema
```

Para verificar, abre `http://localhost:8025`. Para pararlo: `brew services stop mailpit`.

> No uses las dos vías a la vez: ambas usan los puertos 1025 y 8025 y la segunda no podrá arrancar.

---

## 3. Configurar el `.env` del proyecto

Para qué: decirle a Laravel que entregue los correos a Mailpit en vez de al servicio que tenía.

Abre el `.env` del proyecto. **Comenta** el bloque de Mailtrap (o el que tengas) en lugar de borrarlo, para recuperarlo después, y añade el de Mailpit:

```env
# --- Mailtrap (sandbox, plan free con límite de envíos) ---
# MAIL_MAILER=smtp
# MAIL_SCHEME=null
# MAIL_HOST=sandbox.smtp.mailtrap.io
# MAIL_PORT=587
# MAIL_USERNAME=TU_USUARIO_MAILTRAP
# MAIL_PASSWORD=TU_PASSWORD_MAILTRAP

# --- Mailpit local: bandeja en http://localhost:8025 ---
MAIL_MAILER=smtp
MAIL_SCHEME=null
MAIL_HOST=127.0.0.1
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null

# --- Alternativa sin servidor: los correos se escriben en storage/logs/laravel.log ---
# MAIL_MAILER=log

MAIL_FROM_ADDRESS="TU_CORREO@ejemplo.com"
MAIL_FROM_NAME="${APP_NAME}"
```

Reglas importantes:

- **Solo puede haber un `MAIL_MAILER` sin comentar.** Si hay dos, gana el último y te vas a confundir. Para cambiar de alternativa, comenta un bloque y descomenta el otro.
- **Mailpit no usa credenciales**: `MAIL_USERNAME` y `MAIL_PASSWORD` van en `null`. Si dejas las de Mailtrap, Mailpit las ignora, pero es mejor dejarlo limpio.
- **`MAIL_SCHEME=null`**: Mailpit acepta SMTP sin cifrado (TLS), y en local no hace falta.
  - En **Laravel 10 o anterior**, esta variable se llama `MAIL_ENCRYPTION=null`.
- **`MAIL_HOST=127.0.0.1`** sirve si PHP corre directamente en tu Mac (Herd, `php artisan serve`). Si tu app corre **dentro de Docker** (Sail, docker-compose), usa el nombre del servicio (`mailpit`) o `host.docker.internal`.
- **Nunca subas el `.env` al repositorio**: tiene credenciales reales. Comprueba que está en `.gitignore`.

**Cómo verificar**:

```bash
php artisan config:show mail.mailers.smtp
```

Debe mostrar `host` `127.0.0.1` y `port` `1025`.

---

## 4. Reiniciar el proyecto

Para qué: que todos los procesos que ya estaban corriendo lean la configuración nueva.

1. Detén `composer run dev` (o lo que uses: `php artisan serve`, `queue:work`, Horizon) con `Ctrl + C` y vuelve a arrancarlo:

   ```bash
   composer run dev
   ```

2. Si alguna vez cacheaste la configuración, límpiala:

   ```bash
   php artisan config:clear
   ```

   Para saber si está cacheada, mira si existe `bootstrap/cache/config.php`. Si existe, Laravel **no lee el `.env`** hasta que la limpias.

> **Por qué hace falta reiniciar**: un `php artisan queue:work` (y Horizon) carga la configuración una sola vez al arrancar y la mantiene en memoria. Si no lo reinicias, los correos que van por cola seguirán saliendo por el servicio anterior, aunque los que envías al momento ya lleguen a Mailpit. `queue:listen` relee la configuración en cada job, pero reiniciar todo es la forma segura de no dudar.

**Cómo verificar**: el proyecto vuelve a cargar en el navegador y la cola está procesando, sin errores en la terminal.

---

## 5. Enviar un correo de prueba y revisarlo

Para qué: confirmar de punta a punta que los correos salen del proyecto y llegan a la bandeja.

### Opción A: por terminal

Envío rápido con Tinker:

```bash
php artisan tinker
```

```php
Mail::raw('Hola desde Mailpit', fn ($m) => $m->to('prueba@ejemplo.com')->subject('Prueba Mailpit'));
```

> Si tu proyecto tiene su propio comando de prueba, úsalo. Por ejemplo, un `php artisan mail:send-test --sync` que dispare todas las plantillas en cada idioma. `--sync` (si el comando lo tiene) envía sin pasar por la cola, así descartas problemas del worker.

Comprueba con la API de Mailpit cuántos correos llegaron y cuáles son:

```bash
# total de correos en la bandeja
curl -s http://localhost:8025/api/v1/messages | jq '.total'

# asunto y remitente de cada uno
curl -s http://localhost:8025/api/v1/messages | jq -r '.messages[] | "\(.Subject) — \(.From.Address)"'

# vaciar la bandeja
curl -s -X DELETE http://localhost:8025/api/v1/messages
```

> Si no tienes `jq`: `brew install jq`.

### Opción B: por la interfaz web (visual)

**Ruta**: navegador > `http://localhost:8025` > clic en el correo > pestañas `HTML` / `Text` / `Headers` / `Raw`

1. Abre `http://localhost:8025`.
2. Haz clic en el correo para verlo renderizado.
3. Revisa las pestañas:
   - **HTML**: cómo lo ve el usuario.
   - **Text**: la versión de texto plano.
   - **Headers**: remitente, destinatario y cabeceras.
   - **HTML Check**: compatibilidad con clientes de correo.
   - **Link Check**: enlaces rotos.
4. Para vaciar la bandeja, usa el botón **Delete all**.

**Cómo verificar**: el correo aparece en la bandeja con el asunto y el remitente (`MAIL_FROM_ADDRESS`) esperados, y se ve bien renderizado.

---

## Volver a Mailtrap (u otro servicio)

En el `.env`, comenta el bloque de Mailpit, descomenta el de Mailtrap y reinicia el proyecto (paso 4). No hace falta parar Mailpit: puede quedarse corriendo sin molestar.

---

## Comandos útiles del contenedor

```bash
docker stop mailpit       # pararlo (no se vuelve a levantar solo hasta que hagas start)
docker start mailpit      # arrancarlo de nuevo
docker logs -f mailpit    # ver lo que está pasando
docker rm -f mailpit      # borrarlo (se pierden los correos si no usaste volumen)
docker pull axllent/mailpit   # actualizar la imagen (luego borra y vuelve a crear el contenedor)
```

---

## Checklist rápido

- [ ] Docker Desktop muestra **Engine running** (o Mailpit instalado con Homebrew).
- [ ] `docker ps` muestra el contenedor `mailpit` en estado `Up`.
- [ ] `http://localhost:8025` abre la bandeja.
- [ ] `.env`: bloque anterior comentado, bloque de Mailpit activo y un solo `MAIL_MAILER` sin comentar.
- [ ] `composer run dev` (y la cola) reiniciados y sin configuración cacheada.
- [ ] Un correo de prueba aparece en la bandeja con el asunto y el remitente correctos.

---

## Problemas frecuentes

| Síntoma | Qué revisar |
|---------|-------------|
| `open -a Docker` no da error, pero `docker info` dice `Cannot connect to the Docker daemon` | Docker Desktop está esperando algo en pantalla: términos, actualización o inicio de sesión. Ábrelo, atiéndelo y espera a **Engine running**. Si sigue sin arrancar, reinicia el Mac o usa la vía de Homebrew. |
| `docker run` falla con `port is already allocated` | Otro programa ocupa el puerto 1025 u 8025 (otro Mailpit, el de Homebrew, MailHog...). Búscalo con `lsof -i :1025` y `lsof -i :8025`. Para usar otros puertos: `-p 1026:1025` y pon `MAIL_PORT=1026`. |
| `Conflict. The container name "/mailpit" is already in use` | Ya lo creaste antes. Arráncalo con `docker start mailpit` o bórralo con `docker rm -f mailpit` y vuelve a crearlo. |
| Laravel: `Connection could not be established with host 127.0.0.1:1025` | Mailpit no está corriendo (revisa `docker ps`) o la app corre dentro de Docker y debe usar `mailpit` o `host.docker.internal` como `MAIL_HOST`. |
| Los correos siguen llegando a Mailtrap | No reiniciaste la cola o `composer run dev`, o la configuración está cacheada. Ejecuta `php artisan config:clear` y reinicia (paso 4). |
| Los correos enviados al momento llegan, pero los de la cola no | El worker (`queue:work` u Horizon) sigue con la configuración vieja. Reinícialo o ejecuta `php artisan queue:restart`. |
| Error de TLS / `STARTTLS` | `MAIL_SCHEME` (o `MAIL_ENCRYPTION`) no está en `null`. |
| La bandeja está vacía y no hay errores | Revisa que no tengas `MAIL_MAILER=log` activo más abajo en el `.env`: gana el último. Mira `storage/logs/laravel.log`. |

---

## Notas de prueba (rellenar después de experimentar)

- **Fecha**:
- **Entorno** (proveedor / OS / versión):
- **Qué funcionó**:
- **Qué no funcionó**:
- **Pasos extra que hiciste**:
- **Enlaces útiles**:
