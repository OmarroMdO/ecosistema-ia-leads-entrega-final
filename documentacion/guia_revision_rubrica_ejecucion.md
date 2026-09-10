# Guía de revisión, rúbrica y ejecución

Este archivo se agrega para facilitar la revisión de la entrega final y vincular cada criterio solicitado con su evidencia correspondiente dentro del repositorio.

## 1. Ubicación de archivos principales

| Elemento solicitado | Archivo o carpeta |
|---|---|
| Documentación principal del ecosistema | `documentacion/documentacion_entrega_final-Omar Rodriguez.pdf` |
| Matriz de decisión de costos y modelos | `documentacion/matriz_costos_modelos.pdf` |
| Blueprint exportado desde Make | `blueprint/make_blueprint_final.json` |
| Evidencias visuales | `evidencias/` |
| Link al dashboard de Airtable | `evidencias/link_airtable_dashboard_control.txt` |

## 2. Arquitectura del ecosistema

El ecosistema está compuesto por Airtable, Make, OpenAI y Gmail.

Airtable funciona como base de datos, dashboard, punto de revisión humana y registro de errores.

Make funciona como orquestador principal del escenario.

OpenAI se utiliza para clasificar la prioridad comercial del lead y generar una respuesta sugerida.

Gmail se utiliza como canal de envío únicamente después de la aprobación humana.

Evidencias relacionadas:

- `documentacion/documentacion_entrega_final-Omar Rodriguez.pdf`
- `blueprint/make_blueprint_final.json`
- `evidencias/01_make_escenario_router_dos_rutas.png`

## 3. Funcionamiento del escenario

El escenario de Make comienza con Airtable Watch Records sobre la tabla Leads.

El disparador utiliza el campo “Última modificación”, lo que permite detectar nuevos registros y cambios posteriores en los leads.

Luego, un Router separa el flujo en dos rutas:

1. Procesamiento del lead con IA.
2. Envío por Gmail solo después de aprobación humana.

Evidencias relacionadas:

- `blueprint/make_blueprint_final.json`
- `evidencias/01_make_escenario_router_dos_rutas.png`

## 4. Revisión humana HITL

El envío por Gmail no ocurre automáticamente después de la respuesta de IA.

Primero, el lead debe ser aprobado por una persona en Airtable.

Condiciones para enviar:

- Aprobado = marcado
- Estado = Aprobado por humano
- Estado de envío = No enviado
- Error distinto de true

Evidencias relacionadas:

- `evidencias/02_make_ejecucion_gmail_exitosa.png`
- `evidencias/03_airtable_dashboard_control_estado_enviado.png`
- `evidencias/04_email_recibido_gmail_hitl.png`
- `blueprint/make_blueprint_final.json`

## 5. Manejo de errores

La versión corregida incluye Error Handlers en los módulos críticos:

- OpenAI
- Gmail

Cuando ocurre un error, Make crea automáticamente un registro en la tabla Errores de Airtable.

Evidencias relacionadas:

- `evidencias/make_error_handlers_openai_gmail.png`
- `evidencias/06_make_prueba_error_gmail.png`
- `evidencias/07_airtable_registro_error_gmail.png`
- `blueprint/make_blueprint_final.json`

## 6. Prueba de error verificable

Para validar el camino de error se usó un correo inválido en un lead de prueba.

Resultado esperado:

- Gmail falla por correo inválido.
- Make detecta el error.
- La ejecución queda como incomplete execution.
- El Error Handler crea un registro en la tabla Errores de Airtable.
- El error queda con módulo Gmail, tipo Error de envío y estado Abierto.

Evidencias relacionadas:

- `evidencias/06_make_prueba_error_gmail.png`
- `evidencias/07_airtable_registro_error_gmail.png`

## 7. Matriz de costos y modelos

La matriz de costos se entrega como archivo PDF independiente.

Archivo:

- `documentacion/matriz_costos_modelos.pdf`

El documento compara modelos según costo, uso recomendado y conveniencia operativa para el caso de uso.

Modelos contemplados:

- GPT-4.1
- GPT-4.1 mini
- GPT-4o mini
- Claude Haiku 4.5
- Batch API

## 8. Instrucciones para revisar la entrega

Para revisar la entrega:

1. Abrir el README principal del repositorio.
2. Revisar la documentación principal en `documentacion/documentacion_entrega_final-Omar Rodriguez.pdf`.
3. Revisar la matriz de costos en `documentacion/matriz_costos_modelos.pdf`.
4. Revisar el blueprint en `blueprint/make_blueprint_final.json`.
5. Verificar las capturas dentro de la carpeta `evidencias`.
6. Abrir el link al dashboard de Airtable desde `evidencias/link_airtable_dashboard_control.txt`.

## 9. Instrucciones para ejecutar el escenario en Make

Para ejecutar el escenario:

1. Importar el archivo `blueprint/make_blueprint_final.json` en Make.
2. Reconectar las conexiones propias de Airtable, OpenAI y Gmail.
3. Verificar que la base de Airtable tenga las tablas Leads y Errores.
4. Verificar que la tabla Leads tenga los campos de estado, aprobación, respuesta IA, error y estado de envío.
5. Verificar que la tabla Errores tenga los campos fecha de error, lead relacionado, módulo, tipo de error, detalle del error y estado del error.
6. Ejecutar una prueba con un lead en estado Pendiente.
7. Revisar la respuesta generada por IA en Airtable.
8. Aprobar manualmente el lead en Airtable.
9. Ejecutar la ruta de envío por Gmail.
10. Probar un correo inválido para validar el Error Handler de Gmail.

## 10. Mapa de rúbrica y evidencias

| Criterio | Evidencia verificable |
|---|---|
| Ecosistema de automatización funcional | `evidencias/01_make_escenario_router_dos_rutas.png` |
| Uso de IA para clasificación y respuesta | `blueprint/make_blueprint_final.json` y documentación principal |
| Revisión humana antes del envío | `evidencias/02_make_ejecucion_gmail_exitosa.png`, `evidencias/03_airtable_dashboard_control_estado_enviado.png`, `evidencias/04_email_recibido_gmail_hitl.png` |
| Base de datos y dashboard en Airtable | `evidencias/03_airtable_dashboard_control_estado_enviado.png` y link de Airtable |
| Manejo de errores en Make | `evidencias/make_error_handlers_openai_gmail.png` |
| Registro automático de errores en Airtable | `evidencias/07_airtable_registro_error_gmail.png` |
| Prueba de error verificable | `evidencias/06_make_prueba_error_gmail.png` |
| Blueprint exportado | `blueprint/make_blueprint_final.json` |
| Matriz de costos y modelos | `documentacion/matriz_costos_modelos.pdf` |
| Repositorio organizado | carpetas `documentacion`, `blueprint` y `evidencias` |

## 11. Estado de la entrega

La entrega contiene los archivos, configuración y documentación solicitados en la consigna, organizados en carpetas separadas y con evidencias verificables para los criterios principales de revisión.
