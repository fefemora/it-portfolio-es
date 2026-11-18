# Plan de cambios y validación — Windows Security Hardening

[Proyecto](../README.md) · [Evaluación de seguridad](security-assessment.md) · [Guía del laboratorio](baseline-guide.md)

**Estado del laboratorio:** documentación de pasos 1–30 finalizada; las mejoras futuras requieren aprobaciones independientes.  
**Único cambio implementado:** **CHG-001**, ampliación de capacidad máxima del registro Security de 20 a 128 MiB.  
**Validación:** configuración y acceso inmediato al historial verificados; **objetivo de siete días no demostrado**.  
**Cambios no realizados:** auditoría de procesos, PowerShell logging, BitLocker, Secure Boot y ajustes de protección antivirus/firewall.

## 1. Metodología de gestión de cambios

Antes de ejecutar una corrección se debe:

1. Obtener una **línea base de solo lectura** y registrar cuándo se recolectó.
2. Identificar el objetivo de seguridad y por qué se necesita el cambio.
3. Determinar propiedad del equipo y administración por dominio, MDM o políticas locales.
4. Revisar impacto en usuarios, conectividad remota, privacidad y volumen de eventos.
5. Preparar recuperación y plan de reversión específico del cambio.
6. Solicitar autorización para **una modificación acotada**, sin activar funciones no relacionadas.
7. Registrar la acción exacta y observar el estado posterior independientemente.
8. Comprobar funcionalidad y documentar desviaciones o reversión.

Estas condiciones son una guía para cambios futuros: **no se afirma que todas las actividades de aprobación empresarial hayan ocurrido** en el laboratorio personal.

## 2. CHG-001 — Ampliación del registro Security

### Problema y alcance

El registro Security tenía un máximo de **20 MiB** y modo `Circular`. Los eventos disponibles abarcaban inicialmente aproximadamente **1,93 días** en una medición inmediatamente previa. Se propuso ampliar a 128 MiB para aumentar la capacidad de historial investigable.

El propietario del equipo aprobó **solo** cambiar el límite del canal Security. No aprobó habilitar creación de procesos, alterar la retención del canal PowerShell, modificar firewall o Defender, cifrar C: ni cambiar parámetros de firmware.

### Estado anterior verificado

**Consulta ejecutada:**

```powershell
Get-WinEvent -ListLog Security |
    Select-Object LogName,
                  RecordCount,
                  MaximumSizeInBytes,
                  LogMode |
    Format-List
```

**Resultado observado:**

```text
LogName            : Security
RecordCount        : 33959
MaximumSizeInBytes : 20971520
LogMode            : Circular
```

**Nota:** la consulta previa no incluyó el campo `IsEnabled`; el canal había sido verificado como habilitado en otros pasos.

### Ejecución autorizada

```powershell
wevtutil sl Security /ms:134217728
```

El comando amplió el límite de **20 MiB (20.971.520 bytes)** a **128 MiB (134.217.728 bytes)**.

### Validación de configuración posterior

```powershell
Get-WinEvent -ListLog Security |
    Select-Object LogName,
                  IsEnabled,
                  RecordCount,
                  MaximumSizeInBytes,
                  LogMode |
    Format-List
```

**Salida real:**

```text
LogName            : Security
IsEnabled          : True
RecordCount        : 33989
MaximumSizeInBytes : 134217728
LogMode            : Circular
```

Se verificó la configuración de **128 MiB**, habilitación y modo circular. Son **108 MiB adicionales** (6,4 veces la capacidad original). Los conteos de 33.959 y 33.989 provienen de capturas diferentes, por lo que no demuestran exactamente treinta eventos nuevos ni continuidad individual.

### Intervalo de eventos anteriores al cambio

**Consulta:**

```powershell
$oldest = Get-WinEvent -LogName Security -Oldest -MaxEvents 1
$newest = Get-WinEvent -LogName Security -MaxEvents 1

[PSCustomObject]@{
    OldestEvent  = $oldest.TimeCreated
    NewestEvent  = $newest.TimeCreated
    RetainedDays = [math]::Round(
        ($newest.TimeCreated - $oldest.TimeCreated).TotalDays, 2
    )
}
```

**Salida real:**

```text
OldestEvent        NewestEvent         RetainedDays
-----------        -----------         ------------
16-11-2026 0:52:07 17-11-2026 23:06:29         1,93
```

### Comprobación inmediata posterior

```powershell
$oldest = Get-WinEvent -LogName Security -Oldest -MaxEvents 1
$newest = Get-WinEvent -LogName Security -MaxEvents 1

[PSCustomObject]@{
    OldestEvent = $oldest.TimeCreated
    NewestEvent = $newest.TimeCreated
    OldestRecordId = $oldest.RecordId
    NewestRecordId = $newest.RecordId
    RetainedDays = [math]::Round(($newest.TimeCreated - $oldest.TimeCreated).TotalDays, 2)
} | Format-List
```

**Salida real:**

```text
OldestEvent    : 16-11-2026 0:52:21
NewestEvent    : 18-11-2026 0:00:06
OldestRecordId : 2256643
NewestRecordId : 2291047
RetainedDays   : 1,96
```

**Conclusión:** siguieron siendo accesibles eventos con fecha anterior a CHG-001 y se confirmó el límite configurado. El intervalo de **1,96 días** no valida el objetivo de siete días y tampoco acredita ausencia de huecos en los identificadores de registro. No se ejecutó una reversión ni una operación de borrado de eventos durante el procedimiento documentado.

### Estado de CHG-001

| Campo | Resultado |
|---|---|
| Cambio aprobado | Solo tamaño máximo Security: 20 → 128 MiB |
| Acción ejecutada | `wevtutil sl Security /ms:134217728` |
| Capacidad posterior | **128 MiB**, verificada |
| Canal habilitado / retención | `True` / `Circular`, verificados |
| Intervalo anterior | **1,93 días** observados |
| Intervalo posterior inmediato | **1,96 días** observados |
| Objetivo de siete días | **Pendiente de verificación** |
| Rollback | No realizado |
| Resultado | **Implementado y configuración validada; beneficio temporal a largo plazo pendiente** |

## 3. Propuestas de mejora no implementadas

| Candidato | Objetivo | Validación necesaria | Riesgos o requisitos |
|---|---|---|---|
| Evaluación de firewall | Comprobar protección de perfiles y reglas | Estado efectivo y pruebas de conectividad apropiadas | Interrupción de servicios remotos |
| Defender | Comprobar continuidad de protección y firmas | Nueva captura de estado y preferencias | Políticas externas y coexistencia de productos |
| Auditoría de procesos | Ampliar visibilidad de eventos 4688 | Antes/después de `auditpol`, generación de eventos, volumen | Privacidad, comandos sensibles, almacenamiento |
| Logging de PowerShell | Definir cobertura necesaria de Script Block/Module/Transcription | Estado efectivo, eventos, acceso y retención | Volumen, confidencialidad, políticas de organización |
| BitLocker | Proteger C: en reposo | Cifrado y protección posterior, recuperación probada | Pérdida de acceso por claves o problemas de arranque |
| Secure Boot | Protección del proceso de inicio | Resultado posterior y arranque funcional | Compatibilidad de firmware, claves, Hyper-V, recuperación |
| Retención Security | Alcanzar objetivo de siete días | Varias mediciones separadas temporalmente | Variabilidad de eventos y sobrescritura circular |

**No se habilitó ni se simuló ninguna de estas mejoras como cambio productivo adicional.**

## 4. Auditoría de creación de procesos — Propuesta separada

El laboratorio verificó con `auditpol`:

```text
Directiva de auditoría del sistema
Categoría o subcategoría                  Configuración
Seguimiento detallado
  Creación del proceso                    Sin auditoría
```

Esto puede reducir la visibilidad de eventos futuros **4688**, pero no demuestra un incidente. Un piloto futuro debería establecer alcance, privacidad, volumen, autorización, rollback y comprobaciones antes/después. **No se ejecutó un cambio de auditoría** en este proyecto.

## 5. BitLocker y Secure Boot — Cambios diferidos

### Línea base comprobada

| Componente | Resultado |
|---|---|
| C: `VolumeStatus` | `FullyDecrypted` |
| C: `ProtectionStatus` | `Off` |
| C: `EncryptionPercentage` | `0` |
| TPM: Present, Ready, Enabled, Activated | Todos `True` |
| Firmware | `Uefi` |
| Disco del sistema | Disco 0, `GPT`, atributos `IsBoot=True`, `IsSystem=True` del objeto disco |
| Secure Boot | `False` |

No se dispone todavía de los requisitos de respaldo y recuperación confirmados para proceder. Cualquier cambio requiere:

1. Crear y verificar un respaldo independiente del volumen.
2. Preparar un medio de recuperación y comprobar su uso.
3. Planificar almacenamiento seguro **fuera de GitHub** para la clave de recuperación BitLocker.
4. Confirmar edición Windows, firmware, arranque firmado y compatibilidad de Hyper-V.
5. Autorizar por separado el cambio en el firmware o el cifrado.
6. Validar el arranque, los servicios y recuperación; registrar desviaciones.

**No se modificaron BitLocker, TPM, Secure Boot, particiones ni firmware.**

## 6. Telemetría PowerShell — Recomendaciones pendientes

Se observó el canal operativo habilitado, con **808 registros**, **15 MiB** de máximo circular y **5,05 días de intervalo** entre fechas extremas.

Las claves de política `ScriptBlockLogging`, `ModuleLogging` y `Transcription` no existían en las rutas HKLM y HKCU específicas consultadas. Hubo **438** eventos 4104 en una consulta y **439 advertencias** en otra; no se recuperaron registros 4103 bajo una consulta que suprimió errores no terminantes.

Esto no permite establecer cobertura total o ausencia de registros. Una implementación posterior deberá definir objetivos de detección, precedencia de políticas, impacto en privacidad y retención antes de autorizar cambios. **No se modificó el registro de PowerShell**.

## 7. Autenticación y privilegios — Línea base sin cambios

La subcategoría Logon estaba configurada en `Aciertos y errores`. En una medición de 24 horas se encontraron **235 registros 4624**, sin coincidencias 4625, y distribución **229 tipo 5 (servicios), 4 tipo 11 (caché), 2 tipo 7 (desbloqueo)**. Una consulta independiente encontró **233 registros 4672**.

Estas cifras no identifican personas, incidentes ni correlación causal. No se cambiaron permisos ni políticas de autenticación.

## 8. Estado final y criterios de seguimiento

**Fase del proyecto:** completa como evaluación de línea base y gestión de un único cambio. **No equivale a endpoint totalmente endurecido ni a conformidad certificada.**

Próximas condiciones de seguimiento:

- Repetir mediciones de retención Security tras tiempo suficiente y comparar contra siete días.
- Mantener propuestas separadas para auditoría de procesos y PowerShell logging.
- Preparar respaldos y recuperación antes de cualquier acción sobre BitLocker o Secure Boot.
- Evitar publicar cuentas, identificadores privados, contraseñas o claves de recuperación.
- Documentar cada autorización, ejecución, validación y posible reversión como un cambio distinto.

## Referencias oficiales

- [Security Compliance Toolkit — Microsoft Learn](https://learn.microsoft.com/en-us/windows/security/operating-system-security/device-management/windows-security-configuration-framework/security-compliance-toolkit-10)
- [Windows Security Baselines](https://learn.microsoft.com/en-us/windows/security/operating-system-security/device-management/windows-security-configuration-framework/windows-security-baselines)
- [Audit Process Creation](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/audit-process-creation)

*Los marcos y las guías oficiales apoyan las recomendaciones; los cambios y validaciones declarados proceden exclusivamente de la evidencia real del laboratorio.*