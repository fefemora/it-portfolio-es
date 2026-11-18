# Plantilla de investigación técnica / runbook

[Metodología](README.md) · [Criterios de evidencia](evidencia-y-validacion.md)

> **Uso:** copiar esta plantilla para casos nuevos. Los campos son una guía, **no describen un incidente real**.

## 1. Identificación

| Campo | Información |
|---|---|
| Identificador | INC-XXX / SEC-XXX / CHG-XXX |
| Tipo de caso | Real / Simulado / Planificado |
| Estado | Abierto / En investigación / Escalado / Resuelto / Documentado |
| Servicio | [Servicio o componente] |
| Prioridad | [Criterio de impacto y urgencia; sujeto al SLA aplicable] |
| Entorno y alcance | [Solo información no sensible] |

## 2. Síntoma y efecto en el negocio

**Síntoma reportado:** [Qué observa el usuario, sin inferir la causa].

**Impacto:** [Qué servicio dejó de estar disponible, personas o procesos afectados].

**Resultado deseado:** [Cómo reconoceremos la recuperación].

## 3. Evidencias y diagnósticos

| Prueba | Hipótesis que evalúa | Resultado observado | Estado |
|---|---|---|---|
| [Consulta] | [Suposición] | [Resultado real o no ejecutado] | Verificado / No probado |

Registrar comandos exactos **solo cuando se ejecutaron**; etiquetar otros como **propuestos**. No publicar salidas con nombres, cuentas, IP privadas, tokens o claves sin saneamiento.

## 4. Hipótesis

**Principal:** [Hipótesis y evidencia que la respalda].

**Alternativas:** [Otras causas plausibles].

**Causa raíz:** Confirmada / Probable / No establecida.

No convertir una hipótesis razonable en una conclusión verificada.

## 5. Acción, riesgo y autorización

**Acción propuesta:** [Cambio mínimo].

**Riesgo operativo y de seguridad:** [Disponibilidad, privacidad, accesos, efectos secundarios].

**Autorización:** [Pendiente / Aprobada / No requerida para consulta de solo lectura].

**Reversión o recuperación:** [Pasos aplicables y requisitos].

## 6. Verificación del resultado

- [ ] La función afectada opera de nuevo.
- [ ] No se observan efectos secundarios relevantes en las pruebas ejecutadas.
- [ ] El usuario o responsable confirma el resultado cuando corresponde.
- [ ] La evidencia documenta el estado anterior y posterior.
- [ ] Se registran limitaciones y riesgos residuales.

## 7. Comunicación y cierre

**Actualización para usuario:** [Breve, clara y sin suposiciones].

**Cierre técnico:** [Qué se verificó, qué permanece pendiente y si se escaló].

**Lecciones aprendidas:** [Medidas preventivas o mejoras del runbook].
