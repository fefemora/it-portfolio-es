# Resumen ejecutivo — Análisis de eventos de seguridad de Windows

[Proyecto](../README.md) · [Hallazgos técnicos](event-analysis.md) · [Guía de reproducción](lab-guide.md) · [Recomendaciones](monitoring-recommendations.md) · [Portafolio](../../../README.md)

**Tipo de evaluación:** revisión de eventos reales de una estación de trabajo, mediante consultas de solo lectura  
**Estado:** evaluación inicial completada; ningún incidente de seguridad confirmado  
**Evidencia:** resultados anonimizados de PowerShell facilitados por el usuario en noviembre de 2026  
**Alcance:** únicamente registro Security de Windows; sin SIEM, monitoreo de red, análisis EDR ni exportación independiente de evidencias

![Resumen gráfico de los hallazgos](../diagrams/security-review-overview.svg)

## Conclusión ejecutiva

La investigación demostró la recuperación e interpretación de registros Security y la **correlación de un inicio de sesión exitoso (4624) con un evento de asignación de privilegios especiales (4672) mediante sus Logon IDs**.

Una clasificación acotada e independiente identificó **Logon Type 5 (Service)** y la cuenta integrada **LocalSystem**. Los resultados son compatibles con actividad normal de servicios de Windows; **no demuestran abuso de privilegios ni compromiso del equipo**.

Se identificaron dos limitaciones que merecen seguimiento: **registro Security circular de 20 MiB con un intervalo observado de solo 2,03 días** y **auditoría de creación de procesos deshabilitada** al revisar la política. Son hallazgos de visibilidad, no prueba de incidente, filtración ni incumplimiento.

## Hallazgos principales

| Área | Evidencia verificada | Interpretación |
|---|---|---|
| Disponibilidad | Security habilitado; 33.898 registros en el momento de revisión | Disponible para investigación local |
| Retención | Máximo 20.971.520 bytes (20 MiB), circular; más antiguo 2026-11-15 20:02:39, más reciente 2026-11-17 20:40:43 | **2,03 días observados**; una consulta de siete días no equivalía a siete días completos de datos |
| Inventario | 4624: 638; 4634: 37; 4672: 614; 4688: 13 | **1.302 registros**, no personas, sesiones o incidentes únicos |
| Accesos de servicio | 100 de los 100 eventos 4624 más recientes de una muestra fueron tipo 5 | Muestra acotada; no representa a toda la población |
| Cuenta integrada | 5 de 5 eventos 4624 en otra muestra: tipo 5 / LocalSystem | Compatible con servicios habituales |
| Privilegios especiales | 3 registros 4672 recientes de LocalSystem con los mismos 12 privilegios | **Asignación no equivale a uso de privilegios** |
| Correlación | Una coincidencia de Logon ID 4672/4624; una consulta posterior devolvió MatchFound=True, tipo 5, LocalSystem | Correlación verificada, **sin demostrar que ambas consultas se refieran a la misma sesión exacta** |
| Auditoría | Aciertos/errores de inicio de sesión y aciertos de sesión especial habilitados; creación de procesos sin auditoría | Brecha de visibilidad de procesos |
| Cambio de política | Sin registros 4719 coincidentes recuperados | No descarta cambios históricos |

## Pregunta investigada: ¿por qué había eventos 4688?

Cinco registros 4688 recientes tenían la fecha **2026-11-16 14:30:43** y números de registro consecutivos. En una comprobación posterior, `Process Creation` figuraba como `No Auditing`. La consulta de eventos 4719 no recuperó coincidencias.

**Estado: sin resolver.** Un cambio anterior de política es posible, pero **no fue demostrado**. No se identificaron nombres de procesos, la política efectiva en la fecha de esos registros ni una causa raíz. No corresponde afirmar que se ejecutaron procesos maliciosos.

## Recomendaciones propuestas, no implementadas en esta investigación

1. **Revisar capacidad y objetivos de retención:** definir la ventana requerida para investigar y evaluar tamaño del registro o reenvío centralizado, considerando privacidad, recursos y operación.
2. **Evaluar un piloto de auditoría de creación de procesos:** revisar el volumen de eventos adicional y el posible carácter sensible de los argumentos de línea de comandos.
3. **Definir una línea base de monitoreo:** analizar LocalSystem y sesiones de servicio en contexto, sin considerarlas automáticamente sospechosas.
4. **Mejorar captura de evidencia:** consultas acotadas y registros anonimizados, con fechas de recolección y limitaciones explícitas.
5. **Reinvestigar discrepancias solo con respaldo:** no atribuir cambios históricos a una política sin evidencia suficiente.

Consulta las [recomendaciones de monitoreo](monitoring-recommendations.md) para prioridades y criterios de verificación.

## Alcance e integridad de la evidencia

- Las cifras provienen de consultas ejecutadas en **momentos diferentes**; no deben compararse como si fueran un único conjunto.
- Consultar siete días no crea registros que la retención circular ya descartó.
- Se utilizaron muestras de **3, 5 y 100** eventos; los resultados no son generalizables a todos los usuarios ni a todo el historial.
- No hubo preservación forense completa, cadena de custodia, ataque simulado, remediación, atribución individual, confirmación de incidente ni investigación SIEM.
- El trabajo documenta **observaciones técnicas reales**, no un ataque simulado ni una implementación completa de hardening.

## Referencias oficiales

- [Evento 4624 — Inicio de sesión exitoso](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4624)
- [Evento 4672 — Privilegios especiales asignados](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4672)
- [Audit Process Creation](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/audit-process-creation)
- [EventLogMode: Circular](https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.eventing.reader.eventlogmode)
- [Windows Event Forwarding y detección de intrusiones](https://learn.microsoft.com/en-us/windows/security/operating-system-security/device-management/use-windows-event-forwarding-to-assist-in-intrusion-detection)

*Las fuentes oficiales respaldan conceptos y técnicas; no certifican independientemente los resultados del equipo.*