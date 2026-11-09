# Radar IA

Sistema que recoge links que el usuario manda a un bot de Telegram durante la semana, los analiza con varios agentes y envía un informe didáctico por email cada lunes. Opcionalmente (`OBSIDIAN_MEMORY=true`) construye una base de conocimiento en Obsidian dentro de `vault/`. Corre en GitHub Actions, sin servidor ni base de datos. Nombre provisional.

## Contexto de diseño (léelo al empezar cada sesión)

El diseño y el plan viven en el **vault personal** del usuario, en `"C:/Users/Usuario/Documents/Cerebro/01-Proyectos/resumidor-novedades"`, que se añade con `--add-dir` o `additionalDirectories`. Empieza por:

- `Resumidor - Diseño y decisiones.md`: arquitectura, agentes, reglas del vault, configuración.
- `Resumidor - Plan de construcción.md`: hitos con criterios de "hecho cuando".
- `Resumidor - Registro de decisiones.md`: cronología de decisiones.

Si no tienes acceso a esa carpeta, pídele al usuario que la añada y no inventes el diseño.

## Documentación del proyecto

Dentro de esa carpeta, documenta lo que construyas así:

- Arquitectura y contratos de datos: `01-Arquitectura/`.
- Workflows, secrets y costes: `02-Infraestructura/`.
- Una nota por agente en `03-Agentes/`, creada con `99-Plantillas/Resumidor - Plantilla de agente.md`.
- Diseño de las notas del vault generado: `04-Vault del producto/`.
- Al cerrar un hito, actualiza su estado en `05-Hitos/Resumidor - Hitos.md`.
- Una nota por sesión en `06-Diario/` (`Resumidor - Diario AAAA-MM-DD`), con la plantilla de sesión.
- Todas las notas empiezan por `Resumidor - `. No crees carpetas nuevas fuera de esta estructura sin preguntar.

## Dos vaults: no los confundas

- **Vault personal:** fuera del repo. Solo lees y escribes en la carpeta de este proyecto (documentos de diseño y registro de decisiones). No toques nada más.
- **Vault del producto:** la carpeta `vault/` del repo. Es la salida del sistema, la escriben los agentes Clasificador y Enlazador. No la edites a mano salvo que el usuario lo pida, y nunca la mezcles con el vault personal.

## Arquitectura en una pantalla

- Captura (cron ~4 h): bot por sondeo, filtro por `TELEGRAM_CHAT_ID`, Extractor, ítems en ficheros.
- Informe (cron lunes): Analista → Curador → (Clasificador) → Redactor → Verificador → email → (Enlazador).
- Aprendizaje: Actualizador reescribe `profile.md` con el feedback del usuario.
- Los agentes entre paréntesis solo se ejecutan con `OBSIDIAN_MEMORY=true`.
- Haiku para Extractor y Analista. Sonnet para el resto. Modelos configurables por agente.

## Reglas

- Lo determinista (bot, almacenamiento, envío, validación) es código normal, no agentes.
- Todas las llamadas a modelos pasan por `llm.py`. Los tests usan respuestas simuladas.
- Cada agente tiene su prompt en un fichero propio y una salida validada con esquema.
- Nada entra en el vault sin que el Verificador haya aprobado el informe.
- Antes de guardar una nota del vault, el validador comprueba que cada `[[enlace]]` apunte a una nota existente y que las propiedades estén completas.
- El Analista nunca descarta ítems. Descartar es decisión del Curador.
- Nunca commitear `.env` ni claves. Los ajustes secretos van en secrets de GitHub.
- Ignorar todo mensaje que no venga del `TELEGRAM_CHAT_ID` configurado.
- Registrar el coste estimado de cada ejecución.

## Convenciones

- Python, tipado y validación con pydantic, `pytest` y `ruff`. (Si el usuario decide otra cosa, anótalo en el registro de decisiones.)
- Informe HTML en español, tono didáctico, cada afirmación con enlace al original.
- Código y comentarios en español salvo identificadores.

## Forma de trabajar

- Un hito del plan por sesión. Antes de empezar, resume en pocas líneas lo que vas a hacer.
- Primero tests de lo determinista, después los agentes.
- Si una decisión de diseño cambia, no la cambies en silencio: pregunta, y después añade una entrada a `Registro de decisiones.md` con fecha y motivo.
- Al terminar un hito, comprueba su criterio de "hecho cuando" y dilo explícitamente.
