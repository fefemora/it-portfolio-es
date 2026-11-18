# Laboratorio de análisis de eventos de seguridad de Windows

**Área:** Ciberseguridad / Blue Team  
**Estado:** Evaluación inicial de solo lectura documentada y revisada; mejoras de monitoreo pendientes de implementación.  
**Entorno:** Estación de trabajo Windows, registro local Security y Windows PowerShell. No se requiere SIEM ni nuevas máquinas virtuales.

[Portafolio](../../README.md) · **[Resumen ejecutivo](docs/executive-summary.md)** · [Hallazgos técnicos](docs/event-analysis.md) · [Guía del laboratorio](docs/lab-guide.md) · [Recomendaciones](docs/monitoring-recommendations.md)

## Objetivo

Investigar eventos nativos del registro Security de Windows mediante comandos PowerShell de solo lectura, interpretar resultados sin asumir actividad maliciosa y elaborar evidencias reproducibles y seguras para un proceso inicial de monitoreo defensivo.

## Proceso de investigación

1. **Configuración del registro Security:** verificada después de corregir inicialmente un error en el orden de pegado del comando.
2. **Conteo de identificadores de evento:** realizado con ventana de búsqueda de siete días, aunque los datos retenidos solo abarcaban aproximadamente **2,03 días**.
3. **Fechas del evento más antiguo y reciente:** verificadas.
4. **Clasificación de los 100 eventos 4624 más recientes:** todos correspondían a **Logon Type 5 / Service**.
5. **Clasificación de cinco eventos 4624 por SID:** los cinco correspondían a **LocalSystem / S-1-5-18**.
6. **Revisión de política de auditoría:** completada con `auditpol /get /category:*`.
7. **Fecha de cinco eventos 4688:** registros observados con fecha **2026-11-16 14:30:43**.
8. **Búsqueda de eventos 4719:** no se recuperaron registros coincidentes.
9. **Recuperación de cinco marcas de tiempo e identificadores de eventos 4672:** completada.
10. **Clasificación acotada de tres eventos 4672:** los tres eran LocalSystem y contenían los mismos **12 privilegios**.
11. **Correlación de eventos 4672/4624 por Logon ID:** se verificó **una coincidencia** en una ventana acotada.
12. **Clasificación posterior de una sesión correlacionada:** se observó `MatchFound=True`, `LogonType=5` y `AccountCategory=LocalSystem`.
13. **Informe y recomendaciones:** se prepararon el resumen ejecutivo, el análisis técnico y recursos visuales.

## Resumen ejecutivo

[![Resumen visual del análisis de eventos de Windows](diagrams/security-review-overview.svg)](docs/executive-summary.md)

La [síntesis ejecutiva](docs/executive-summary.md) presenta los resultados principales, el [análisis técnico](docs/event-analysis.md) registra la evidencia, la [guía](docs/lab-guide.md) documenta las consultas y las [recomendaciones](docs/monitoring-recommendations.md) presentan mejoras propuestas.

El resultado es **un análisis inicial de registros**, no el descubrimiento de un incidente confirmado.

## Hallazgos iniciales

La primera consulta exitosa recuperó **1.302 eventos** de cuatro identificadores:

| Evento | Descripción | Registros |
|---|---|---:|
| 4624 | Inicio de sesión exitoso | 638 |
| 4672 | Privilegios especiales asignados a un nuevo inicio de sesión | 614 |
| 4634 | Cierre de sesión | 37 |
| 4688 | Creación de proceso | 13 |

Los identificadores **4625** y **1102** no se recuperaron en esa consulta. Los conteos corresponden a **registros**, no a usuarios únicos ni incidentes confirmados.

Una consulta posterior de 100 eventos 4624 identificó **100 inicios de sesión de tipo 5 / Service**. Esto no significa que los 638 registros de la consulta anterior fueran de ese tipo.

Un análisis independiente de cinco eventos 4624 clasificó los cinco como **LocalSystem (S-1-5-18)**. El resultado no permite inferir la categoría de cuenta de los otros eventos.

### Auditoría y eventos de proceso

La política de auditoría consultada registraba aciertos y errores de inicio de sesión, auditoría de inicios de sesión especiales exitosos y cierres de sesión exitosos. Sin embargo, la subcategoría **Creación del proceso** figuraba como **Sin auditoría** al realizar la comprobación.

Se observaron cinco eventos 4688 con fecha **2026-11-16 14:30:43**, cuyos identificadores de registro eran consecutivos (**2266582–2266586**). No se estableció por qué había eventos 4688 históricos a pesar de la configuración actual de no auditoría. **Un cambio de política anterior es solamente una hipótesis**, no un hecho probado.

La búsqueda posterior de eventos 4719 arrojó `NoMatchingEventsFound`, pero esto no descarta cambios históricos cuyos eventos ya no estén retenidos.

### Eventos 4672 y correlación

Se observaron cinco registros 4672 con marcas de tiempo comprendidas entre las **20:44:56 y las 21:00:46 del 17/11/2026**. Una muestra separada de tres eventos correspondía a LocalSystem y contenía los mismos **12 privilegios**. Esa muestra incluía un evento posterior a las **21:05:00**.

**La asignación de privilegios no demuestra su utilización.**

Una comprobación acotada encontró **una coincidencia 4624** para un evento 4672 del **17/11/2026 a las 21:08:51**, correlacionando localmente `SubjectLogonId` con `TargetLogonId` y mostrando solo el resultado booleano y el conteo.

Otra consulta posterior de correlación y clasificación devolvió `MatchFound=True`, `LogonType=5` y `AccountCategory=LocalSystem`. Como esa consulta **no mostró la marca de tiempo**, no puede afirmarse que se refiera exactamente a la sesión anterior de las 21:08:51.

**No se confirmó actividad maliciosa.**

## Estado y limitaciones de evidencia

Los conteos iniciales provienen de resultados reales de PowerShell copiados desde la consola. No se realizó una exportación independiente y todavía no se registró la hora exacta de todas las consultas.

La comprobación posterior del canal Security informó:

- **Habilitado.**
- **33.898 registros** en ese momento.
- Tamaño máximo **20 MiB**.
- Modo de retención **Circular**.
- Evento más antiguo: **2026-11-15 20:02:39**.
- Evento más reciente: **2026-11-17 20:40:43**.
- Intervalo observado: **2,03 días**.

Este intervalo no garantiza una política mínima de retención ni prueba que hayan existido eventos durante siete días completos.

**Relación con el proyecto posterior:** [Windows Security Hardening](../Windows-Security-Hardening/README.md) documentó más adelante un cambio autorizado que aumentó el límite del registro Security de 20 a 128 MiB. Esta investigación conserva sus **mediciones originales anteriores al cambio**.

## Referencias oficiales

- [auditpol /get — Microsoft Learn](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/auditpol-get)
- [Auditoría de creación de procesos](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/audit-process-creation)
- [Get-WinEvent](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.diagnostics/get-winevent)
- [Auditoría de seguridad de Windows](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/basic-security-audit-policies)
- [MITRE ATT&CK — Tácticas empresariales](https://attack.mitre.org/tactics/enterprise/)

*Las referencias sustentan la interpretación y los métodos, pero no prueban ejecución. La evidencia real procede de las consultas documentadas.*