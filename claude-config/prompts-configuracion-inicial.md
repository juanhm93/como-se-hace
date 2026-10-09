# Preparar un proyecto para Claude Code con prompts de configuración inicial

**Antes**: Ninguna.
**Después**: Ninguna.

---

## Objetivo

**Qué se quiere lograr**: que Claude Code entienda un proyecto existente (cómo está armado, qué piezas tiene y cómo se conectan) y deje esa comprensión guardada en un archivo, para que en cada sesión nueva arranque con contexto y no desde cero.

**Por qué importa**: sin contexto, Claude tiene que redescubrir el proyecto en cada conversación, comete errores por suponer cosas y gasta tiempo leyendo archivos que ya leyó antes. Con un buen `CLAUDE.md` las respuestas son más precisas y coherentes con la arquitectura real.

**Para quién / cuándo usar esta guía**: cuando empiezas a usar Claude Code en un proyecto que ya existe (propio o heredado), sobre todo si tiene varias partes (backend, frontend, móvil). No hace falta si el proyecto es un script pequeño de un solo archivo.

> Prompts de ejemplo tomados del curso de Claude Code de Platzi, con la redacción corregida.

---

## El camino en pocas palabras

1. **Analizar**: le pides a Claude que recorra el proyecto y te explique su arquitectura en un "big picture" (vista general).
2. **Guardar**: con ese análisis todavía en la conversación, le pides que lo escriba en `CLAUDE.md`, el archivo que Claude Code lee solo al empezar cada sesión.
3. **Poner a andar**: le pides ayuda para levantar el proyecto en tu máquina, aprovechando lo que ya entendió.
4. **Planificar cambios**: antes de programar una funcionalidad nueva, le pides que analice el impacto en cada parte del sistema.

**Analogía**: es como contratar a alguien nuevo en el equipo. Primero le das un recorrido por la oficina (análisis), luego le dejas un manual de bienvenida en su escritorio (`CLAUDE.md`) para que no tenga que preguntar lo mismo cada mañana, y por último le pides que planifique antes de tocar nada importante.

### Conceptos clave

| Concepto | Qué es (en simple) |
|----------|--------------------|
| Contexto | Todo lo que Claude "tiene presente" en la conversación actual: lo que leyó, lo que le dijiste y lo que respondió. Se pierde al cerrar la sesión. |
| `CLAUDE.md` | Archivo Markdown en la raíz del proyecto que Claude Code carga automáticamente al iniciar. Funciona como su memoria persistente del proyecto. |
| Big picture | Vista general de la arquitectura: qué módulos hay, qué hace cada uno y cómo se comunican. |
| Mención con `@` | Forma de señalarle a Claude un archivo o carpeta concreta (ej. `@backend`) para que la incluya en su análisis. |
| Análisis de impacto | Revisión previa de qué partes del sistema hay que cambiar para agregar una funcionalidad, antes de escribir código. |

---

## Lo que vas a tener al final

- Un análisis de arquitectura del proyecto generado por Claude.
- Un `CLAUDE.md` en la raíz del proyecto que Claude usará como memoria en las próximas sesiones.
- El backend corriendo en local.
- Un plan de impacto por componente para una funcionalidad nueva (ejemplo: sistema de ratings).

---

## Requisitos

- Claude Code instalado y con sesión iniciada.
- Una terminal abierta en la **raíz** del proyecto (donde quieres que quede el `CLAUDE.md`).
- Para el paso 3: las herramientas que use el proyecto para correr (en el ejemplo, Docker).

---

## 1. Analizar la arquitectura del proyecto

Para qué: que Claude construya una vista general del sistema y la deje en el contexto de la conversación, que es la materia prima del `CLAUDE.md`.

Elige la variante según cómo esté organizado tu proyecto.

### Variante A: varios proyectos en un mismo repositorio (monorepo)

Úsala cuando hay subcarpetas que son proyectos independientes. Menciónalas con `@` para que Claude no se quede solo con una.

```text
Analiza el proyecto y entiende cuál es su arquitectura. Es importante que entiendas que hay más de un proyecto contenido en @backend @frontend @mobile/. Utiliza estas carpetas para crear un big picture completo de la arquitectura del sistema.
```

### Variante B: un proyecto, señalando las carpetas importantes

Úsala cuando quieres que Claude se enfoque en ciertas carpetas. Agrega las menciones con `@` al final del prompt.

```text
Analiza el proyecto y entiende cuál es su arquitectura. Utiliza estas carpetas para crear un big picture completo de la arquitectura del sistema: @CARPETA_1 @CARPETA_2
```

### Variante C: análisis general

Úsala en proyectos pequeños o cuando no sabes aún cómo está organizado.

```text
Analiza el proyecto y entiende cuál es su arquitectura. Crea un big picture completo de la arquitectura del sistema.
```

> Los nombres de las carpetas con `@` deben coincidir exactamente con los reales, incluidas mayúsculas (`@Frontend` y `@frontend` no son lo mismo en Linux/macOS).

**Cómo verificar**: Claude responde con una descripción de los módulos, tecnologías, cómo se comunican (API, base de datos, etc.) y la estructura de carpetas. Revisa que no haya omitido ninguna parte importante; si falta algo, pídele que lo analice antes de pasar al paso 2.

---

## 2. Guardar el análisis en `CLAUDE.md`

Para qué: convertir lo que está en el contexto (que se pierde al cerrar la sesión) en memoria persistente que Claude cargará solo en cada sesión nueva.

Hazlo **en la misma conversación** del paso 1, para que Claude reutilice el análisis sin volver a leer todo.

Si el paso 1 fue de arquitectura:

```text
Gracias, excelente análisis de arquitectura. Ahora, con esta información que tienes en el contexto, crea el archivo CLAUDE.md para que se pueda utilizar como memoria durante el resto del desarrollo de este proyecto.
```

Versión genérica:

```text
Gracias, excelente análisis. Ahora, con esta información que tienes en el contexto, crea el archivo CLAUDE.md para que se pueda utilizar como memoria durante el resto del desarrollo de este proyecto.
```

> Alternativa: Claude Code tiene el comando `/init`, que analiza el proyecto y genera un `CLAUDE.md` en un solo paso. Los prompts de esta guía dan más control porque primero revisas el análisis y luego decides guardarlo.

**Cómo verificar**:

```bash
ls CLAUDE.md && head -40 CLAUDE.md
```

El archivo existe en la raíz y describe la arquitectura, comandos para correr el proyecto y convenciones. Léelo completo y corrige lo que esté mal: lo que diga ahí, Claude lo tomará como verdad en las próximas sesiones.

---

## 3. Levantar el backend en local

Para qué: aprovechar el contexto ya cargado para que Claude te guíe a correr el proyecto, en lugar de buscar las instrucciones a mano.

```text
Ahora ayúdame a tener el servicio de backend corriendo en mi local. Verás que ya tengo Docker instalado y listo para correr.
```

> Cambia "Docker" por la herramienta que tengas instalada (ej. PHP + Composer, Node, Python). Decirle qué ya tienes evita que te proponga instalar cosas innecesarias.

**Cómo verificar**: el servicio arranca sin errores y responde (ej. abrir `http://localhost:PUERTO` o hacer `curl http://localhost:PUERTO/RUTA_DE_SALUD`). Si Claude descubrió pasos que no estaban documentados, pídele que los agregue al `CLAUDE.md`.

---

## 4. Analizar el impacto de una funcionalidad nueva

Para qué: obtener un plan por componente antes de programar, para no descubrir a mitad de camino que un cambio rompe otra parte del sistema.

```text
Think deeply. Necesito implementar un sistema de ratings en este proyecto; el rating de un curso puede ir de 1 a 5 estrellas. Tu tarea es analizar el impacto que va a tener la implementación de este feature en el proyecto. Analiza qué acciones deben hacerse en cada uno de los componentes del proyecto (backend, frontend) para implementar el rating.
```

Qué hace cada parte del prompt:

| Parte | Por qué está |
|-------|--------------|
| `Think deeply` | Le pide a Claude que razone con más profundidad antes de responder. Útil en análisis con muchas piezas. |
| Regla de negocio (1 a 5 estrellas) | Le da un dato concreto para que no tenga que suponer el rango. |
| "Analiza el impacto" | Deja claro que quieres un plan, no código todavía. |
| Componentes nombrados | Obliga a revisar cada parte del sistema por separado. |

> Agrega `mobile` a la lista de componentes si tu proyecto lo tiene. Para otra funcionalidad, cambia la descripción y las reglas de negocio manteniendo la misma estructura.

**Cómo verificar**: Claude entrega acciones por componente (ej. backend: migración, modelo, endpoint, validación del rango; frontend: componente de estrellas, llamada a la API, mostrar promedio). No debería haber modificado archivos; si lo hizo, revisa con `git status`.

---

## Checklist rápido

- [ ] Análisis de arquitectura revisado y completo
- [ ] `CLAUDE.md` creado en la raíz y corregido a mano
- [ ] Backend corriendo en local
- [ ] Plan de impacto de la funcionalidad revisado antes de programar

---

## Problemas frecuentes

| Síntoma | Qué revisar |
|---------|-------------|
| El análisis ignora una parte del proyecto | Menciónala explícitamente con `@CARPETA` (variante A o B del paso 1). |
| El `CLAUDE.md` queda genérico o vacío de detalles | Probablemente se pidió en una sesión nueva, sin el análisis en el contexto. Haz los pasos 1 y 2 en la misma conversación. |
| El `CLAUDE.md` se creó en una subcarpeta | Claude Code se abrió desde esa subcarpeta. Ciérralo y ábrelo desde la raíz del proyecto. |
| Claude sigue usando información vieja del proyecto | El `CLAUDE.md` está desactualizado. Edítalo o pídele que lo actualice con los cambios recientes. |
| En el paso 4 Claude empieza a escribir código | Recuérdale que solo quieres el análisis, sin modificar archivos. |

---

## Notas de prueba (rellenar después de experimentar)

- **Fecha**:
- **Entorno** (proveedor / OS / versión):
- **Qué funcionó**:
- **Qué no funcionó**:
- **Pasos extra que hiciste**:
- **Enlaces útiles**:
