# Guía de revisión, rúbrica y ejecución

Este archivo funciona como guía central de revisión para la entrega final. Su objetivo es facilitar la lectura del repositorio, indicar dónde está cada entregable y vincular cada criterio técnico con una evidencia verificable.

La entrega corresponde al proyecto **Ecosistema IA Leads**, un sistema de automatización para clasificar leads con IA, generar una respuesta sugerida, solicitar intervención humana y enviar la respuesta final solo después de la aprobación.

## 1. Estructura del repositorio

El repositorio está organizado en tres carpetas principales:

- `documentacion/`
- `blueprint/`
- `evidencias/`

Archivos principales:

- `documentacion/documentacion_entrega_final-Omar Rodriguez.pdf`
- `documentacion/matriz_costos_modelos.pdf`
- `documentacion/guia_revision_rubrica_ejecucion.md`
- `blueprint/make_blueprint_final.json`
- `evidencias/`

La carpeta `evidencias` contiene capturas del escenario, pruebas exitosas, prueba de error, interfaz KPI y prueba final HITL.

## 2. Arquitectura general del ecosistema

El ecosistema utiliza:

- **Airtable** como base de datos, dashboard, punto de aprobación humana y registro de errores.
- **Make** como orquestador principal del escenario.
- **OpenAI** para clasificar la prioridad comercial del lead y generar una respuesta sugerida.
- **Gmail** como acción de aviso humano y como canal de envío final.
- **HITL** como instancia de intervención humana antes del envío final.

Flujo general:

Airtable → Router → OpenAI → Airtable → Gmail aviso humano → Sleep → Airtable Get a Record → Router HITL

Desde el Router HITL:

- Si el lead fue aprobado: Gmail envío final → Airtable enviado.
- Si el lead no fue aprobado: Airtable standby.

Evidencias relacionadas:

- `blueprint/make_blueprint_final.json`
- `evidencias/10_make_flujo_hitl_aviso_espera_revision.png`
- `evidencias/13_make_ejecucion_hitl_aprobado_exitosa.png`

## 3. Funcionamiento del escenario en Make

El escenario comienza con **Airtable Watch Records** sobre la tabla `Leads`.

El disparador utiliza el campo **Última modificación**, lo que permite detectar registros nuevos y cambios posteriores en los leads.

Funcionamiento paso a paso:

1. Airtable detecta un lead en estado `Pendiente`.
2. Router 12 permite avanzar solo leads pendientes y sin error.
3. OpenAI analiza el lead, clasifica la prioridad comercial y genera una respuesta sugerida.
4. Airtable 4 actualiza el lead con la respuesta IA y el estado `Procesado por IA`.
5. Gmail 21 envía un aviso al revisor humano.
6. Tools 22 pausa el flujo durante 5 minutos.
7. Airtable 23 vuelve a leer el mismo lead.
8. Router 24 evalúa si hubo intervención humana.
9. Si el lead fue aprobado, Gmail 25 envía la respuesta final al lead.
10. Airtable 27 marca el lead como `Enviado`.
11. Si el lead no fue aprobado, Airtable 26 lo deja en estado `Esperando aprobación humana`.

Evidencias relacionadas:

- `blueprint/make_blueprint_final.json`
- `evidencias/10_make_flujo_hitl_aviso_espera_revision.png`
- `evidencias/13_make_ejecucion_hitl_aprobado_exitosa.png`

## 4. HITL implementado

El HITL fue ajustado para incorporar una intervención humana explícita antes del envío final.

El flujo no envía la respuesta al lead inmediatamente después de la IA. Primero envía un aviso al humano revisor mediante Gmail, espera 5 minutos, vuelve a leer el registro de Airtable y recién ahí decide si continúa o queda en espera.

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

Si esas condiciones se cumplen, el flujo continúa con el envío final por Gmail.

Si esas condiciones no se cumplen, el flujo pasa a la rama standby y actualiza el lead como:

- `Estado = Esperando aprobación humana`
- `Estado de envio = No enviado`

Evidencias relacionadas:

- `evidencias/10_make_flujo_hitl_aviso_espera_revision.png`
- `evidencias/12_email_aviso_humano_hitl.png`
- `evidencias/13_make_ejecucion_hitl_aprobado_exitosa.png`
- `evidencias/14_airtable_hitl_aviso_visible_enviado.png`
- `evidencias/15_email_respuesta_final_hitl.png`
- `blueprint/make_blueprint_final.json`

## 5. Interfaz KPI

Se creó una interfaz en Airtable llamada **Dashboard KPI Leads** para mostrar indicadores del ecosistema.

La interfaz muestra información de control como:

- Total de leads.
- Distribución por prioridad.
- Datos visibles del estado general del sistema.

Evidencia relacionada:

- `evidencias/09_airtable_interfaz_kpi_dashboard.png`

Nota sobre el enlace público: Airtable no permitió generar un enlace público web de la interfaz desde el plan utilizado. Por ese motivo se deja evidencia visual directa de la interfaz KPI implementada.

El link al dashboard/base de Airtable se mantiene documentado en:

- `evidencias/link_airtable_dashboard_control.txt`

## 6. Manejo de errores

El escenario incluye manejo de errores en módulos críticos.

Módulos con Error Handler:

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

En el caso de OpenAI, se configuró retry automático porque puede tratarse de errores temporales de API.

En el caso de Gmail 25, se dejó retry manual para evitar reintentos innecesarios ante errores como correo inválido o dato mal cargado.

Evidencias relacionadas:

- `evidencias/make_error_handlers_openai_gmail.png`
- `evidencias/06_make_prueba_error_gmail.png`
- `evidencias/07_airtable_registro_error_gmail.png`
- `blueprint/make_blueprint_final.json`

## 7. Pruebas realizadas

### Prueba de flujo exitoso anterior

Se validó que el sistema pudiera procesar un lead, generar respuesta IA, aprobar manualmente y enviar un correo final.

Evidencias:

- `evidencias/01_make_escenario_router_dos_rutas.png`
- `evidencias/02_make_ejecucion_gmail_exitosa.png`
- `evidencias/03_airtable_dashboard_control_estado_enviado.png`
- `evidencias/04_email_recibido_gmail_hitl.png`

### Prueba de error

Se utilizó un correo inválido para verificar que el Error Handler registrara el fallo en Airtable.

Resultado esperado:

- Gmail falla por correo inválido.
- Make detecta el error.
- La ejecución queda como incomplete execution.
- El error se registra automáticamente en Airtable.

Evidencias:

- `evidencias/06_make_prueba_error_gmail.png`
- `evidencias/07_airtable_registro_error_gmail.png`

### Prueba HITL final

Se probó el flujo final con aviso humano, espera, aprobación, envío final y actualización en Airtable.

Resultado obtenido:

- El lead fue procesado por IA.
- El humano recibió un aviso por Gmail.
- El flujo esperó 5 minutos.
- Airtable volvió a leer el lead.
- El humano aprobó el registro.
- Gmail envió la respuesta final.
- Airtable marcó el estado de envío como `Enviado`.

Evidencias:

- `evidencias/12_email_aviso_humano_hitl.png`
- `evidencias/13_make_ejecucion_hitl_aprobado_exitosa.png`
- `evidencias/14_airtable_hitl_aviso_visible_enviado.png`
- `evidencias/15_email_respuesta_final_hitl.png`

## 8. Matriz de costos y modelos

La matriz de decisión de costos y modelos se entrega como PDF independiente.

Archivo:

- `documentacion/matriz_costos_modelos.pdf`

El documento compara distintos modelos según costo aproximado, uso recomendado y conveniencia operativa para este ecosistema.

Modelos contemplados:

- GPT-4.1
- GPT-4.1 mini
- GPT-4o mini
- Claude Haiku 4.5
- Batch API

## 9. Blueprint actualizado

El blueprint actualizado se encuentra en:

- `blueprint/make_blueprint_final.json`

Este archivo contiene la versión final del escenario con:

- Trigger de Airtable por `Última modificación`.
- Procesamiento con OpenAI.
- Aviso humano por Gmail.
- Pausa de espera.
- Relectura del lead.
- Router de aprobación humana.
- Rama aprobada.
- Rama standby.
- Envío final al lead.
- Actualización de estado enviado.
- Manejo de errores en OpenAI.
- Manejo de errores en Gmail final.

## 10. Instrucciones para revisar la entrega

Para revisar el proyecto:

1. Abrir el README principal del repositorio.
2. Revisar esta guía en `documentacion/guia_revision_rubrica_ejecucion.md`.
3. Revisar la documentación principal en `documentacion/documentacion_entrega_final-Omar Rodriguez.pdf`.
4. Revisar la matriz de costos en `documentacion/matriz_costos_modelos.pdf`.
5. Revisar el blueprint actualizado en `blueprint/make_blueprint_final.json`.
6. Verificar las capturas dentro de `evidencias/`.
7. Abrir el link de Airtable desde `evidencias/link_airtable_dashboard_control.txt`.
8. Revisar la evidencia KPI en `evidencias/09_airtable_interfaz_kpi_dashboard.png`.
9. Revisar la evidencia HITL final en las capturas 10, 12, 13, 14 y 15.

## 11. Instrucciones para ejecutar el escenario

Para ejecutar el escenario en Make:

1. Importar el archivo `blueprint/make_blueprint_final.json`.
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

## 12. Mapa de rúbrica y evidencias

| Criterio | Evidencia verificable |
|---|---|
| Repositorio organizado | Carpetas `documentacion`, `blueprint` y `evidencias` |
| Documentación principal | `documentacion/documentacion_entrega_final-Omar Rodriguez.pdf` |
| Matriz de costos y modelos | `documentacion/matriz_costos_modelos.pdf` |
| Blueprint actualizado | `blueprint/make_blueprint_final.json` |
| Ecosistema de automatización funcional | `evidencias/10_make_flujo_hitl_aviso_espera_revision.png` |
| Uso de IA para clasificación y respuesta | `blueprint/make_blueprint_final.json` |
| Intervención humana HITL | `evidencias/12_email_aviso_humano_hitl.png` y `evidencias/13_make_ejecucion_hitl_aprobado_exitosa.png` |
| Espera antes de continuar | `evidencias/10_make_flujo_hitl_aviso_espera_revision.png` |
| Relectura del lead después de la espera | `blueprint/make_blueprint_final.json` y `evidencias/13_make_ejecucion_hitl_aprobado_exitosa.png` |
| Envío final solo aprobado por humano | `evidencias/13_make_ejecucion_hitl_aprobado_exitosa.png`, `evidencias/14_airtable_hitl_aviso_visible_enviado.png` y `evidencias/15_email_respuesta_final_hitl.png` |
| Rama standby si no hay aprobación | `blueprint/make_blueprint_final.json` y `evidencias/10_make_flujo_hitl_aviso_espera_revision.png` |
| Interfaz KPI | `evidencias/09_airtable_interfaz_kpi_dashboard.png` |
| Link a Airtable | `evidencias/link_airtable_dashboard_control.txt` |
| Manejo de errores en Make | `evidencias/make_error_handlers_openai_gmail.png` |
| Registro automático de errores en Airtable | `evidencias/07_airtable_registro_error_gmail.png` |
| Prueba de error verificable | `evidencias/06_make_prueba_error_gmail.png` |

## 13. Estado final de la entrega

La entrega queda actualizada con los puntos observados por el profesor:

- Interfaz KPI creada en Airtable.
- Evidencia visual de la interfaz KPI.
- Flujo HITL ajustado con aviso humano, espera, relectura y decisión posterior.
- Rama standby para leads sin aprobación.
- Blueprint actualizado.
- Evidencias nuevas del flujo HITL final.
- Manejo de errores actualizado para el nuevo envío final.
- Guía de revisión y rúbrica consolidada en este archivo.
