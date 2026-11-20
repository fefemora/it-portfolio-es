# <img src="../../assets/icons/it-support.svg" width="32" alt="Mesa de ayuda"> Mesa de ayuda L2 · Casos de incidentes

![Soporte](https://img.shields.io/badge/Mesa_de_ayuda-L2-14283F?style=flat-square) ![Simulados](https://img.shields.io/badge/Escenarios-3_simulados-64748B?style=flat-square) ![Método](https://img.shields.io/badge/Método-Alineado_con_ITIL-14283F?style=flat-square)

[← Soporte TI](../README.md) · [Portafolio](../../README.md)

[![Resumen de los incidentes de mesa de ayuda L2](incident-overview.svg)](incidents/INC-002-M365-Authentication.md)

Tres escenarios de incidentes exploran situaciones que pueden llegar al soporte de segundo nivel: pérdida de acceso MFA, problemas de resolución DNS y sesiones VPN conectadas sin acceso a servicios internos.

**Son casos ficticios.** Los pasos diagnósticos son propuestas y no se ejecutaron en sistemas de clientes.

## Casos y análisis técnicos

| Incidente | Pregunta principal | Material relacionado |
|---|---|---|
| **[INC-002 · Autenticación en Microsoft 365](incidents/INC-002-M365-Authentication.md)** | ¿El fallo está en MFA, la cuenta o los requisitos de acceso condicional? | [Análisis de inicio de sesión en Entra ID](incidents/INC-002-Technical-Analysis.md) |
| **[INC-003 · Resolución DNS](incidents/INC-003-DNS-Resolution.md)** | ¿El problema está en el cliente, el servidor DNS o la ruta hacia el servicio? | [Árbol de decisión DNS](incidents/INC-003-DNS-Decision-Tree.md) |
| **[INC-004 · Conectividad VPN](incidents/INC-004-VPN-Connectivity.md)** | ¿El túnel permite llegar efectivamente a los recursos internos? | [Procedimiento de diagnóstico VPN](incidents/INC-004-VPN-Connectivity.md) |

## Cómo se gestionan los casos

```mermaid
flowchart LR
    A["Reporte y alcance"] --> B["Impacto y prioridad"]
    B --> C["Investigar"]
    C --> D{"¿Resolución autorizada?"}
    D -->|Sí| E["Validar con el usuario"]
    D -->|No| F["Escalar"]
    F --> E
    E --> G["Documentar y cerrar"]
    style A fill:#D1FAE5,stroke:#059669,color:#065F46
    style G fill:#D1FAE5,stroke:#059669,color:#065F46
```

La verificación de identidad, el mínimo privilegio y los criterios de escalamiento forman parte del diagnóstico. No son controles que debamos eludir para recuperar un servicio.

## Documentación

La [plantilla de informe de incidente](templates/incident-report-template.md) permite registrar impacto, investigación, cambios propuestos, criterios de validación y comunicación con usuarios.

**Alcance:** un procedimiento bien documentado no equivale a una resolución verificada en producción. Los resultados de estos escenarios no se presentan como tickets reales de clientes.

[← Soporte TI](../README.md)
