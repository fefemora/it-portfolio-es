# Plantilla de informe de incidente

> **Plantilla del portafolio:** formato estandarizado para escenarios de soporte TI. Los datos de ejemplo deben ser ficticios o estar anonimizados.

## 1. Información general

| Campo | Valor o descripción |
|---|---|
| ID del incidente | INC-XXX |
| Título | Descripción breve |
| Fecha de reporte | AAAA-MM-DD |
| Reportado por | Usuario o departamento ficticio |
| Nivel de soporte | L1 / L2 / L3 |
| Categoría | Hardware / Software / Redes / Identidades / Seguridad |
| Servicio afectado | Servicio o aplicación |
| Impacto | Bajo / Medio / Alto |
| Urgencia | Baja / Media / Alta |
| Prioridad | P1 / P2 / P3 / P4 |
| Estado | Abierto / En progreso / Pendiente / Resuelto / Cerrado |

## 2. Descripción del incidente

**Problema reportado:** describir síntomas proporcionados por el usuario.

**Comportamiento esperado:** explicar el funcionamiento normal del servicio.

**Comportamiento observado:** registrar el fallo real o informado.

## 3. Impacto en el negocio

Registrar cantidad de usuarios afectados, servicios implicados, consecuencias operativas, alternativas temporales y posibles implicancias de seguridad.

## 4. Prioridad y SLA

**Impacto:** [Bajo / Medio / Alto]  
**Urgencia:** [Baja / Media / Alta]  
**Prioridad asignada:** [P1 / P2 / P3 / P4]

**Justificación:** explicar por qué se eligió esa prioridad.

**SLA aplicable:** documentar objetivos de respuesta y resolución acordados, solo si están disponibles. **No inventar compromisos contractuales.**

## 5. Diagnóstico inicial

| Paso | Acción de diagnóstico | Resultado esperado | Hallazgo real |
|---|---|---|---|
| 1 | Verificación inicial | Comportamiento esperado | No probado |
| 2 | Comprobaciones básicas | Comportamiento esperado | No probado |
| 3 | Diagnóstico avanzado | Comportamiento esperado | No probado |

## 6. Análisis de causa raíz

**Hipótesis inicial:** describir la causa sospechada.

**Evidencia de respaldo:** registrar eventos, errores y resultados relevantes.

**Causa raíz confirmada:** completar solo cuando exista evidencia suficiente; en caso contrario, marcar **No confirmada**.

## 7. Procedimiento de resolución

Registrar cronológicamente las acciones correctivas propuestas o realizadas. Identificar cada una como:

- Propuesta.
- Simulada.
- Ejecutada y verificada.

## 8. Escalamiento

**¿Requiere escalamiento?:** Sí / No  
**Equipo:** L2 / L3 / Redes / Infraestructura / Seguridad / Proveedor

**Motivo:** explicar si faltan permisos, autorización o conocimientos especializados.

## 9. Validación y pruebas

| Comprobación | Resultado esperado | Resultado real |
|---|---|---|
| Disponibilidad del servicio | Servicio recuperado | No probado |
| Funcionalidad del usuario | Operación normal | No probado |
| Controles de seguridad | Permanecen habilitados | No probado |
| Confirmación del usuario | Usuario confirma recuperación | No obtenida |

## 10. Comunicación con el usuario

**Aviso inicial:** acuse de recibo y pasos siguientes.

**Actualización:** estado de investigación y próximos pasos.

**Comunicación de cierre:** solución y confirmación, cuando corresponda.

## 11. Cierre del incidente

**Estado final:** [Abierto / Resuelto / Cerrado / Simulado]  
**Resumen de resolución:** describir el resultado.
**Lecciones aprendidas:** mejoras al diagnóstico, monitoreo o documentación.
**Prevención:** recomendaciones para disminuir recurrencias.

## 12. Evidencia y documentación

Agregar, cuando exista: capturas de pantalla, eventos, salidas de comandos, informes técnicos y documentación de fabricante.

Toda información sensible debe anonimizarse antes de publicarse.

---

**Advertencia:** plantilla educativa y de portafolio. Los resultados simulados nunca deben presentarse como hallazgos productivos verificados.