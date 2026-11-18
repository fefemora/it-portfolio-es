# Estado de localización — Portafolio en español

**Repositorio de referencia:** [it-portfolio-en](https://github.com/fefemora/it-portfolio-en)  
**Repositorio de destino:** [it-portfolio-es](https://github.com/fefemora/it-portfolio-es)

## Resumen de migración

- **47/47 archivos originales replicados:** se conservan estructura, rutas, documentos fuente, archivos de evidencia y diagramas.
- **11/11 archivos SVG adaptados:** se tradujeron sus etiquetas y textos principales conservando los elementos gráficos y conexiones. **Revisión visual de desbordamiento y legibilidad pendiente.**
- **17/32 archivos Markdown traducidos al español:** portada, secciones generales y **todos los documentos de Soporte TI**, incluidos los cuatro incidentes y sus anexos.
- **15/32 Markdown restantes:** el contenido original en inglés se mantiene temporalmente para evitar pérdidas mientras se localiza.
- **Tres archivos .txt de evidencia:** se mantienen exactamente como los registros originales, sin alterar la salida de los comandos ni los datos observados.
- **LICENSE:** se conserva el texto legal original.

**Importante:** este repositorio todavía está en proceso de traducción. Que un archivo exista aquí **no significa** que su traducción haya concluido. No se inventan resultados ni se modifican las evidencias reales.

## Archivos Markdown ya localizados

1. `README.md`
2. `Cloud/README.md`
3. `Cybersecurity/README.md`
4. `Documentation/README.md`
5. `Networking/README.md`
6. `Systems-Administration/README.md`
7. `IT-Support/README.md`
8. `IT-Support/Service-Desk-Incident-Management/README.md`
9. `IT-Support/Service-Desk-Incident-Management/incidents/INC-002-M365-Authentication.md`
10. `IT-Support/Service-Desk-Incident-Management/incidents/INC-003-DNS-Decision-Tree.md`
11. `IT-Support/Windows-Troubleshooting-Lab/README.md`
12. `IT-Support/Windows-Troubleshooting-Lab/evidence/validation-notes.md`
13. `IT-Support/Service-Desk-Incident-Management/incidents/INC-002-Technical-Analysis.md`
14. `IT-Support/Service-Desk-Incident-Management/incidents/INC-003-DNS-Resolution.md`
15. `IT-Support/Service-Desk-Incident-Management/incidents/INC-004-VPN-Connectivity.md`
16. `IT-Support/Service-Desk-Incident-Management/templates/incident-report-template.md`
17. `IT-Support/Service-Desk-Incident-Management/images/README.md`

## Traducciones de documentos técnicos pendientes

### Soporte TI

**Localización finalizada.** Los incidentes reales y simulados, procedimientos, formularios, flujos Mermaid y notas de validación están disponibles en español. Las salidas originales de consola permanecen sin modificaciones.

### Ciberseguridad y seguridad de la información
- `Cybersecurity/Windows-Security-Event-Analysis/README.md`
- `Cybersecurity/Windows-Security-Event-Analysis/docs/event-analysis.md`
- `Cybersecurity/Windows-Security-Event-Analysis/docs/executive-summary.md`
- `Cybersecurity/Windows-Security-Event-Analysis/docs/lab-guide.md`
- `Cybersecurity/Windows-Security-Event-Analysis/docs/monitoring-recommendations.md`
- `Cybersecurity/Windows-Security-Hardening/README.md`
- `Cybersecurity/Windows-Security-Hardening/docs/baseline-guide.md`
- `Cybersecurity/Windows-Security-Hardening/docs/security-assessment.md`
- `Cybersecurity/Windows-Security-Hardening/docs/validation-plan.md`
- `Information-Security/Enterprise-Risk-Assessment/README.md`
- `Information-Security/Enterprise-Risk-Assessment/assets/asset-inventory.md`
- `Information-Security/Enterprise-Risk-Assessment/controls/control-mapping.md`
- `Information-Security/Enterprise-Risk-Assessment/reports/executive-summary.md`
- `Information-Security/Enterprise-Risk-Assessment/risk-register/risk-register.md`
- `Information-Security/Enterprise-Risk-Assessment/treatment/risk-treatment-plan.md`

## Criterios de calidad

1. Mantener las **cifras, marcas de tiempo, comandos, salidas y limitaciones** del original.
2. Traducir la explicación y la documentación, **no falsificar ni reescribir el resultado de una consola real**.
3. Mantener todos los enlaces internos relativos y los identificadores de incidentes.
4. Identificar claramente casos **reales**, **simulados**, **propuestos**, **ejecutados** y **verificados**.
5. Mantener los nombres de tecnologías y parámetros de herramientas cuando su traducción cause confusión.
6. Verificar al finalizar los flujos Mermaid, los SVG y los enlaces entre repositorios.
7. Registrar cualquier cambio funcional o de estado por separado de la traducción.

## Próxima prioridad

Continuar con las investigaciones de eventos y el laboratorio de Windows Hardening, y posteriormente con la evaluación de riesgos de seguridad de la información. Mantener íntegros los comandos y las salidas originales.
