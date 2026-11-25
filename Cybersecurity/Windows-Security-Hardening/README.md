# <img src="../../assets/icons/cybersecurity.svg" width="32" alt="Ciberseguridad"> Evaluación de seguridad y hardening de Windows

![Windows](https://img.shields.io/badge/Windows-Seguridad_de_endpoints-14283F?style=flat-square) ![PowerShell](https://img.shields.io/badge/PowerShell-L%C3%ADnea_base-14283F?style=flat-square) ![Estado](https://img.shields.io/badge/EVALUACI%C3%93N-30_pasos_documentados-059669?style=flat-square) ![Pendiente](https://img.shields.io/badge/MEJORAS-Pendientes-E5A000?style=flat-square)

[← Ciberseguridad](../README.md) · [Portafolio](../../README.md) · **[Guía de 30 pasos](docs/baseline-guide.md)** · [Matriz de evaluación](docs/security-assessment.md) · [Plan de cambios y validación](docs/validation-plan.md) · [Resumen técnico](RESUMEN-ES.md)

[![Flujo de trabajo del laboratorio de seguridad de Windows](diagrams/hardening-workflow.svg)](docs/baseline-guide.md)

## Resumen de la evaluación

Evalué la línea base de seguridad de una estación de trabajo Windows mediante **28 comprobaciones técnicas y dos etapas de análisis y cierre**. Revisé Windows Firewall, Microsoft Defender, auditoría, registros de PowerShell, BitLocker, TPM y seguridad de arranque.

La mayor parte del trabajo fue de solo lectura. Se autorizó un único cambio acotado, **CHG-001**, para aumentar el tamaño máximo del registro Security de **20 MiB a 128 MiB**. La nueva configuración quedó verificada, pero todavía **no se ha demostrado una retención de siete días**.

| Área | Qué muestra la evidencia |
|---|---|
| **Firewall** | Perfiles Domain, Private y Public habilitados; valores efectivos de entrada **Block** y salida **Allow** |
| **Microsoft Defender** | Antivirus, protección en tiempo real, Tamper Protection y NIS activos en los campos consultados |
| **Auditoría de seguridad** | Inicio de sesión configurado para aciertos y errores; creación de procesos en **Sin auditoría** |
| **Registro Security** | CHG-001 elevó el máximo de **20 a 128 MiB**; intervalo observado inmediato **1,93 → 1,96 días** |
| **BitLocker** | Volumen del sistema completamente descifrado, protección **Off** |
| **Arranque** | TPM preparado, UEFI y GPT observados; **Secure Boot deshabilitado** |
| **PowerShell** | Canal operativo activo; algunas claves de política ausentes en las rutas inspeccionadas; eventos 4104 observados |
| **Autenticación** | 235 registros 4624 y 233 registros 4672 en consultas independientes de 24 horas; los conteos no demuestran abuso |

## CHG-001 · Cambio controlado

**Motivo:** el registro Security circular tenía un límite original de 20 MiB, con una ventana relativamente corta para investigar eventos.

**Acción aprobada:** aumentar únicamente la capacidad máxima.

```powershell
wevtutil sl Security /ms:134217728
```

La comprobación posterior mostró `MaximumSizeInBytes=134217728`, `IsEnabled=True` y `LogMode=Circular`. El intervalo inmediato entre los registros más antiguo y reciente pasó de **1,93** a **1,96 días**. Esto confirma la configuración y el acceso inmediato al historial, **no** siete días de retención futura.

[Consultar evidencias antes/después y criterios de validación](docs/validation-plan.md)

## Hallazgos y medidas pendientes

| Área | Observación | Próxima medida propuesta |
|---|---|---|
| **Cifrado** | Protección BitLocker desactivada | Preparar respaldos y recuperación antes de autorizar el cifrado |
| **Secure Boot** | Deshabilitado pese a UEFI/GPT y TPM preparado | Verificar compatibilidad y recuperación antes de cambiar firmware |
| **Auditoría de procesos** | Creación de procesos sin auditoría | Considerar piloto acotado según privacidad y volumen de eventos |
| **Retención** | 128 MiB configurados; siete días sin validar | Volver a medir fechas extremas tras un intervalo suficiente |
| **PowerShell** | Telemetría parcial y cobertura efectiva desconocida | Revisar requisitos antes de habilitar registros adicionales |

Son **recomendaciones**, no cambios ejecutados. Durante esta evaluación no se confirmó ningún incidente de seguridad.

## Documentación y evidencias

| Documento | Contenido |
|---|---|
| **[Guía de laboratorio · 30 pasos](docs/baseline-guide.md)** | Comandos ejecutados, salidas reales e interpretación |
| **[Matriz de evaluación](docs/security-assessment.md)** | Estado de controles, hallazgos, límites y prioridades |
| **[Plan de cambios y validación](docs/validation-plan.md)** | Alcance de CHG-001, comprobaciones antes/después y trabajo pendiente |
| **[Investigación previa de eventos Windows](../Windows-Security-Event-Analysis/README.md)** | Análisis de eventos que motivó la evaluación posterior |

## Competencias aplicadas

**PowerShell · Evaluación de seguridad de Windows · Auditoría de eventos · Microsoft Defender · Protección de endpoints · Gestión de cambios · Validación**

Este proyecto documenta una **evaluación real basada en evidencias**. No significa que se haya aplicado una línea base empresarial completa ni que el equipo cumpla una certificación de seguridad. Las observaciones detalladas permanecen en los informes enlazados y los resultados no confirmados se señalan explícitamente.

[← Volver a Ciberseguridad](../README.md)
