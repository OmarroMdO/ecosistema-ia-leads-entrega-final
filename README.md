# Ecosistema IA Leads - Entrega Final

Trabajo final: **Ecosistema de Automatización IA Autónomo para gestión de leads**.

El proyecto automatiza parte del proceso de gestión comercial de leads usando Airtable, Make, OpenAI y Gmail, manteniendo una instancia de revisión humana antes del envío final.

## Caso de uso

El sistema recibe leads en Airtable, analiza la información con inteligencia artificial, clasifica la prioridad comercial y genera una respuesta sugerida.

La respuesta no se envía automáticamente. Primero queda disponible para revisión humana en Airtable. Cuando una persona aprueba el lead, Make envía la respuesta por Gmail y actualiza el estado del registro.

## Arquitectura general

Flujo principal:

Airtable → Make → OpenAI → Airtable → Revisión humana → Gmail → Airtable

Componentes:

- **Airtable:** base de datos, dashboard, revisión humana y registro de errores.
- **Make:** orquestador principal del escenario.
- **OpenAI:** clasificación de prioridad y generación de respuesta sugerida.
- **Gmail:** canal de envío una vez aprobada la respuesta.
- **HITL:** aprobación manual antes del envío final.

## Funcionamiento del escenario

El escenario de Make comienza con un módulo de Airtable que observa registros de la tabla Leads mediante el campo **Última modificación**.

Luego, un Router divide el flujo en dos rutas:

1. **Procesamiento con IA**
   - Se ejecuta cuando el lead está en estado Pendiente.
   - OpenAI analiza el lead y genera una respuesta sugerida.
   - Airtable se actualiza con la respuesta IA y el estado Procesado por IA.

2. **Envío aprobado por humano**
   - Se ejecuta solo cuando el lead fue aprobado manualmente.
   - Requiere: Aprobado marcado, Estado Aprobado por humano, Estado de envío No enviado y Error distinto de true.
   - Gmail envía la respuesta.
   - Airtable actualiza el estado de envío a Enviado.

## Manejo de errores

La versión corregida incorpora Error Handlers en los módulos críticos:

- OpenAI
- Gmail

Cuando ocurre un error, Make registra automáticamente el fallo en la tabla Errores de Airtable.

El registro incluye:

- Fecha de error
- Lead relacionado
- Módulo afectado
- Tipo de error
- Detalle del error
- Estado del error

También se agregó una condición para evitar reprocesar registros marcados con error.

## Costos y modelos

La matriz de decisión de costos y modelos se entrega como PDF independiente dentro de la carpeta `documentacion`.

Allí se comparan modelos como GPT-4.1, GPT-4.1 mini, GPT-4o mini, Claude Haiku 4.5 y Batch API, considerando costo, uso recomendado y conveniencia operativa.

## Estructura del repositorio

```text
documentacion/
  documentacion_entrega_final-Omar Rodriguez.pdf
  matriz_costos_modelos.pdf

blueprint/
  make_blueprint_final.json

evidencias/
  capturas del escenario, pruebas exitosas, pruebas de error y link al dashboard de Airtable
