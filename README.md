# Portafolio de Soporte TI, Administración de Sistemas y Ciberseguridad

**Soporte TI L1/L2 · Administración de sistemas · Seguridad de la información · Ciberseguridad**

Bienvenido a mi portafolio profesional. Aquí documento cómo investigo problemas técnicos, organizo evidencias, evalúo riesgos y valido resultados. Mis proyectos reúnen **soporte y resolución de incidentes**, **operaciones seguras de TI** y **análisis de seguridad**.

**[English version / Versión en inglés](https://github.com/fefemora/it-portfolio-en)**

> **Transparencia de evidencias:** distingo los **casos reales**, los **escenarios simulados** y los **proyectos planificados**. Los ejercicios ficticios no se presentan como incidentes reales ni como controles implementados. **Estado de traducción:** la estructura y todos los archivos originales ya están replicados; la adaptación al español de los documentos técnicos y textos de diagramas sigue en curso. Algunos enlaces todavía llevan a contenido en inglés.

## Explora el portafolio

**[Soporte TI y resolución de incidentes](IT-Support/README.md)** · [Evaluación de riesgos de seguridad de la información](Information-Security/Enterprise-Risk-Assessment/README.md) · **[Análisis real de eventos de seguridad de Windows](Cybersecurity/Windows-Security-Event-Analysis/README.md)**

[![Mapa de navegación de casos de soporte TI](IT-Support/it-support-navigation.svg)](IT-Support/README.md)

## Proyectos destacados

| Proyecto | Competencias demostradas | Evidencia | Estado |
|---|---|---|---|
| **[INC-001 — Diagnóstico de Windows Update](IT-Support/Windows-Troubleshooting-Lab/README.md)** | PowerShell, eventos, almacenamiento, validación funcional | Caso real, datos anonimizados | **Completado** |
| **[Gestión de incidentes de mesa de ayuda L2](IT-Support/Service-Desk-Incident-Management/README.md)** | Microsoft 365/MFA, DNS, VPN y escalamiento | Tres escenarios simulados | **Documentados** |
| **[Evaluación de riesgos de seguridad empresarial](Information-Security/Enterprise-Risk-Assessment/README.md)** | Inventario, matriz de riesgos, NIST CSF 2.0, CIS IG1 y plan de tratamiento | Empresa ficticia | **Documentación completada** |
| **[Análisis de eventos de seguridad de Windows](Cybersecurity/Windows-Security-Event-Analysis/README.md)** | Eventos 4624/4672, política de auditoría, correlación y límites de retención | Investigación real de solo lectura | **Evaluación inicial completada** |
| **[Windows Security Hardening](Cybersecurity/Windows-Security-Hardening/README.md)** | Línea base, Defender, firewall, auditoría, cambios y validación | Evidencia real, 30 pasos documentados | **Evaluación finalizada; mejoras pendientes** |

## Casos y resultados relevantes

### 01 — Diagnóstico real de Windows: INC-001

[![Recuperación de espacio en disco](IT-Support/Windows-Troubleshooting-Lab/storage-recovery.svg)](IT-Support/Windows-Troubleshooting-Lab/README.md)

Investigación de fallos de actualización relacionados con el error `0x80073D02`. El espacio libre aumentó de **18,6 GiB a 63,1 GiB** y se validó la actualización satisfactoria de aplicaciones. **No se confirmó una causa raíz única.**

### 02 — Mesa de ayuda L2: escenarios simulados

[![Escenarios simulados de mesa de ayuda](IT-Support/Service-Desk-Incident-Management/incident-overview.svg)](IT-Support/Service-Desk-Incident-Management/README.md)

Tres informes ficticios muestran cómo formular hipótesis, investigar, aplicar criterios de seguridad y decidir cuándo escalar un incidente. Los procedimientos propuestos **no se presentan como acciones ejecutadas**.

### 03 — Evaluación de riesgos de seguridad de la información

[![Panel ejecutivo de riesgos](Information-Security/Enterprise-Risk-Assessment/reports/executive-dashboard.svg)](Information-Security/Enterprise-Risk-Assessment/README.md)

Caso ficticio de una empresa de 50 personas, con ocho escenarios ilustrativos: dos críticos, cuatro altos y dos moderados. Incluye NIST CSF 2.0, CIS Controls IG1 y una hoja de ruta propuesta de 90 días. **No se evaluó ninguna organización real.**

### 04 — Investigación real de registros de seguridad

[![Resumen visual de eventos de seguridad](Cybersecurity/Windows-Security-Event-Analysis/diagrams/security-review-overview.svg)](Cybersecurity/Windows-Security-Event-Analysis/README.md)

Análisis de eventos 4624, 4672 y 4688, con correlación acotada de sesión, evaluación de políticas y detección de limitaciones de monitoreo. **No se confirmó actividad maliciosa.**

### 05 — Evaluación y endurecimiento de seguridad de Windows

[![Proceso de evaluación y validación](Cybersecurity/Windows-Security-Hardening/diagrams/hardening-workflow.svg)](Cybersecurity/Windows-Security-Hardening/README.md)

Evaluación de **30 pasos**, con evidencia real de Windows Firewall, Microsoft Defender, PowerShell y auditoría de autenticación. Se aprobó **un cambio específico**: aumentar el tamaño máximo del registro Security de **20 MiB a 128 MiB**. La retención de siete días no se ha validado, y los cambios de BitLocker y Secure Boot permanecen pendientes.

## Competencias técnicas demostradas

| Área | Evidencia en el portafolio |
|---|---|
| Windows y PowerShell | Diagnóstico, eventos y verificación del almacenamiento |
| Soporte TI L2 y gestión de incidentes | Priorización, hipótesis, documentación y escalamiento |
| Microsoft 365 e identidades | Casos simulados de Entra ID y MFA |
| Redes | Diagnósticos simulados de DNS y VPN |
| Seguridad de la información / GRC | Inventario, evaluación y tratamiento de riesgos |
| Ciberseguridad / Blue Team | Auditoría Windows, análisis de eventos, controles y validación |
| Comunicación profesional | Informes técnicos, resúmenes ejecutivos y límites de evidencia |

## Próximos proyectos

| Área | Proyecto previsto |
|---|---|
| Administración de sistemas | Laboratorio de Active Directory e identidades |
| Redes | Laboratorio de diagnóstico de conectividad |
| Ciberseguridad | Laboratorio SIEM con Wazuh |
| Cloud | Redes y seguridad en la nube |

## Metodología de documentación

Los proyectos describen el **problema u objetivo**, el **alcance**, las **hipótesis**, los **procedimientos**, la **evidencia**, la **validación**, los **riesgos**, las **limitaciones** y las **lecciones aprendidas**. Las recomendaciones se distinguen de los cambios autorizados y efectivamente realizados.

No publico credenciales, datos confidenciales de clientes ni identificadores sensibles de infraestructura.

## Orientación profesional

Mi objetivo es combinar experiencia en soporte y administración de sistemas con competencias en ciberseguridad, gestión de identidades y seguridad de infraestructura y servicios cloud.

## Contacto

- **GitHub:** [github.com/fefemora](https://github.com/fefemora)
- **Versión en inglés:** [it-portfolio-en](https://github.com/fefemora/it-portfolio-en)

---

*Portafolio profesional en proceso de localización al español. La evidencia real y los escenarios ficticios están diferenciados.*