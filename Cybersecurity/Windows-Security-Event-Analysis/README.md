# <img src="../../assets/icons/cybersecurity.svg" width="32" alt="Ciberseguridad"> Análisis de eventos de seguridad de Windows

![Windows](https://img.shields.io/badge/Windows-Eventos_de_seguridad-14283F?style=flat-square) ![PowerShell](https://img.shields.io/badge/PowerShell-Solo_lectura-14283F?style=flat-square) ![Evidencia](https://img.shields.io/badge/CASO_REAL-Evaluaci%C3%B3n_inicial_completa-059669?style=flat-square)

[← Ciberseguridad](../README.md) · [Portafolio](../../README.md) · **[Resumen ejecutivo](docs/executive-summary.md)** · [Hallazgos](docs/event-analysis.md) · [Guía](docs/lab-guide.md) · [Recomendaciones](docs/monitoring-recommendations.md)

[![Resumen de investigación de eventos de Windows](diagrams/security-review-overview.svg)](docs/executive-summary.md)

Revisé los registros de seguridad de una estación de trabajo Windows para entender qué actividad permitían observar y cómo sus ajustes de auditoría y retención afectaban la investigación. Verifiqué una correlación acotada entre eventos **4624 y 4672**. En la medición, el registro Security conservaba aproximadamente **2,03 días** de eventos.

**Resultado:** una línea base inicial de monitoreo con límites de visibilidad identificados. **No se confirmó un incidente ni actividad no autorizada.**

## Proceso de investigación

1. **Comprobar el historial disponible:** configuración, retención e identificadores de eventos presentes.
2. **Revisar muestras acotadas:** clasificar tipos de inicio de sesión, cuentas integradas y asignación de privilegios.
3. **Correlacionar con cuidado:** relacionar eventos 4624 y 4672 mediante identificadores de sesión sin publicar datos de cuentas.
4. **Documentar lo comprobado:** registrar límites de auditoría, eventos 4688 históricos sin explicación confirmada y preguntas abiertas.

La [guía de 12 pasos](docs/lab-guide.md) conserva los comandos y las salidas. La investigación no confirmó ningún incidente.

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