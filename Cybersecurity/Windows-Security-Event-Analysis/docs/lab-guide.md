# Guía práctica: análisis de eventos de seguridad de Windows con PowerShell

[Proyecto](../README.md) · [Resumen ejecutivo](executive-summary.md) · [Análisis técnico](event-analysis.md) · [Recomendaciones](monitoring-recommendations.md) · [Portafolio](../../../README.md)

**Estado:** comprobaciones ejecutadas y validadas parcialmente con resultados de consola reales.  
**Enfoque:** consultas de solo lectura. No se modificaron políticas, borraron eventos ni simularon ataques.

## Requisitos

- Windows PowerShell, preferentemente **como administrador** para acceder a Security.
- Registro de eventos Security existente.
- No publicar nombres reales de usuarios, SID personales, direcciones IP, nombres de equipos ni mensajes completos sin revisión.

## Paso 1 — Revisar configuración del registro Security

Ejecutar en **una sola línea** para evitar errores al pegar:

```powershell
Get-WinEvent -ListLog Security | Select-Object LogName, IsEnabled, RecordCount, MaximumSizeInBytes, LogMode
```

**Objetivo:** verificar habilitación, cantidad de registros, límite de tamaño y modo de retención.

**Salida real (verificada después de repetir la consulta):**

```text
LogName            : Security
IsEnabled          : True
RecordCount        : 33898
MaximumSizeInBytes : 20971520
LogMode            : Circular
```

**Interpretación:** el registro estaba habilitado y contenía **33.898 eventos**, con tamaño máximo de **20 MiB**. El modo `Circular` permite sobrescribir registros antiguos cuando alcanza su capacidad.

**Problema resuelto durante la consulta:** un primer pegado dejó `Select-Object` antes de `Get-WinEvent`, produciendo `ParserError: EmptyPipeElement`. Fue un error de introducción del comando, **no una falla del registro Security**.

## Paso 2 — Contar eventos Security de los últimos siete días

```powershell
$ids = 4624,4625,4634,4672,4688,1102

Get-WinEvent -FilterHashtable @{
    LogName   = 'Security'
    Id        = $ids
    StartTime = (Get-Date).AddDays(-7)
} -ErrorAction SilentlyContinue |
    Group-Object Id |
    Sort-Object Name |
    Select-Object Name, Count
```

**Objetivo:** recuperar coincidencias dentro de una ventana móvil de siete días y agrupar por identificador.

**Resultado real:**

```text
Name Count
---- -----
4624   638
4634    37
4672   614
4688    13
```

**Validación:** cuatro identificadores tuvieron registros; 4625 y 1102 no aparecieron. Esto **no demuestra** que nunca hayan ocurrido. La auditoría, retención, permisos, alcance y `-ErrorAction SilentlyContinue` pueden influir en la recuperación.

**Significado:** 4624 = inicio de sesión exitoso; 4634 = cierre de sesión; 4672 = privilegios especiales asignados; 4688 = creación de proceso. Los conteos no representan personas únicas ni pruebas de comportamiento malicioso.

## Paso 3 — Medir intervalo de eventos retenidos

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

**Objetivo:** calcular el tiempo entre el evento más antiguo y el más reciente todavía retenidos.

**Salida real en formato local:**

```text
OldestEvent         NewestEvent         RetainedDays
-----------         -----------         ------------
15-11-2026 20:02:39 17-11-2026 20:40:43         2,03
```

**Interpretación:** el historial observado abarcaba **2,03 días**, entre el 15 y el 17 de noviembre de 2026. La consulta del paso anterior, aunque solicitó siete días, no podía recuperar siete días completos de datos bajo esa medición posterior. La retención circular **podría** sobrescribir eventos, pero esta medición no demuestra por sí sola que haya ocurrido ni que la grabación haya sido continua.

**Limitación:** los conteos y las fechas provienen de consultas diferentes, posiblemente tomadas en momentos distintos.

## Paso 4 — Clasificar una muestra acotada de sesiones exitosas

**Primer intento:** una consulta sin límite procesaba todo el XML de cada 4624 disponible y no terminó rápidamente. Se sustituyó por una consulta de **100 registros**, sin cambios de configuración.

**Comando ejecutado con éxito:**

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4624
} -MaxEvents 100 |
ForEach-Object {
    $xml = [xml]$_.ToXml()
    ($xml.Event.EventData.Data |
        Where-Object Name -eq 'LogonType').'#text'
} |
Group-Object |
Sort-Object Count -Descending |
Select-Object Name, Count
```

**Objetivo:** contar únicamente los tipos de inicio de sesión de los 100 registros 4624 más recientes, sin mostrar usuarios, IP ni mensajes.

**Resultado:**

```text
Name Count
---- -----
5      100
```

**Interpretación:** los 100 registros fueron **tipo 5 (Service)**. Puede tratarse de actividad normal; no se confirmó acceso malicioso.

**Límite de muestreo:** son los 100 registros más recientes, no una muestra aleatoria ni todos los 638 de la consulta anterior. No corresponde extrapolarlo a todos los accesos.

## Paso 5 — Clasificar cuentas en una muestra de cinco eventos

Un intento de agrupación sobre 100 registros no respondió rápidamente; para reducir carga se limitó a **cinco**.

**Comando de una línea realmente ejecutado:**

```powershell
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4624} -MaxEvents 5 | ForEach-Object { $e = [xml]$_.ToXml(); $d = @{}; foreach ($x in $e.Event.EventData.Data) { $d[$x.Name] = $x.'#text' }; [PSCustomObject]@{ LogonType=$d['LogonType']; AccountCategory=$(switch ($d['TargetUserSid']) { 'S-1-5-18' {'SYSTEM'; break} 'S-1-5-19' {'LOCAL SERVICE'; break} 'S-1-5-20' {'NETWORK SERVICE'; break} default {'Other'} }) } }
```

**Objetivo:** clasificar el SID de destino en categorías integradas sin revelar identificadores personales.

**Resultado:**

```text
LogonType AccountCategory
--------- ---------------
5         SYSTEM
5         SYSTEM
5         SYSTEM
5         SYSTEM
5         SYSTEM
```

**Interpretación:** los cinco eventos eran **tipo 5 / LocalSystem (S-1-5-18)**. Es compatible con servicios legítimos, pero no identifica el proceso responsable ni la categoría de las restantes sesiones.

La consulta completa anterior pudo ser lenta por múltiples razones; el resultado acotado no permite determinar su causa exacta.

## Paso 6 — Revisar política avanzada de auditoría

```powershell
auditpol /get /category:*
```

**Objetivo:** consultar la política efectiva de auditoría, sin modificarla.

**Extracto observado, presentado en español:**

| Subcategoría | Configuración |
|---|---|
| Inicio de sesión | Aciertos y errores |
| Cierre de sesión | Aciertos |
| Inicio de sesión especial | Aciertos |
| Bloqueo de cuentas | Aciertos |
| Creación de procesos | Sin auditoría |
| Terminación de procesos | Sin auditoría |
| Sistema de archivos | Sin auditoría |
| Registro | Sin auditoría |
| Cambio de directiva de auditoría | Aciertos |
| Administración de cuentas de usuario | Aciertos |
| Administración de grupos de seguridad | Aciertos |
| Integridad del sistema | Aciertos y errores |

**Interpretación:** existe auditoría configurada para inicios de sesión exitosos y fallidos y para accesos especiales exitosos. Que 4625 no apareciera en una consulta anterior no invalida esa configuración.

**Hallazgo pendiente:** la primera consulta recuperó **13 eventos 4688**, pese a que creación de procesos estaba configurada como **Sin auditoría** en el momento de revisar. Podría deberse a configuraciones históricas, pero no se confirmó. Se requiere revisar fechas y evidencia de cambios.

**Consideración de seguridad:** auditar procesos, Registro y sistema de archivos puede incrementar considerablemente el volumen de eventos y mostrar información sensible. No se habilitaron subcategorías.

## Paso 7 — Revisar fechas de cinco eventos 4688

```powershell
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4688} -MaxEvents 5 |
    Select-Object TimeCreated, Id, RecordId
```

**Objetivo:** recuperar solo fechas e identificadores, sin rutas de ejecutables, comandos ni usuarios.

**Salida real:**

```text
TimeCreated           Id RecordId
-----------           -- --------
16-11-2026 14:30:43 4688  2266586
16-11-2026 14:30:43 4688  2266585
16-11-2026 14:30:43 4688  2266584
16-11-2026 14:30:43 4688  2266583
16-11-2026 14:30:43 4688  2266582
```

**Interpretación:** los cinco eventos comparten el mismo segundo, **16/11/2026 14:30:43**, y tienen IDs consecutivos. Esto es compatible con un conjunto de eventos generados rápidamente, pero se desconocen los procesos y la política existente en esa fecha.

**Estado:** discrepancia con la política actual de creación de procesos todavía **sin resolver**; un cambio previo es solo una hipótesis.

## Paso 8 — Buscar cambios de política de auditoría

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4719
} -MaxEvents 5 |
Select-Object TimeCreated, Id, RecordId
```

**Objetivo:** buscar hasta cinco eventos 4719 de cambio en la política de auditoría del sistema.

**Resultado:** sin registros; PowerShell informó `NoMatchingEventsFound` (`ObjectNotFound`).

**Interpretación:** no se recuperaron 4719 dentro del historial disponible. Esto **no prueba que nunca se cambiara la política**, especialmente con una retención circular de aproximadamente 2,03 días observados. La discrepancia 4688 permanece abierta.

## Paso 9 — Obtener metadatos de accesos especiales 4672

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4672
} -MaxEvents 5 |
Select-Object TimeCreated, Id, RecordId
```

**Objetivo:** recopilar un conjunto pequeño de metadatos sin mostrar cuentas, IP o privilegios.

**Salida observada:**

```text
TimeCreated           Id RecordId
-----------           -- --------
17-11-2026 21:00:46 4672  2286080
17-11-2026 20:57:13 4672  2286068
17-11-2026 20:54:58 4672  2285921
17-11-2026 20:53:51 4672  2285919
17-11-2026 20:44:56 4672  2285681
```

**Interpretación:** cinco registros 4672 recuperados del 17/11/2026, entre 20:44:56 y 21:00:46. La asignación de privilegios especiales **no demuestra** actividad no autorizada. No se puede identificar cuentas distintas ni privilegios concretos a partir de estos metadatos.

## Paso 10 — Revisar privilegios de tres eventos 4672

```powershell
$events = Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4672
} -MaxEvents 3

$results = foreach ($event in $events) {
    $xml = [xml]$event.ToXml()
    $data = @{}

    foreach ($item in $xml.Event.EventData.Data) {
        $data[$item.Name] = $item.'#text'
    }

    $category = switch ($data['SubjectUserSid']) {
        'S-1-5-18' { 'LocalSystem'; break }
        'S-1-5-19' { 'LocalService'; break }
        'S-1-5-20' { 'NetworkService'; break }
        default { 'Other' }
    }

    [PSCustomObject]@{
        TimeCreated = $event.TimeCreated
        AccountCategory = $category
        Privileges = $data['PrivilegeList']
    }
}

$results | Format-List
```

**Objetivo:** clasificar los SID integrados y nombres de privilegios en tres registros recientes. No muestra nombres personales ni identificadores de sesiones.

**Resultados:**

| Fecha/hora local | Cuenta | Privilegios |
|---|---|---:|
| 2026-11-17 21:05:00 | LocalSystem | 12 |
| 2026-11-17 21:00:46 | LocalSystem | 12 |
| 2026-11-17 20:57:13 | LocalSystem | 12 |

Los tres incluían exactamente:

- `SeAssignPrimaryTokenPrivilege`
- `SeTcbPrivilege`
- `SeSecurityPrivilege`
- `SeTakeOwnershipPrivilege`
- `SeLoadDriverPrivilege`
- `SeBackupPrivilege`
- `SeRestorePrivilege`
- `SeDebugPrivilege`
- `SeAuditPrivilege`
- `SeSystemEnvironmentPrivilege`
- `SeImpersonatePrivilege`
- `SeDelegateSessionUserImpersonatePrivilege`

**Interpretación:** privilegios típicos de contextos de Windows altamente privilegiados. El evento 4672 indica asignación **al iniciar la sesión**, no que hayan sido utilizados ni que exista abuso.

Son tres registros de otra captura; la presencia de uno a las 21:05:00 confirma que el registro siguió recibiendo eventos respecto de la muestra anterior. No se identificó proceso ni se estableció causalidad.

## Paso 11 — Correlacionar eventos 4672 y 4624

```powershell
$special = Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4672
} -MaxEvents 1

$xml = [xml]$special.ToXml()
$logonId = ($xml.Event.EventData.Data |
    Where-Object Name -eq 'SubjectLogonId').'#text'

$start = $special.TimeCreated.AddMinutes(-2)
$end = $special.TimeCreated.AddMinutes(1)

$matches = @(Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4624
    StartTime = $start
    EndTime = $end
} -MaxEvents 100 | Where-Object {
    $e = [xml]$_.ToXml()
    ($e.Event.EventData.Data |
        Where-Object Name -eq 'TargetLogonId').'#text' -eq $logonId
})

[PSCustomObject]@{
    Event4672Time = $special.TimeCreated
    Matching4624Events = $matches.Count
    CorrelationFound = ($matches.Count -gt 0)
}
```

**Objetivo:** comparar `SubjectLogonId` de un 4672 seleccionado con `TargetLogonId` de 4624 dentro de una ventana de dos minutos antes y un minuto después, sin mostrar identificadores reales de sesión.

**Salida real:**

```text
Event4672Time       Matching4624Events CorrelationFound
-------------       ------------------ ----------------
17-11-2026 21:08:51                  1             True
```

**Interpretación:** se recuperó una coincidencia 4624 para el 4672 consultado. Esto apoya que ambos eventos se relacionan con una sesión, **no demuestra** qué persona intervino, qué privilegios utilizó ni actividad maliciosa.

**Límites:** un único 4672 y hasta 100 candidatos 4624; no se puede generalizar. El Logon ID se comparó localmente, sin publicarlo.

## Paso 12 — Clasificar una sesión correlacionada

**Objetivo:** seleccionar el 4672 más reciente en una nueva ejecución, buscar su 4624 correspondiente y obtener solo tipo de sesión y categoría integrada de cuenta.

**Comando de solo lectura reproducible:**

```powershell
$special = Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4672
} -MaxEvents 1

$s = [xml]$special.ToXml()
$logonId = ($s.Event.EventData.Data |
    Where-Object Name -eq 'SubjectLogonId').'#text'

$start = $special.TimeCreated.AddMinutes(-2)
$end = $special.TimeCreated.AddMinutes(1)

$match = Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4624
    StartTime = $start
    EndTime = $end
} -MaxEvents 100 | Where-Object {
    $x = [xml]$_.ToXml()
    ($x.Event.EventData.Data |
        Where-Object Name -eq 'TargetLogonId').'#text' -eq $logonId
} | Select-Object -First 1

if ($match) {
    $x = [xml]$match.ToXml()
    $d = @{}
    foreach ($item in $x.Event.EventData.Data) {
        $d[$item.Name] = $item.'#text'
    }

    $category = switch ($d['TargetUserSid']) {
        'S-1-5-18' { 'LocalSystem'; break }
        'S-1-5-19' { 'LocalService'; break }
        'S-1-5-20' { 'NetworkService'; break }
        default { 'Other' }
    }

    [PSCustomObject]@{
        MatchFound = $true
        LogonType = $d['LogonType']
        AccountCategory = $category
    }
} else {
    [PSCustomObject]@{
        MatchFound = $false
        LogonType = 'N/A'
        AccountCategory = 'N/A'
    }
}
```

**Incidente menor durante el proceso:** una comprobación de la variable `$match` devolvió `No matching event stored. Rerun Step 12.` porque en esa sesión no estaba almacenada una coincidencia. La consulta completa, ejecutada después, terminó correctamente.

**Salida real:**

```text
MatchFound LogonType AccountCategory
---------- --------- ---------------
      True 5         LocalSystem
```

**Hallazgo:** un evento 4624 correlacionado con 4672 correspondió a **Logon Type 5 (Service)** y cuenta integrada **LocalSystem**.

**Distinción esencial:** esta ejecución seleccionó el **4672 más reciente en ese momento**, sin fijar ID de registro. Como la salida no incluyó la marca de tiempo, **no podemos afirmar** que fuera el evento anterior de las 21:08:51. No demuestra uso de privilegios ni actividad maliciosa.

## Informe del caso — Completado

El [resumen ejecutivo](executive-summary.md) sintetiza la evidencia, discrepancia 4688, retención y límites de muestras. Las [recomendaciones de monitoreo](monitoring-recommendations.md) siguen siendo propuestas. Ninguna configuración se modificó.

## Reglas de documentación

- Diferenciar observaciones reales, resultados esperados e hipótesis.
- Para cada paso, conservar comando exacto, objetivo, salida real, interpretación y limitaciones.
- No inventar capturas, horas de consulta, alertas o conclusiones.

## Referencias oficiales

- [auditpol /get](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/auditpol-get)
- [Audit Process Creation](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/audit-process-creation)
- [Get-WinEvent](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.diagnostics/get-winevent)
- [Evento 4624](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4624)
- [Evento 4634](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4634)
- [Evento 4672](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4672)
- [SID conocidos de Windows](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-identifiers)
- [Evento 4719](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4719)
- [Evento 4688](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4688)

*Todos los resultados documentados proceden de las consultas reales, no de las referencias externas.*