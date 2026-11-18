# Análisis de eventos de seguridad de Windows — Hallazgos iniciales

[Proyecto](../README.md) · [Resumen ejecutivo](executive-summary.md) · [Guía reproducible](lab-guide.md) · [Recomendaciones](monitoring-recommendations.md)

**Estado:** triaje inicial; ningún incidente confirmado.  
**Fuente:** salidas reales de PowerShell compartidas por el usuario desde el registro Security local.  
**Ventana:** siete días anteriores a la ejecución, sin hora exacta de captura para la primera consulta.

## Configuración verificada del registro Security

| Propiedad | Valor observado |
|---|---|
| LogName | Security |
| IsEnabled | True |
| RecordCount | 33.898 |
| MaximumSizeInBytes | 20.971.520 bytes (20 MiB) |
| LogMode | Circular |

El canal estaba habilitado. En modo circular, los eventos antiguos pueden sobrescribirse; consultar siete días no garantiza siete días históricos completos.

Posteriormente se observaron las fechas **2026-11-15 20:02:39** (más antiguo) y **2026-11-17 20:40:43** (más reciente), con un intervalo de **2,03 días** en hora local. Esto no demuestra registro ininterrumpido ni sobrescritura efectiva.

## Intervalo de registros retenidos

| Métrica | Resultado observado |
|---|---|
| Evento Security retenido más antiguo | 2026-11-15 20:02:39 |
| Más reciente | 2026-11-17 20:40:43 |
| Diferencia entre extremos | 2,03 días |

La consulta original de siete días **no debe presentarse como siete días de actividad observada**: al medir la retención, solo existía aproximadamente dos días de historial accesible.

## Observaciones de eventos

| ID | Registros obtenidos | Significado |
|---|---:|---|
| 4624 | 638 | Inicio de sesión exitoso |
| 4634 | 37 | Cierre de sesión |
| 4672 | 614 | Asignación de privilegios especiales a una nueva sesión |
| 4688 | 13 | Creación de proceso |
| 4625 | No recuperado | Inicio de sesión fallido |
| 1102 | No recuperado | Eliminación del registro de auditoría Security |

**Total: 1.302 eventos coincidentes.** Son registros, no necesariamente sesiones, cuentas, procesos o incidentes distintos.

## Muestra verificada por tipo de inicio de sesión

Una consulta posterior acotada recuperó los **100 eventos 4624 más recientes** y extrajo el campo XML `LogonType`. El resultado fue:

| Tipo | Interpretación | Eventos de la muestra |
|---|---|---:|
| 5 | Inicio de sesión de servicio | 100 |

Las sesiones de servicio pueden ser normales en Windows. No se demostró conducta maliciosa. Una consulta inicial de XML sin límite no terminó con rapidez y fue sustituida por otra de tamaño acotado. Esto constituye un ajuste metodológico por rendimiento, no un fallo demostrado del registro.

**Límite:** los 100 eventos corresponden a una muestra posterior, no a los 638 registros anteriores ni a una muestra aleatoria representativa.

## Muestra verificada por categoría de cuenta

Otra consulta posterior, limitada a **cinco eventos 4624**, clasificó el SID `TargetUserSid` sin revelar nombres de usuarios.

| Tipo | Identidad clasificada | Cantidad |
|---|---|---:|
| 5 (Servicio) | LocalSystem (`S-1-5-18`) | 5 |

Estos cinco eventos son compatibles con funcionamiento habitual del sistema, pero no permiten identificar los servicios responsables ni generalizar al resto de sesiones. La consulta original más extensa no respondió con rapidez, por lo que se limitó a cinco registros.

## Evaluación verificada de política de auditoría

Se ejecutó `auditpol /get /category:*` en modo lectura. A continuación se traducen las configuraciones relevantes de la consola española.

| Subcategoría | Configuración observada | Consecuencia |
|---|---|---|
| Inicio de sesión | Aciertos y errores | Auditoría de ambos resultados configurada |
| Cierre de sesión | Aciertos | Auditoría de cierres exitosos |
| Inicio de sesión especial | Aciertos | Auditoría de accesos especiales |
| Creación de proceso | Sin auditoría | No estaba habilitada al inspeccionar |
| Terminación de procesos | Sin auditoría | Auditoría deshabilitada |
| Sistema de archivos / Registro | Sin auditoría | Subcategorías de acceso a objetos deshabilitadas |
| Cambio de directiva de auditoría | Aciertos | Cambios exitosos configurados |
| Integridad del sistema | Aciertos y errores | Ambos resultados configurados |

**Posible brecha de visibilidad:** al inspeccionar la política, creación de procesos estaba deshabilitada, reduciendo la futura cobertura habitual de eventos 4688 si la configuración se mantiene.

**Discrepancia abierta:** la primera consulta encontró **13 eventos 4688**. Un cambio histórico de política es una hipótesis posible, pero no comprobada. No se modificaron políticas durante el análisis; se priorizaron consultas acotadas para evitar exponer rutas o parámetros privados.

## Muestra de marcas de tiempo de eventos 4688

Se obtuvieron metadatos de los **cinco registros 4688 más recientes**:

| Fecha/hora local | Evento | ID de registro |
|---|---|---|
| 2026-11-16 14:30:43 | 4688 | 2266586 |
| 2026-11-16 14:30:43 | 4688 | 2266585 |
| 2026-11-16 14:30:43 | 4688 | 2266584 |
| 2026-11-16 14:30:43 | 4688 | 2266583 |
| 2026-11-16 14:30:43 | 4688 | 2266582 |

Los cinco compartían segundo y tenían identificadores consecutivos, compatibles con un grupo de creaciones de proceso registradas. La política de auditoría exacta vigente **en el momento de esos eventos** no se conoce.

No se confirmó ningún proceso concreto, cambio de política, incidente o causa raíz.

## Búsqueda de eventos 4719

La consulta de solo lectura:

```powershell
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4719} -MaxEvents 5 |
    Select-Object TimeCreated, Id, RecordId
```

no devolvió coincidencias; PowerShell informó `NoMatchingEventsFound`.

Esto demuestra únicamente que **no se recuperaron eventos 4719 del registro retenido**. No permite asegurar que la política nunca haya cambiado ni explicar los eventos 4688. La retención circular limita la interpretación histórica.

## Metadatos de eventos 4672

Una consulta acotada recuperó los cinco registros 4672 más recientes:

| Fecha/hora local | Evento | ID de registro |
|---|---|---:|
| 2026-11-17 21:00:46 | 4672 | 2286080 |
| 2026-11-17 20:57:13 | 4672 | 2286068 |
| 2026-11-17 20:54:58 | 4672 | 2285921 |
| 2026-11-17 20:53:51 | 4672 | 2285919 |
| 2026-11-17 20:44:56 | 4672 | 2285681 |

Se confirmó la presencia de cinco registros de asignación de privilegios especiales. No sabemos por esos metadatos si corresponden a sesiones o cuentas distintas, qué privilegios recibieron o si hubo actividad no autorizada.

## Muestra verificada de privilegios 4672

Otra consulta limitada a tres eventos analizó localmente los campos XML `SubjectUserSid` y `PrivilegeList`:

| Fecha/hora local | Categoría de cuenta | Privilegios asignados |
|---|---|---:|
| 2026-11-17 21:05:00 | LocalSystem (`S-1-5-18`) | 12 |
| 2026-11-17 21:00:46 | LocalSystem (`S-1-5-18`) | 12 |
| 2026-11-17 20:57:13 | LocalSystem (`S-1-5-18`) | 12 |

**Lista idéntica de privilegios para los tres:**

- `SeAssignPrimaryTokenPrivilege`
- `SeTcbPrivilege`
- `SeSecurityPrivilege`
- `SeTakeOwnershipPrivilege`
- `SeLoadDriverPrivilege`
- `SeBackupPrivilege`
- `SeRestorePrivilege`
- `SeDebugPrivilege`
- `SeAuditPrivilege`
- `SeSystemEnvironmentPrivilege`
- `SeImpersonatePrivilege`
- `SeDelegateSessionUserImpersonatePrivilege`

**Interpretación:** LocalSystem es una identidad integrada con altos privilegios. Los registros pueden corresponder a actividad legítima, pero no identifican el servicio responsable. El evento 4672 documenta **asignación**, no ejecución o abuso de privilegios.

**Limitaciones:** son tres eventos recientes de una medición distinta, no los 614 registros originales. Tampoco representan necesariamente tres servicios o cuentas diferentes.

## Correlación verificada entre eventos 4672 y 4624

Una consulta de solo lectura comparó el `SubjectLogonId` de un 4672 reciente con el `TargetLogonId` de eventos 4624 dentro de una ventana acotada desde dos minutos antes hasta un minuto después.

| Campo | Resultado |
|---|---|
| Evento 4672 seleccionado | 2026-11-17 21:08:51 |
| Registros 4624 coincidentes | 1 |
| `CorrelationFound` | `True` |

**Hallazgo:** se recuperó un 4624 con el mismo identificador de sesión. Esto demuestra vinculación entre eventos, **no atribución a una persona ni acceso indebido**.

**Limitaciones:** se analizó un 4672 reciente y un máximo de 100 candidatos 4624 en el intervalo. No se publicó el Logon ID real y no puede generalizarse a todos los eventos.

## Clasificación posterior de una sesión correlacionada

Una **consulta independiente posterior** seleccionó el 4672 más reciente disponible, comparó IDs de sesión contra 4624 cercanos y clasificó el SID conocido de la cuenta:

| Campo | Resultado |
|---|---|
| `MatchFound` | `True` |
| `LogonType` | `5` (Servicio) |
| `AccountCategory` | `LocalSystem` |

**Interpretación:** una sesión exitosa de servicio bajo LocalSystem estaba correlacionada con asignación de privilegios. Esto es compatible con actividad del sistema y no demuestra abuso.

**Distinción esencial:** la segunda consulta no entregó marca de tiempo ni ID de registro. **No se puede asegurar** que clasificara el mismo evento del 17/11/2026 a las 21:08:51. Un intento anterior fallido de reutilización de variables tampoco invalida los dos resultados positivos.

## Interpretación y limitaciones generales

- Una cantidad elevada de 4672 no implica actividad sospechosa sin conocer categorías de cuentas y contexto.
- Que 4625 o 1102 no aparezcan **no demuestra** que jamás ocurrieron.
- La consulta inicial utilizó `-ErrorAction SilentlyContinue`; la evaluación posterior puede revisar explícitamente posibles errores.
- Se verificó la configuración del registro después de corregir un error inicial de orden de pegado. El modo circular limita el historial.
- La muestra de 100 eventos `LogonType` y la muestra separada de cinco `TargetUserSid` no permiten atribución completa.
- No se revisaron todos los detalles personales, capturas ni exportaciones forenses, y no se confirmó severidad o incidente.

## Próximas investigaciones propuestas

1. Mantener sin resolver la discrepancia 4688 frente a la política actual, salvo que nueva evidencia justifique consultas acotadas.
2. Si es útil, ampliar cuidadosamente las muestras más allá de servicios LocalSystem.
3. Conservar como observaciones distintas las dos correlaciones exitosas, sin afirmar que son la misma sesión.
4. **Completado:** [resumen ejecutivo](executive-summary.md) y [recomendaciones](monitoring-recommendations.md) no implementadas.

## Referencias oficiales

- [auditpol /get](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/auditpol-get)
- [Audit Process Creation](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/audit-process-creation)
- [Evento 4624](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4624)
- [SID conocidos de Windows](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-identifiers)
- [Evento 4672](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4672)
- [Evento 4719](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4719)
- [Evento 4688](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4688)
- [Get-WinEvent](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.diagnostics/get-winevent)

*Las referencias explican los eventos; los hallazgos reales provienen solamente de las salidas de consola suministradas.*