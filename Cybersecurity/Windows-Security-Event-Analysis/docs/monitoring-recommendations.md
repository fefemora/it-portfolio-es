# Recomendaciones para mejorar el monitoreo — Propuestas

[Proyecto](../README.md) · [Resumen ejecutivo](executive-summary.md) · [Hallazgos técnicos](event-analysis.md) · [Guía del laboratorio](lab-guide.md)

**Estado: propuestas, no ejecutadas en esta investigación.** Ninguna de las acciones siguientes fue implementada ni validada como parte de este ejercicio de solo lectura. Las prioridades se refieren a **una estación de trabajo de laboratorio**, no a una evaluación de riesgos organizacionales ni de cumplimiento.

| Prioridad | Propuesta | Evidencia que la motiva | Cómo validar si se aprueba |
|---|---|---|---|
| **1 — Alta** | Definir un objetivo de retención del registro Security y evaluar capacidad o reenvío de eventos | Máximo 20 MiB, circular; intervalo observado 2,03 días | Comparar propiedades y fechas de los registros más antiguo y reciente a lo largo del tiempo; verificar el reenvío si se implementa |
| **2 — Media** | Realizar un piloto de **Audit Process Creation** con cambio controlado | La subcategoría `Process Creation` informó `No Auditing` | Registrar `auditpol` antes/después; validar eventos nuevos 4688 de forma segura y vigilar el volumen |
| **3 — Media** | Establecer línea base de eventos 4624/4672, diferenciando servicios del sistema y cuentas atípicas | Muestras acotadas dominadas por LocalSystem; una correlación de Logon ID verificada | Comparar muestras limitadas y anonimizadas en distintas fechas y categorías de cuentas |
| **4 — Media** | Definir un proceso liviano de recopilación y anonimización | Evidencia de consola; consultas sin fechas de captura alineadas ni exportaciones firmadas | Registrar fecha, consulta, alcance, resultado anonimizado y limitaciones |
| **5 — Investigar solo si se justifica** | Buscar evidencia adicional de la discrepancia 4688 frente a la política actual | Cinco 4688 del 2026-11-16 14:30:43; posteriormente `No Auditing` y ningún 4719 coincidente | Revisar registros históricos autorizados de configuración/cambios y mantener el hallazgo abierto sin evidencia |

## Consideraciones operativas y de privacidad

- Un registro Security mayor o una recolección centralizada consume recursos de almacenamiento y red. Deben definirse **tamaño y plazo objetivo antes del cambio**.
- Auditar creación de procesos incrementa los eventos; los argumentos de línea de comandos pueden contener información sensible. Revisar privacidad y permisos antes de ejecutar pilotos.
- La auditoría de acceso a objetos requiere configuraciones adicionales y puede producir grandes volúmenes. No activar indiscriminadamente todas las subcategorías.
- Preferir herramientas integradas y consultas acotadas, dadas las limitaciones de recursos del equipo.
- Todo cambio requiere aprobación independiente y plan de validación antes/después. **Esta investigación no realizó cambios.**

## Próxima evaluación sugerida

Definir un objetivo de retención razonable y volver a medir fechas extremas del registro en diferentes días. Separadamente, decidir si la auditoría de procesos aporta valor suficiente frente al volumen y riesgos asociados. **No declarar un control implementado sin validarlo.**

**Aclaración histórica:** el proyecto relacionado [Windows Security Hardening](../../Windows-Security-Hardening/README.md) documenta una modificación **posterior** del tamaño máximo de Security de 20 a 128 MiB bajo CHG-001. Este documento conserva las propuestas y condiciones observadas **antes** de esa modificación.

## Referencias oficiales

- [Modo de eventos y retención](https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.eventing.reader.eventlogmode)
- [Audit Process Creation](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/audit-process-creation)
- [Windows Event Forwarding](https://learn.microsoft.com/en-us/windows/security/operating-system-security/device-management/use-windows-event-forwarding-to-assist-in-intrusion-detection)
- [Audit Special Logon](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/audit-special-logon)

*Las referencias respaldan las propuestas, no demuestran su implementación.*