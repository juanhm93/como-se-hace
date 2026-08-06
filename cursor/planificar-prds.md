Convertir un requerimiento ambiguo en un documento técnico accionable es una de las tareas más complicadas del desarrollo. Con Cursor y su modo agente puedes transformar un problema difuso en un PRD claro, listo para implementación, sin escribir el documento tú mismo. Esta guía es para desarrolladores que quieren delegar tareas de producto en un agente de IA.

Qué es un PRD y por qué importa en desarrollo de software

El Product Requirement Document es el puente entre negocio e ingeniería. Cuando tu jefe llega con un problema incompleto, un PRD bien estructurado te ahorra semanas de retrabajo.


ejemplo:

Prompt 1 — Encuadre
None
Actúa como un Product Manager senior.
Voy a darte un requerimiento de negocio ambiguo y quiero que me
ayudes a convertirlo en un PRD claro y suficientemente
implementable. No escribas el PRD todavía. Primero quiero que
identifiques las decisiones de producto que faltan por definir.
Requerimiento: "Necesitamos una herramienta interna para activar o
desactivar features por empresa, ambiente o porcentaje de tráfico,
sin tener que hacer deploy cada vez.
"
Lístame las 6-8 decisiones clave que debemos resolver antes de
escribir el PRD, con su propuesta de resolución y agrupadas por 
categoría(alcance, usuarios, dominio, técnico). Sé conciso.

Prompt 2 — Resolver ambigüedad
None
Acepto casi todo. Cambia esto: el alcance NO incluye OAuth, roles
ni permisos avanzados; el login es solo un usuario demo. El
targeting debe soportar ambiente, empresa y rollout porcentual.
Persistencia local con SQLite. Ahora bloquea estas decisiones como
definitivas.


Prompt 3 — Generar el PRD
None
Ahora escribe el PRD completo en Markdown con esta estructura
exacta: 1) Contexto y problema, 2) Objetivo, 3) Público objetivo y
usuarios, 4) Alcance (In scope / Out of scope explícito), 5)
Conceptos de dominio (qué es una flag, una targeting rule, el
evaluador), 6) Requerimientos funcionales numerados, 7)
Requerimientos no funcionales, 8) Criterios de aceptación del MVP,
9) Riesgos y supuestos.
Usa lenguaje claro y conciso. Que cada requerimiento funcional sea
verificable.
Guarda el PRD en docs/prds