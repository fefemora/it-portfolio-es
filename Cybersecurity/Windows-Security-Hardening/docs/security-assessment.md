# Registro de evaluación — Línea base de seguridad de Windows

[Proyecto](../README.md) · [Guía de comprobaciones](baseline-guide.md) · [Plan de validación](validation-plan.md)

**Fase:** evaluación completada, pasos 1–30 documentados.  
**Control de cambios:** **CHG-001 fue el único cambio implementado y su configuración se verificó**. La retención de siete días y las mejoras restantes siguen pendientes.

## Resumen de evaluación de controles

| Dominio | Objetivo | Evidencia verificada | Riesgo / límite | Estado |
|---|---|---|---|---|
| Firewall | Limitar tráfico no solicitado | Tres perfiles habilitados. Consulta inicial: acciones `NotConfigured`. `ActiveStore`: entrada `Block`, salida `Allow` | No se revisaron todas las reglas ni tráfico efectivo | **Configuración seleccionada verificada** |
| Defender Antivirus | Protección de endpoints | Servicio, antivirus, protección real, Tamper Protection y NIS activos; firmas con antigüedad 0 días (actualizadas el 2026-11-17 a las 09:41:14, hora de consola) | No se probaron firmas, exclusiones ni eficacia frente a malware | **Indicadores verificados** |
| Defender avanzado | PUA, nube y bloqueo temprano | `MAPSReporting=2`, `SubmitSamplesConsent=1`, `PUAProtection=1`, `DisableBlockAtFirstSeen=False` | No se verificó toda la cadena de operación cloud | **Preferencias verificadas** |
| Registro Security | Evidencia para investigaciones | CHG-001: máximo de **20 a 128 MiB**, `Circular`; 33.989 registros después | Intervalo inmediato **1,96 días**; siete días no comprobados | **Configuración verificada; retención pendiente** |
| Auditoría de procesos | Visibilidad de creación de procesos | `Process Creation: No Auditing` | Puede limitar eventos nuevos 4688 | **Brecha potencial de visibilidad; cambio no aprobado** |
| Cifrado del sistema | Proteger información en reposo | C: `FullyDecrypted`, `ProtectionStatus=Off`, `0%` | Exposición ante pérdida física; depende del contexto | **Estado verificado; cifrado diferido** |
| TPM | Capacidad de hardware | `Present/Ready/Enabled/Activated=True` | No acredita disponibilidad de respaldos o claves | **Estado verificado** |
| Secure Boot | Integridad de inicio | `Confirm-SecureBootUEFI=False`, firmware UEFI | Arranque seguro deshabilitado | **Verificado; cambio diferido** |
| Disco del sistema | Compatibilidad de plataforma | Disco 0 GPT, `IsBoot=True` y `IsSystem=True` del objeto disco | No prueba seguridad del firmware ni particiones individuales | **Verificado** |
| Script Block Logging | Visibilidad de bloques de scripts | Claves HKLM/HKCU ausentes en rutas consultadas; 4104 recuperados: 438 y luego 439 advertencias | Cobertura real y origen de eventos no determinados | **Metadatos verificados, eficacia desconocida** |
| Module Logging | Telemetría de módulos | Claves HKLM/HKCU ausentes; consulta 4103 devolvió 0 | Errores no terminantes suprimidos y retención circular | **No se recuperaron 4103; cobertura desconocida** |
| PowerShell Transcription | Registro de sesiones | Claves HKLM/HKCU ausentes en rutas examinadas | Transcripciones de sesión y otras fuentes no revisadas | **Políticas consultadas; cobertura desconocida** |
| PowerShell Operational | Mantener eventos PowerShell | Activo, 808 registros en 11 identificadores, 15 MiB, Circular | Intervalo **5,05 días** no constituye SLA | **Canal e intervalo verificados** |
| Auditoría de acceso | Registrar autenticación | `Logon = Aciertos y errores` | Política configurada no garantiza retención completa | **Verificada** |
| Autenticación 4624/4625 | Evaluar actividad de acceso | 235 registros 4624; sin grupo 4625 en 24 horas | Son eventos, no usuarios únicos | **Agregados verificados** |
| Tipos de sesión 4624 | Diferenciar servicios y personas | Tipo 5: 229; tipo 11: 4; tipo 7: 2 | Predominio de servicios no garantiza benignidad | **Distribución verificada** |
| Privilegios especiales 4672 | Identificar asignación de privilegios | 233 registros de 24 horas | No es prueba de abuso o de 233 administradores | **Conteo verificado** |
| Identidad y acceso local | Evaluar exposición de roles locales | Sin medición | Requiere consulta acotada y anonimizada | **No evaluado** |
| Estado de actualizaciones | Identificar actualizaciones pendientes | Sin medición | Requiere una comprobación de solo lectura | **No evaluado** |

## Evidencia del cambio CHG-001

La única modificación registrada y autorizada en el laboratorio fue:

```powershell
wevtutil sl Security /ms:134217728
```

**Antes:** `MaximumSizeInBytes=20971520` (20 MiB), `RecordCount=33959`, `Circular`.  
**Después:** `MaximumSizeInBytes=134217728` (128 MiB), `IsEnabled=True`, `RecordCount=33989`, `Circular`.

Se observó una diferencia entre las fechas extremas del registro de **1,93 días antes** y **1,96 días inmediatamente después**, pero **no se validó** el objetivo de retener siete días. Tampoco se concluyó que exactamente treinta eventos nuevos explicaran la diferencia de conteos: las mediciones son independientes y la retención es circular.

[Ver salidas exactas y el procedimiento de validación](baseline-guide.md#chg-001--ampliación-autorizada-del-registro-security).

## Priorización de hallazgos

| Prioridad de planificación | Hallazgo | Evaluación | Recomendación (no implementada) |
|---|---|---|---|
| Alta atención; recuperación previa | C: sin cifrar, Secure Boot deshabilitado, TPM preparado y UEFI/GPT | Posible exposición de datos y cadena de arranque | Asegurar respaldo verificado, medios de recuperación y custodia de claves, después aprobar cambios separados |
| Media; retención | Límite Security de 128 MiB verificado, intervalo inmediato 1,96 días | La cobertura de siete días no está demostrada | Medir extremos nuevamente tras suficiente tiempo y ajustar según volumen |
| Media; detección | Auditoría de creación de procesos deshabilitada | Posible falta de telemetría 4688 | Evaluar piloto y restricciones de privacidad, costo operacional y reversión |
| Evaluar telemetría | Claves de políticas PowerShell ausentes pero hay 4104 | No se conoce la cobertura efectiva | Revisar alcance de políticas y casos de uso antes de habilitar registros adicionales |
| Observar tendencias | 235 eventos 4624, 233 eventos 4672, 229 sesiones tipo servicio | Conteos compatibles con actividad de servicios, sin prueba de benignidad o malicia | Analizar patrones agregados; correlacionar solo con motivo y autorización |
| Mantener | Indicadores activos de Defender y firewall | Estado positivo en campos concretos | Revisión periódica de la configuración |

**La prioridad se utiliza para ordenar trabajos, no corresponde a CVSS ni a una certificación de cumplimiento.**

## Matriz de auditoría de PowerShell

| Función | HKLM | HKCU | Evidencia |
|---|---|---|---|
| Script Block Logging | Clave ausente | Clave ausente | 438 eventos 4104 inicialmente; 439 registros de advertencia después |
| Module Logging | Clave ausente | Clave ausente | 0 eventos 4103 recuperados |
| Transcription | Clave ausente | Clave ausente | No se examinaron archivos de transcripción |
| Operational | No aplica a estas claves | No aplica a estas claves | Canal activo, 15 MiB, circular, 808 eventos, 5,05 días observados |

Las dos rutas de política consultadas en cada función **no prueban todas las configuraciones efectivas**. Los eventos 4104 no permiten inferir que exista registro completo de todos los scripts ni actividad maliciosa. El conteo 4103 se realizó con supresión de errores no terminantes.

## Interpretación de la autenticación

La política de Logon estaba configurada para auditar **aciertos y errores**. En una consulta de 24 horas se recuperaron **235 eventos 4624** y no apareció grupo para 4625. En otra medición se clasificaron los 235 eventos 4624: **229 tipo 5 (servicio, 97,4 %), 4 tipo 11 (inicio interactivo en caché) y 2 tipo 7 (desbloqueo)**.

Otra consulta recuperó **233 eventos 4672**. Esta coincidencia aproximada de volumen **no equivale** a vincular uno a uno ambas clases. Eventos 4672 indican **asignación** de privilegios a una sesión, no ejercicio de privilegios ni acceso malicioso.

## Reglas de clasificación

- **Verificado:** existe un comando ejecutado y un resultado real aportado para el control.
- **Evidencia anterior:** medición del proyecto Blue Team relacionado; no debe reclasificarse como dato nuevo.
- **Propuesto:** posible mejora sin autorización ni implementación.
- **Desconocido:** no probado; no equivale a vulnerable o ausente.
- **No aplicable:** requiere justificar la plataforma y el entorno.

## Criterios de evaluación

Los controles se evalúan según finalidad, rol del dispositivo, precedencia de políticas, privacidad, compatibilidad y costos operacionales. Una desviación respecto de una línea base empresarial **no implica automáticamente una vulnerabilidad**.

Los registros se consultaron de forma acotada y anonimizada. No se divulgaron cuentas, tokens, IP, datos de clientes ni secretos BitLocker. No se ejecutaron simulaciones de ataque ni se confirmó ningún incidente.

## Referencias y documentos relacionados

- [Guía completa de 30 pasos y salidas exactas](baseline-guide.md)
- [Plan de validación y CHG-001](validation-plan.md)
- [Caso anterior de análisis de eventos](../../Windows-Security-Event-Analysis/README.md)
- [Windows Security Baselines — Microsoft Learn](https://learn.microsoft.com/en-us/windows/security/operating-system-security/device-management/windows-security-configuration-framework/windows-security-baselines)
- [auditpol /get](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/auditpol-get)

*La evaluación se basa en evidencias limitadas a las comprobaciones ejecutadas; no se declara cumplimiento organizacional.*
