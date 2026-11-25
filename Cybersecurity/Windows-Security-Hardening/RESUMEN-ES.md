# Evaluación de seguridad de Windows — Resumen técnico en español

[Portafolio](../../README.md) · [Proyecto y evidencia completa](README.md) · [Evaluación detallada](docs/security-assessment.md) · [Plan de validación](docs/validation-plan.md) · [Metodología](../../metodologia/README.md)

**Tipo:** Laboratorio real de evaluación de un equipo Windows, con consultas mayoritariamente de solo lectura y **un único cambio específicamente autorizado**.

**Estado:** Fase de evaluación y documentación completada (30 pasos); algunas medidas de endurecimiento y comprobaciones posteriores permanecen pendientes.

## Objetivo y método

Evaluar controles nativos de Windows —Defender, firewall, políticas de auditoría, PowerShell, BitLocker, TPM y Secure Boot— mediante evidencia real; separar lo observado de lo inferido, de lo recomendado y de lo implementado.

El trabajo siguió la secuencia **línea base → interpretación → evaluación del riesgo → autorización del cambio → validación → documentación**.

## Hallazgos principales

| Área | Resultado respaldado por evidencia | Limitación |
|---|---|---|
| Firewall de Windows | Perfiles habilitados; política efectiva consultada en ActiveStore con entrada Block y salida Allow | No se probaron reglas concretas ni tráfico real |
| Microsoft Defender | Antivirus y protección en tiempo real activos; indicadores de protección avanzada verificados | No se comprobó eficacia frente a amenazas reales |
| Auditoría de procesos | Subcategoría Process Creation: **Sin auditoría** | Se documentó como posible brecha de visibilidad; no se modificó |
| Registro Security | CHG-001 elevó capacidad de **20 a 128 MiB**; configuración verificada | Meta de **7 días** sin validación; observación inmediata de **1,96 días** |
| BitLocker C: | Volumen descifrado; protección Off | Su activación queda aplazada hasta preparar respaldos y recuperación |
| TPM / arranque | TPM preparado; UEFI y GPT verificados; Secure Boot False | No se cambiaron opciones de firmware |
| PowerShell | Canal Operational activo; historial observado de **5,05 días**; eventos 4104 presentes | No se puede concluir cobertura integral de logging |
| Module Logging | Rutas de política HKLM/HKCU consultadas sin clave; consulta de ID 4103 devolvió 0 | Cobertura efectiva no determinada |
| Transcription | Claves de política HKLM/HKCU ausentes en las rutas verificadas | No se investigaron archivos de transcripción |

## Eventos de autenticación y privilegios

La auditoría **Logon** se encontraba configurada para aciertos y errores. En una consulta de las últimas 24 horas se recuperaron **235 eventos 4624** y no apareció un grupo de eventos 4625.

La distribución de los 235 eventos exitosos fue:

- **229** de tipo **5 — Service**.
- **4** de tipo **11 — CachedInteractive**.
- **2** de tipo **7 — Unlock**.

Una consulta diferente recuperó **233 eventos 4672** relacionados con la asignación de privilegios especiales a nuevas sesiones. Son conteos de registros, **no cantidades de personas, incidentes ni evidencia de intrusión**.

## Cambio ejecutado: CHG-001

**Única modificación aprobada:** aumentar la capacidad máxima del registro Security de **20 MiB a 128 MiB**.

**Verificación:** el valor posterior se observó en el sistema. **No se validó todavía** que el registro mantenga siete días de eventos; aumentar la capacidad no demuestra por sí solo un período de retención.

No se activaron BitLocker, Secure Boot ni auditoría de creación de procesos durante esta fase.

## Recomendaciones priorizadas

1. Preparar copia de seguridad fiable, mecanismo de recuperación y custodia de claves **antes** de evaluar cambios de cifrado o firmware.
2. Volver a medir la retención real del registro Security tras un período suficiente.
3. Evaluar una futura auditoría de creación de procesos con autorización, análisis de privacidad, rendimiento y reversión.
4. Determinar necesidades y alcance de registros de PowerShell sin asumir que la ausencia de una política significa ausencia total de telemetría.
5. Revisar tendencias de autenticación con datos agregados y realizar investigaciones más profundas únicamente con alcance y autorizaciones adecuados.

## Alcance del cierre

**Se completó una evaluación de línea base, no una certificación de seguridad ni un endurecimiento integral del equipo.** No se confirmó incidente alguno. Se preservó la distinción entre evidencias observadas, cambios autorizados y controles pendientes.
