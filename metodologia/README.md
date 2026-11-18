# Cómo trabajo — Metodología de investigación y soporte TI

[Portafolio](../README.md) · [Plantilla de investigación](plantilla-investigacion.md) · [Criterios de evidencia](evidencia-y-validacion.md)

## Propósito

Este apartado reúne una metodología **propia y reutilizable** para documentar problemas de soporte TI, casos de seguridad y decisiones de administración de sistemas. Está alineada en sus principios con prácticas de gestión de incidentes y cambios, pero **no se presenta como un procedimiento oficial ni una certificación de ITIL**.

## Ciclo de trabajo

1. **Recepción y alcance:** identificar síntomas, impacto, urgencia, servicios afectados, límites y autorizaciones.
2. **Recopilación de evidencias:** elegir consultas mínimas, preferiblemente de solo lectura; registrar la fuente, la fecha y el contexto cuando estén disponibles.
3. **Hipótesis y diagnóstico:** separar hechos de explicaciones posibles; comparar hipótesis y probar primero las de menor riesgo.
4. **Decisión y gestión de cambios:** valorar continuidad operativa, privacidad, permisos, respaldo y reversión antes de modificar sistemas.
5. **Resolución y validación:** aplicar únicamente acciones autorizadas; comprobar el resultado esperado y detectar efectos secundarios.
6. **Documentación y aprendizaje:** registrar qué funcionó, qué no se verificó, cuándo escalar y qué prevenir en futuras incidencias.

## Tres formas de aplicar el método

| Modalidad | Qué significa | Ejemplo del portafolio |
|---|---|---|
| Caso real resuelto | Hay evidencia de ejecución y resultado funcional; la causa raíz puede seguir abierta | [INC-001 — Actualizaciones de Windows](../IT-Support/Windows-Troubleshooting-Lab/README.md) |
| Simulación documentada | Es una propuesta de diagnóstico, no una intervención realizada | [INC-002 — Microsoft 365 / MFA](../IT-Support/Service-Desk-Incident-Management/incidents/INC-002-M365-Authentication.md) |
| Evaluación técnica con cambios controlados | Se observa la línea base, se priorizan riesgos y se distingue lo implementado de lo pendiente | [Windows Security Hardening](../Cybersecurity/Windows-Security-Hardening/README.md) |

## Criterios de cierre

- **Resuelto:** el servicio funciona y hay validación funcional, aunque no siempre se confirma una causa raíz única.
- **Escalado:** las pruebas indican que se necesitan permisos, equipos o decisiones fuera del alcance.
- **Evaluación finalizada:** se entregan hallazgos y recomendaciones; los cambios propuestos pueden seguir pendientes.
- **Simulación documentada:** se presenta un procedimiento razonado; no se atribuyen resultados técnicos reales.

## Principios de actuación

**Evidencia antes que suposición · Mínimo privilegio · Privacidad desde el diseño · Cambios autorizados · Validación independiente · Comunicación clara.**

No se publican datos de clientes, credenciales, registros privados ni identificadores sensibles. Los casos reales, simulados y planificados se identifican explícitamente.
