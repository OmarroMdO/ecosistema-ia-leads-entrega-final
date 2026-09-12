# Ecosistema IA Leads - Entrega Final

Trabajo final: **Ecosistema de Automatización IA Autónomo para gestión de leads**.

El proyecto automatiza parte del proceso de gestión comercial de leads usando Airtable, Make, OpenAI y Gmail. El flujo clasifica leads con IA, genera una respuesta sugerida, solicita intervención humana y envía la respuesta final solo después de la aprobación.

## Guía principal de revisión

Para facilitar la corrección, se agregó una guía específica con instrucciones de ejecución y mapa de evidencias por criterio de la rúbrica:

- [Abrir guía de revisión, rúbrica y ejecución](documentacion/guia_revision_rubrica_ejecucion.md)

## Arquitectura general

El ecosistema utiliza:

- **Airtable:** base de datos, dashboard, aprobación humana y registro de errores.
- **Make:** orquestador principal del escenario.
- **OpenAI:** clasificación de prioridad comercial y generación de respuesta sugerida.
- **Gmail:** aviso al revisor humano y envío final al lead.
- **HITL:** intervención humana antes del envío final.

Flujo general:

Airtable → Router → OpenAI → Airtable → Gmail aviso humano → Sleep → Airtable Get a Record → Router HITL

Desde el Router HITL:

- Si el lead fue aprobado: Gmail envío final → Airtable enviado.
- Si el lead no fue aprobado: Airtable standby.

## Funcionamiento del escenario

El escenario de Make comienza con **Airtable Watch Records** sobre la tabla `Leads`.

El disparador utiliza el campo **Última modificación**, lo que permite detectar registros nuevos y cambios posteriores en los leads.

Funcionamiento principal:

1. Airtable detecta un lead en estado `Pendiente`.
2. Router 12 permite avanzar solo leads pendientes y sin error.
3. OpenAI analiza el lead, clasifica la prioridad comercial y genera una respuesta sugerida.
4. Airtable 4 actualiza el lead con la respuesta IA y el estado `Procesado por IA`.
5. Gmail 21 envía un aviso al revisor humano.
6. Tools 22 pausa el flujo durante 5 minutos.
7. Airtable 23 vuelve a leer el mismo lead.
8. Router 24 evalúa si hubo aprobación humana.
9. Si el lead fue aprobado, Gmail 25 envía la respuesta final.
10. Airtable 27 marca el lead como `Enviado`.
11. Si el lead no fue aprobado, Airtable 26 lo deja en estado `Esperando aprobación humana`.

## HITL implementado

El HITL fue ajustado según la devolución del profesor.

La respuesta al lead no se envía inmediatamente después de la IA. Primero se envía un aviso al humano revisor, el flujo espera 5 minutos, vuelve a leer Airtable y recién ahí decide si continúa o queda en espera.

Componentes del HITL:

- **Gmail 21:** aviso al revisor humano.
- **Tools 22:** pausa de espera de 5 minutos.
- **Airtable 23:** relectura del lead.
- **Router 24:** decisión según aprobación humana.
- **Gmail 25:** envío final al lead solo si fue aprobado.
- **Airtable 26:** estado standby si no fue aprobado.

Condiciones de la rama aprobada:

- `Aprobado = true`
- `Estado = Aprobado por humano`
- `Estado de envio = No enviado`
- `Error != true`

Evidencias principales:

- `evidencias/10_make_flujo_hitl_aviso_espera_revision.png`
- `evidencias/12_email_aviso_humano_hitl.png`
- `evidencias/13_make_ejecucion_hitl_aprobado_exitosa.png`
- `evidencias/14_airtable_hitl_aviso_visible_enviado.png`
- `evidencias/15_email_respuesta_final_hitl.png`

## Interfaz KPI

Se creó una interfaz en Airtable llamada **Dashboard KPI Leads** para visualizar indicadores del ecosistema.

La interfaz muestra:

- Total de leads.
- Distribución por prioridad.
- Datos visibles del estado general del sistema.

Evidencia:

- `evidencias/09_airtable_interfaz_kpi_dashboard.png`

El link al dashboard/base de Airtable queda documentado en:

- `evidencias/link_airtable_dashboard_control.txt`

## Manejo de errores

El escenario incluye Error Handlers en módulos críticos.

Módulos con manejo de error:

- `OpenAI 3 → Airtable 19 → Retry 20`
- `Gmail 25 → Airtable 28 → Retry 29`

Cuando ocurre un error, Make registra automáticamente el fallo en la tabla `Errores` de Airtable.

Datos registrados:

- Fecha de error.
- Lead relacionado.
- Módulo afectado.
- Tipo de error.
- Detalle del error.
- Estado del error.

En OpenAI se configuró retry automático por posibles errores temporales de API.

En Gmail 25 se dejó retry manual para evitar reintentos innecesarios ante errores como correo inválido o dato mal cargado.

## Matriz de costos y modelos

La matriz de decisión de costos y modelos se entrega como PDF independiente:

- `documentacion/matriz_costos_modelos.pdf`

El documento compara modelos según costo aproximado, uso recomendado y conveniencia operativa.

Modelos contemplados:

- GPT-4.1
- GPT-4.1 mini
- GPT-4o mini
- Claude Haiku 4.5
- Batch API

## Estructura del repositorio

| Carpeta | Contenido |
|---|---|
| `documentacion/` | Documentación principal, matriz de costos y guía de revisión |
| `blueprint/` | Blueprint final exportado desde Make |
| `evidencias/` | Capturas de flujo, pruebas, errores, KPI y HITL final |

Archivos principales:

| Archivo | Descripción |
|---|---|
| `documentacion/documentacion_entrega_final-Omar Rodriguez.pdf` | Documentación principal del ecosistema |
| `documentacion/matriz_costos_modelos.pdf` | Matriz de decisión de costos y modelos |
| `documentacion/guia_revision_rubrica_ejecucion.md` | Guía de revisión, ejecución y mapa de rúbrica |
| `blueprint/make_blueprint_final.json` | Blueprint actualizado del escenario final |

## Evidencias destacadas

| Evidencia | Archivo |
|---|---|
| Flujo HITL con aviso, espera y decisión | `evidencias/10_make_flujo_hitl_aviso_espera_revision.png` |
| Aviso humano recibido por Gmail | `evidencias/12_email_aviso_humano_hitl.png` |
| Ejecución HITL aprobada exitosa en Make | `evidencias/13_make_ejecucion_hitl_aprobado_exitosa.png` |
| Airtable con lead aprobado y enviado | `evidencias/14_airtable_hitl_aviso_visible_enviado.png` |
| Email final enviado al lead | `evidencias/15_email_respuesta_final_hitl.png` |
| Interfaz KPI en Airtable | `evidencias/09_airtable_interfaz_kpi_dashboard.png` |
| Prueba de error Gmail | `evidencias/06_make_prueba_error_gmail.png` |
| Registro automático de error en Airtable | `evidencias/07_airtable_registro_error_gmail.png` |

## Instrucciones para revisar

1. Abrir la guía principal: `documentacion/guia_revision_rubrica_ejecucion.md`.
2. Revisar la documentación principal: `documentacion/documentacion_entrega_final-Omar Rodriguez.pdf`.
3. Revisar la matriz de costos: `documentacion/matriz_costos_modelos.pdf`.
4. Revisar el blueprint actualizado: `blueprint/make_blueprint_final.json`.
5. Revisar las evidencias dentro de `evidencias/`.
6. Revisar la interfaz KPI en `evidencias/09_airtable_interfaz_kpi_dashboard.png`.
7. Revisar la prueba HITL final en las capturas 10, 12, 13, 14 y 15.

## Instrucciones para ejecutar el escenario

1. Importar el archivo `blueprint/make_blueprint_final.json` en Make.
2. Reconectar las conexiones propias de Airtable, OpenAI y Gmail.
3. Verificar que la base de Airtable tenga las tablas `Leads` y `Errores`.
4. Verificar que la tabla `Leads` tenga los campos necesarios: nombre, correo electrónico, interés, mensaje, urgencia, prioridad IA, respuesta IA, aprobado, estado, estado de envío y error.
5. Verificar que la tabla `Errores` tenga los campos: fecha de error, lead relacionado, módulo, tipo de error, detalle del error y estado del error.
6. Ejecutar una prueba con un lead en estado `Pendiente`.
7. Esperar el aviso humano enviado por Gmail 21.
8. Durante la pausa de 5 minutos, aprobar el lead en Airtable.
9. Confirmar que el Router 24 derive el flujo por la rama aprobada.
10. Verificar que Gmail 25 envíe la respuesta final.
11. Verificar que Airtable 27 actualice el estado de envío a `Enviado`.

## Estado final

La entrega queda actualizada con:

- Repositorio organizado.
- Blueprint final actualizado.
- Interfaz KPI creada en Airtable.
- Evidencia visual de la interfaz KPI.
- Flujo HITL con aviso humano, espera, relectura y decisión posterior.
- Rama standby para leads sin aprobación.
- Evidencias nuevas del flujo HITL final.
- Manejo de errores actualizado para el nuevo envío final.
- Guía de revisión y rúbrica consolidada.
