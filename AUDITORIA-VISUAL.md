# Auditoría de recursos visuales y localización técnica

**Repositorio:** [Portafolio en español](README.md)  
**Fecha de revisión:** 18 de noviembre de 2026  
**Alcance:** 11 SVG del repositorio, flujos Mermaid documentados y 37 archivos Markdown (32 originales y cinco documentos adicionales) para enlaces relativos y delimitadores de código.

## Resultado y grado de verificación

**Revisión de código fuente y maquetación declarada: completada.** Los 11 SVG se inspeccionaron mediante sus nodos de texto, atributos, dimensiones, etiquetas y composición declarada. Se aplicaron cambios para reducir riesgo de desbordamiento y mejorar el idioma y la accesibilidad.

**Límite explícito:** no se completó una inspección de los **11 SVG rasterizados en un navegador** a resoluciones de escritorio y móvil. Por ello, **no se certifica ausencia absoluta de texto cortado, sustitución de tipografías ni diferencias de renderizado entre navegadores**. Esa comprobación visual final queda pendiente.

## Inventario de los 11 SVG examinados

| Archivo | Revisión de código y etiquetas | Cambios / observaciones |
|---|---|---|
| [Mapa de navegación de soporte TI](IT-Support/it-support-navigation.svg) | Revisado | Etiqueta resumida; título y descripción accesibles; `viewBox` presente |
| [Resumen de incidentes L2](IT-Support/Service-Desk-Incident-Management/incident-overview.svg) | Revisado | Frases de simulación acortadas, contraste del ID INC-003 mejorado, `viewBox` y metadatos accesibles añadidos |
| [Recuperación de almacenamiento](IT-Support/Windows-Troubleshooting-Lab/storage-recovery.svg) | Revisado | Formato decimal español **18,6 / 63,1 GiB**, `viewBox` y descripción accesible |
| [Análisis de eventos de seguridad](Cybersecurity/Windows-Security-Event-Analysis/diagrams/security-review-overview.svg) | Revisado | Rótulos de correlación acortados para caber en panel lateral; descripción accesible existente |
| [Flujo de endurecimiento Windows](Cybersecurity/Windows-Security-Hardening/diagrams/hardening-workflow.svg) | Revisado | Etiquetas de pasos precisadas; `CHG-001` identificado como único cambio ejecutado |
| [Inventario ficticio de activos](Information-Security/Enterprise-Risk-Assessment/assets/asset-overview.svg) | Revisado | «Windows endpoints» corregido a «Equipos Windows»; descripción accesible |
| [Mapeo de controles](Information-Security/Enterprise-Risk-Assessment/controls/control-mapping-overview.svg) | Revisado | Metadatos accesibles, nombres de marcos conservados como nombres técnicos |
| [Panel ejecutivo de riesgos](Information-Security/Enterprise-Risk-Assessment/reports/executive-dashboard.svg) | Revisado | Metadatos accesibles, términos y datos del caso ficticio revisados |
| [Distribución de riesgos](Information-Security/Enterprise-Risk-Assessment/reports/risk-distribution.svg) | Revisado | `viewBox` y metadatos accesibles añadidos; conteos ficticios conservados |
| [Mapa de calor](Information-Security/Enterprise-Risk-Assessment/risk-register/risk-heatmap.svg) | Revisado | «R-003 to R-006» corregido a «R-003 a R-006»; metadatos accesibles |
| [Hoja de ruta de 90 días](Information-Security/Enterprise-Risk-Assessment/treatment/risk-treatment-roadmap.svg) | Revisado | Metadatos accesibles y terminología del plan verificados |

## Verificaciones técnicas realizadas

| Criterio | Resultado |
|---|---|
| Inventario de SVG | **11 / 11** revisados |
| Namespace SVG | **11 / 11** presentes |
| Escalado mediante `viewBox` | **11 / 11** presentes |
| Títulos y descripciones en español | **11 / 11** presentes |
| `role="img"` y `aria-labelledby` | **11 / 11** presentes |
| Etiquetas `<text>` balanceadas | **11 / 11** |
| Entidades `&` no escapadas en SVG | **0** encontradas |
| Rótulos originalmente en inglés detectados | Se corrigieron «Windows endpoints», «R-003 to R-006» y decimales anglosajones |
| Panel de correlación de eventos | Se acortó el texto que excedía el ancho disponible según la inspección de coordenadas |
| Enlaces internos en Markdown | **37 / 37 archivos** comprobados; no se detectaron rutas relativas faltantes mediante comprobación estática |
| Delimitadores de código Markdown | Comprobados; no se detectaron bloques con cantidad impar de delimitadores |
| Flujos Mermaid | Revisadas las etiquetas de los bloques presentes; se encuentran en español técnico salvo nombres propios, comandos y términos aceptados |
| Renderizado final a escritorio y móvil | **Pendiente**; la inspección de fuente no sustituye una captura visual |

## Convenciones editoriales

- Traducir «Service Desk» según contexto como **mesa de ayuda**; mantener **L2**, **MFA**, **DNS**, **VPN**, **PowerShell**, **Microsoft Entra ID** y otros nombres técnicos.
- Distinguir **caso real**, **escenario simulado**, **medida propuesta**, **acción ejecutada** y **resultado verificado**.
- Mantener cifras y hallazgos del ejercicio sin transformarlos en afirmaciones de incidentes reales.
- Conservar los nombres de las aplicaciones, comandos, identificadores y archivos originales cuando funcionan como referencias técnicas.
- Emplear coma decimal en las etiquetas explicativas en español; las salidas reales de consola permanecen tal cual se obtuvieron.

## Revisión final visual todavía necesaria

Para certificar que el texto encaja exactamente según el navegador, abrir cada SVG desde los enlaces anteriores y observarlo, al menos, en una ventana de escritorio y otra estrecha de móvil. Revisar especialmente el **panel lateral de correlación**, las **tarjetas de incidentes L2**, las **cinco tarjetas del flujo de hardening** y los **pies de página de los informes de riesgos**.

Un control estático de coordenadas y estructura no determina la representación final de la tipografía; si aparece un corte, ajustar el tamaño de fuente o dividir la etiqueta en líneas y volver a verificarla antes de declarar QA visual completa.

## Estado

**Localización técnica y comprobación estructural completadas; prueba visual renderizada pendiente.** Esta distinción preserva la integridad del proyecto y evita declarar una verificación que no se ha realizado.

[Volver al portafolio](README.md) · [Registro de localización](LOCALIZACION.md)
