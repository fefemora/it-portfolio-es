# Estado de localización — Portafolio en español

**Repositorio de referencia:** [it-portfolio-en](https://github.com/fefemora/it-portfolio-en)  
**Repositorio de destino:** [it-portfolio-es](https://github.com/fefemora/it-portfolio-es)

## Resumen de migración

- **47/47 archivos originales replicados:** se conservan estructura, rutas, documentos fuente, archivos de evidencia y diagramas.
- **11/11 archivos SVG adaptados:** se tradujeron sus etiquetas y textos principales conservando los elementos gráficos y conexiones. **Revisión visual de desbordamiento y legibilidad pendiente.**
- **32/32 archivos Markdown originales localizados al español:** portada, áreas generales, todos los incidentes de Soporte TI, la evaluación de riesgos, el análisis de eventos de Windows y los 30 pasos de Windows Hardening. Los cuatro documentos extensos del último laboratorio fueron reorganizados para facilitar la consulta, preservando comandos, resultados y hallazgos verificables.
- **0/32 documentos Markdown originales pendientes:** localización documental completada. Queda una revisión visual manual de los diagramas y, si corresponde, ajustes editoriales menores.
- **Tres archivos .txt de evidencia:** se mantienen exactamente como los registros originales, sin alterar la salida de los comandos ni los datos observados.
- **LICENSE:** se conserva el texto legal original.

**Importante:** la traducción documental está completa, pero la comprobación visual de todos los SVG todavía está pendiente. Los comandos, datos de consola y hallazgos reales se mantuvieron sin inventar resultados. La organización y extensión de las explicaciones en español pueden diferir de las del original, sin cambiar la evidencia.

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
18. `Cybersecurity/Windows-Security-Event-Analysis/README.md`
19. `Information-Security/Enterprise-Risk-Assessment/README.md`
20. `Cybersecurity/Windows-Security-Event-Analysis/docs/executive-summary.md`
21. `Cybersecurity/Windows-Security-Event-Analysis/docs/monitoring-recommendations.md`
22. `Information-Security/Enterprise-Risk-Assessment/assets/asset-inventory.md`
23. `Information-Security/Enterprise-Risk-Assessment/risk-register/risk-register.md`
24. `Information-Security/Enterprise-Risk-Assessment/controls/control-mapping.md`
25. `Information-Security/Enterprise-Risk-Assessment/reports/executive-summary.md`
26. `Information-Security/Enterprise-Risk-Assessment/treatment/risk-treatment-plan.md`
27. `Cybersecurity/Windows-Security-Event-Analysis/docs/event-analysis.md`
28. `Cybersecurity/Windows-Security-Event-Analysis/docs/lab-guide.md`
29. `Cybersecurity/Windows-Security-Hardening/README.md`
30. `Cybersecurity/Windows-Security-Hardening/docs/baseline-guide.md`
31. `Cybersecurity/Windows-Security-Hardening/docs/security-assessment.md`
32. `Cybersecurity/Windows-Security-Hardening/docs/validation-plan.md`

## Estado de localización por área

### Soporte TI

**Localización finalizada.** Los incidentes reales y simulados, procedimientos, formularios, flujos Mermaid y notas de validación están disponibles en español. Las salidas originales de consola permanecen sin modificaciones.

### Ciberseguridad y seguridad de la información

**Localización documental finalizada.** Los informes de análisis de eventos, evaluación de riesgos empresariales y los cuatro archivos de Windows Security Hardening están en español. El laboratorio de hardening conserva una guía central con los 30 pasos, comandos y salidas, más documentos resumidos de evaluación y cambio para evitar duplicaciones.

## Documentación adicional en español

Se agregaron [Cómo trabajo — Metodología](metodologia/README.md), una [plantilla de investigación](metodologia/plantilla-investigacion.md), los [criterios de evidencia](metodologia/evidencia-y-validacion.md) y un [resumen técnico del laboratorio Windows Security Hardening](Cybersecurity/Windows-Security-Hardening/RESUMEN-ES.md). Estos archivos no formaban parte del conjunto de 32 Markdown originales.

## Criterios de calidad

1. Mantener las **cifras, marcas de tiempo, comandos, salidas y limitaciones** del original.
2. Traducir la explicación y la documentación, **no falsificar ni reescribir el resultado de una consola real**.
3. Mantener todos los enlaces internos relativos y los identificadores de incidentes.
4. Identificar claramente casos **reales**, **simulados**, **propuestos**, **ejecutados** y **verificados**.
5. Mantener los nombres de tecnologías y parámetros de herramientas cuando su traducción cause confusión.
6. Verificar al finalizar los flujos Mermaid, los SVG y los enlaces entre repositorios.
7. Registrar cualquier cambio funcional o de estado por separado de la traducción.

## Próxima prioridad

Realizar una **revisión visual manual de los 11 SVG** y verificar su legibilidad a distintos tamaños; continuar con mantenimiento bilingüe y referencias cruzadas. Mantener íntegros los comandos, salidas y límites de las evidencias originales.
