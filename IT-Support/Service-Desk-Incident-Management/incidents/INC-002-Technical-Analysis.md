# INC-002 — Análisis técnico de autenticación

[← Caso INC-002](INC-002-M365-Authentication.md) · [Mesa de ayuda](../README.md) · [Soporte TI](../../README.md)

**Incidente:** INC-002 — Error de autenticación en Microsoft 365  
**Tecnología:** Microsoft Entra ID / Microsoft 365  
**Nivel de soporte:** Mesa de ayuda L2  
**Tipo de escenario:** Simulado

## 1. Objetivo de la investigación

Identificar posibles causas de un fallo de autenticación de Microsoft 365 después del reemplazo de un teléfono móvil, preservando los controles de seguridad de la organización.

## 2. Registros de inicio de sesión en Microsoft Entra

El técnico consultaría los intentos de autenticación del usuario afectado en Microsoft Entra ID.

**Ruta:** Centro de administración de Microsoft Entra → Entra ID → Monitoring & health → Sign-in logs.

Información relevante:

- Fecha y hora de autenticación.
- Aplicación y recurso.
- Estado del inicio de sesión.
- Código de error y motivo del fallo.
- Detalles de autenticación.
- Evaluación de Conditional Access.
- Datos del dispositivo.
- IP de origen y ubicación.

El acceso depende de los permisos administrativos, del licenciamiento y de la configuración de retención aplicable.

## 3. Códigos de error relevantes

| Código | Descripción | Investigación recomendada |
|---|---|---|
| AADSTS50076 | Se requiere autenticación multifactor | Revisar el desafío MFA y los métodos disponibles |
| AADSTS50079 | Se requiere registrar MFA | Revisar los requisitos de inscripción |
| AADSTS50053 | Cuenta o inicio de sesión bloqueado por una condición de seguridad | Investigar bloqueos, restricciones y eventos relacionados |
| AADSTS53003 | Bloqueo por Conditional Access | Revisar la política aplicada y los detalles del inicio de sesión |

**Un código por sí solo no demuestra la causa raíz.**

## 4. Proceso de investigación simulado

### Paso 1 — Confirmar identidad

Verificar la identidad mediante un procedimiento aprobado **antes** de realizar operaciones de recuperación de cuenta.

### Paso 2 — Revisar la cuenta

Comprobar si la cuenta está habilitada y si existen restricciones relevantes.

### Paso 3 — Analizar los intentos de acceso

Localizar el intento fallido y revisar el motivo, los detalles de autenticación y el resultado de Conditional Access.

### Paso 4 — Evaluar el registro MFA

Determinar si el empleado conserva acceso a un método válido. Cambiar de teléfono **no implica necesariamente** que todos los métodos previos hayan dejado de funcionar.

### Paso 5 — Elegir recuperación autorizada

Si la evidencia respalda una falla de registro MFA, seguir el proceso aprobado. Según la configuración y los permisos, podría utilizarse un **Temporary Access Pass (TAP)** u otra recuperación autorizada.

### Paso 6 — Validar acceso

Después de la recuperación autorizada, confirmar la autenticación y el acceso a las aplicaciones afectadas.

## 5. Matriz de decisiones

| Hallazgo | Posible causa | Acción siguiente |
|---|---|---|
| No se completa el desafío MFA | Método registrado inaccesible | Verificar identidad y evaluar recuperación autorizada |
| Se solicita inscripción MFA | Registro ausente o incompleto | Revisar requisitos e inscripción |
| Conditional Access bloquea el acceso | Incumplimiento de política | Revisar evaluación y escalar cambios |
| Cuenta deshabilitada o restringida | Administración de cuentas o acción de seguridad | Comprobar estado y escalar |
| Múltiples intentos sospechosos | Posible incidente | Escalar al equipo de Seguridad |

## 6. Consideraciones de seguridad

- No solicitar ni registrar contraseñas.
- No deshabilitar MFA para facilitar diagnóstico.
- No eludir Conditional Access sin autorización.
- Verificar identidad antes de modificar métodos MFA.
- Registrar cambios administrativos en el ticket.
- Escalar actividad sospechosa.

## 7. Evidencia necesaria en un incidente real

En un entorno autorizado, se documentarían fechas y correlaciones de inicio de sesión, códigos de error, hallazgos MFA/Conditional Access, acciones de recuperación aprobadas, resultados de validación y confirmación del usuario.

Los identificadores y datos sensibles deben ocultarse antes de publicar evidencia.

## 8. Conclusión técnica

El escenario simulado es compatible con una posible dificultad de registro o disponibilidad de MFA después de cambiar de teléfono. **No se revisaron registros reales de Microsoft Entra.**

**Causa raíz:** no confirmada.  
**Corrección:** propuesta; no ejecutada.  
**Validación:** no realizada.

El caso demuestra un proceso L2 estructurado y una recuperación consciente de la seguridad, **no una solución productiva verificada**.

## Referencias

- [Registros de inicio de sesión de Entra](https://learn.microsoft.com/en-us/entra/identity/monitoring-health/concept-sign-ins)
- [Métodos de autenticación](https://learn.microsoft.com/en-us/entra/identity/authentication/concept-authentication-methods-manage)
- [Códigos de error de Entra](https://learn.microsoft.com/en-us/entra/identity-platform/reference-error-codes)

*Estas referencias respaldan el método y no prueban acciones ejecutadas.*