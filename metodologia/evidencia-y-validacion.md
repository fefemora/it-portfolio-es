# Calidad de evidencia, validación y control de cambios

[Metodología](README.md) · [Plantilla](plantilla-investigacion.md)

## Clasificación de las afirmaciones

| Clasificación | Uso correcto |
|---|---|
| **Verificado** | Resultado comprobado mediante una consulta o una validación real documentada. |
| **Observado** | Registro o dato capturado, sin inferir una causa por sí solo. |
| **Inferido / Hipótesis** | Posible explicación aún no confirmada. |
| **Propuesto** | Procedimiento o mejora no ejecutados. |
| **Simulado** | Escenario ficticio identificado explícitamente. |
| **Pendiente** | Resultado, validación o autorización aún no disponible. |

## Controles de calidad

1. Preservar los valores, horas, códigos de error y salidas originales de consola.
2. Mantener separadas **configuración**, **efectividad** y **cobertura real** de un control.
3. Confirmar que cada comando mostrado como ejecutado tiene su resultado observado.
4. Evitar afirmar una causa raíz sin evidencia suficiente.
5. No confundir conteos de eventos con usuarios distintos, incidentes ni actividad maliciosa.
6. Evaluar cambios mediante autorización explícita, alcance limitado, respaldo y reversión.
7. Comprobar que los enlaces, diagramas y referencias funcionan sin revelar información privada.

## Ejemplo real de distinción entre implementación y validación

En el [laboratorio Windows Security Hardening](../Cybersecurity/Windows-Security-Hardening/README.md), el cambio **CHG-001** aumentó el tamaño máximo del registro de eventos Security de **20 MiB a 128 MiB**. El nuevo valor de capacidad se verificó, pero **el objetivo de conservar siete días de historial todavía no está validado**. Son dos resultados diferentes, aunque formen parte de la misma mejora.

Del mismo modo, la observación de eventos de PowerShell 4104 o de inicios de sesión 4624 no demuestra automáticamente cobertura completa ni presencia de un incidente.

## Resguardo de la documentación

El portafolio no debe contener contraseñas, claves de recuperación, tokens, registros íntegros con datos privados, listas de usuarios reales ni rutas sensibles. Los textos de procedimientos pueden conservar la sintaxis original de los comandos; las capturas o salidas observadas deben sanearse antes de publicarse.
