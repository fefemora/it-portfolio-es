# Guía del laboratorio — Línea base de seguridad de Windows (30 pasos)

[Descripción del proyecto](../README.md) · [Matriz de evaluación](security-assessment.md) · [Gestión del cambio](validation-plan.md) · [Investigación de eventos relacionada](../../Windows-Security-Event-Analysis/README.md)

**Estado:** 28 comprobaciones y dos etapas de análisis/cierre documentadas; **solo CHG-001** fue autorizado e implementado. Los demás comandos son consultas de solo lectura. La retención de siete días no está verificada, y BitLocker y Secure Boot no fueron modificados.

**Metodología:** preservar el comando ejecutado y la salida real de consola literalmente; traducir solo su explicación. Los eventos no identifican usuarios ni incidentes por sí solos. Los resultados corresponden a instantes distintos y no necesariamente se pueden comparar directamente.

## Índice rápido

| Área | Pasos |
|---|---|
| Firewall, Defender y auditoría básica | 1–7 |
| Protección de plataforma (BitLocker, TPM, Secure Boot, UEFI, GPT) | 8–12 |
| Telemetría y políticas de PowerShell | 13–24 |
| Autenticación y privilegios especiales | 25–28 |
| Síntesis, riesgos y cierre | 29–30 |

## Paso 1 — Revisión de perfiles de Windows Firewall (verificado)

**Comando ejecutado:**

```powershell
Get-NetFirewallProfile |
    Select-Object Name, Enabled, DefaultInboundAction, DefaultOutboundAction |
    Format-Table -AutoSize
```

**Salida real:**

```text
Name    Enabled DefaultInboundAction DefaultOutboundAction
----    ------- -------------------- ---------------------
Domain     True        NotConfigured         NotConfigured
Private    True        NotConfigured         NotConfigured
Public     True        NotConfigured         NotConfigured
```

**Análisis:** Los tres perfiles indicaron `Enabled=True`. Los valores iniciales `NotConfigured` no definían el comportamiento efectivo; la segunda consulta en `ActiveStore` confirmó `Block` de entrada y `Allow` de salida en Domain, Private y Public. No se examinaron reglas individuales ni tráfico real.

### Comprobación adicional 1b — ActiveStore

```powershell
Get-NetFirewallProfile -PolicyStore ActiveStore |
    Select-Object Name, Enabled, DefaultInboundAction, DefaultOutboundAction |
    Format-Table -AutoSize
```

```text
Name    Enabled DefaultInboundAction DefaultOutboundAction
----    ------- -------------------- ---------------------
Domain     True                Block                 Allow
Private    True                Block                 Allow
Public     True                Block                 Allow
```

**Interpretación:** aquí se revisó la configuración efectiva de los perfiles. Las reglas individuales y el comportamiento del tráfico quedan fuera del alcance.

## Paso 2 — Estado de Microsoft Defender Antivirus (verificado)

**Comando ejecutado:**

```powershell
Get-MpComputerStatus |
    Select-Object AMServiceEnabled,
                  AntivirusEnabled,
                  RealTimeProtectionEnabled,
                  AntivirusSignatureAge,
                  AntivirusSignatureLastUpdated |
    Format-List
```

**Salida real:**

```text
AMServiceEnabled              : True
AntivirusEnabled              : True
RealTimeProtectionEnabled     : True
AntivirusSignatureAge         : 0
AntivirusSignatureLastUpdated : 17-11-2026 9:41:14
```

**Análisis:** El servicio antivirus, el antivirus y la protección en tiempo real estaban activos. La antigüedad de firmas era cero días y la última actualización se registró el 17/11/2026 a las 09:41:14. Es una fotografía de estado, no una prueba de detección de malware.

## Paso 3 — Configuraciones avanzadas de Microsoft Defender (verificado)

**Comando ejecutado:**

```powershell
Get-MpComputerStatus |
    Select-Object IsTamperProtected,
                  RealTimeProtectionEnabled,
                  NISEnabled |
    Format-List

Get-MpPreference |
    Select-Object MAPSReporting,
                  SubmitSamplesConsent |
    Format-List
```

**Salida real:**

```text
IsTamperProtected         : True
RealTimeProtectionEnabled : True
NISEnabled                : True

MAPSReporting        : 2
SubmitSamplesConsent : 1
```

**Análisis:** Tamper Protection y Network Inspection System informaron `True`. `MAPSReporting=2` señala participación avanzada y `SubmitSamplesConsent=1` la entrega automática de muestras seguras. No se probó conectividad cloud ni eficacia de protección.

## Paso 4 — Protección PUA y Block at First Sight (verificado)

**Comando ejecutado:**

```powershell
Get-MpPreference |
    Select-Object PUAProtection,
                  DisableBlockAtFirstSeen |
    Format-List
```

**Salida real:**

```text
PUAProtection           : 1
DisableBlockAtFirstSeen : False
```

**Análisis:** `PUAProtection=1` representa bloqueo de aplicaciones potencialmente no deseadas; `DisableBlockAtFirstSeen=False` indica que esa funcionalidad no está deshabilitada por la preferencia. No se ejecutaron pruebas de detección.

## Paso 5 — Política de auditoría de creación de procesos (verificado)

**Comando ejecutado:**

```powershell
auditpol /get /subcategory:"{0cce922b-69ae-11d9-bed3-505054503030}"
```

**Salida real:**

```text
Directiva de auditoría del sistema
Categoría o subcategoría                  Configuración
Seguimiento detallado
  Creación del proceso                    Sin auditoría
```

**Análisis:** La subcategoría `Creación del proceso` figuraba como `Sin auditoría`. Reduce la visibilidad de nuevos eventos 4688 y constituye una oportunidad de monitoreo, no un ataque confirmado. Habilitarla está pendiente de aprobación.

## Paso 6 — Capacidad y retención del registro Security (verificado)

**Comando ejecutado:**

```powershell
Get-WinEvent -ListLog Security |
    Select-Object LogName,
                  RecordCount,
                  MaximumSizeInBytes,
                  LogMode |
    Format-List
```

**Salida real:**

```text
LogName            : Security
RecordCount        : 33959
MaximumSizeInBytes : 20971520
LogMode            : Circular
```

**Análisis:** Security estaba habilitado y en modo `Circular` con límite previo de 20 MiB y 33.959 registros. Su intervalo observado antes del único cambio fue de **1,93 días**, distinto del intervalo de 2,03 días de la investigación anterior. Ninguna medición garantiza retención futura ni acredita pérdida de datos.

### Comprobación adicional 6b — Fechas de retención previas al cambio

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

```text
OldestEvent        NewestEvent         RetainedDays
-----------        -----------         ------------
16-11-2026 0:52:07 17-11-2026 23:06:29         1,93
```

La diferencia de 0,10 días frente a una medición anterior no debe interpretarse como una tasa de pérdida o sobrescritura verificada.

## Paso 7 — Espacio disponible en el volumen del sistema (verificado)

**Comando ejecutado:**

```powershell
[PSCustomObject]@{
    FreeGiB = [math]::Round((Get-PSDrive -Name C).Free / 1GB, 2)
}
```

**Salida real:**

```text
FreeGiB
-------
  56,81
```

**Análisis:** El disco C: tenía **56,81 GiB disponibles** en la consulta. Es espacio libre del volumen, no del registro Security y no garantiza capacidad futura.

## CHG-001 — Ampliación autorizada del registro Security

**Autorización:** el propietario del equipo aprobó exclusivamente el cambio de capacidad máxima de Security de **20 MiB** a **128 MiB**. No se aprobó auditoría adicional, cifrado, firmware ni cambios de firewall.

**Situación previa:** `RecordCount=33959`, `MaximumSizeInBytes=20971520`, modo `Circular`. El estado `IsEnabled` no se capturó en esa consulta previa concreta.

**Comando realmente ejecutado:**

```powershell
wevtutil sl Security /ms:134217728
```

**Comprobación posterior realmente ejecutada:**

```powershell
Get-WinEvent -ListLog Security |
    Select-Object LogName,
                  IsEnabled,
                  RecordCount,
                  MaximumSizeInBytes,
                  LogMode |
    Format-List
```

**Salida posterior:**

```text
LogName            : Security
IsEnabled          : True
RecordCount        : 33989
MaximumSizeInBytes : 134217728
LogMode            : Circular
```

**Resultado:** capacidad máxima configurada **128 MiB** (aumento de **108 MiB**, factor 6,4), canal habilitado y modo circular conservado. Los registros pasaron de 33.959 a 33.989 en consultas distintas; esto no prueba 30 eventos nuevos ni continuidad individual de todos los registros.

### Retención inmediatamente después de CHG-001

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

```text
OldestEvent    : 16-11-2026 0:52:21
NewestEvent    : 18-11-2026 0:00:06
OldestRecordId : 2256643
NewestRecordId : 2291047
RetainedDays   : 1,96
```

**Evaluación:** antes se observaron **1,93 días** y después **1,96 días** entre registros extremos; se conservó acceso a eventos con fechas anteriores al cambio. **No se demostró la retención objetivo de siete días**. No se ejecutó rollback ni se limpiaron eventos en los pasos documentados.

## Paso 8 — Estado de cifrado BitLocker en C: (verificado)

**Comando ejecutado:**

```powershell
Get-BitLockerVolume -MountPoint C: |
    Select-Object MountPoint,
                  VolumeStatus,
                  ProtectionStatus,
                  EncryptionPercentage |
    Format-List
```

**Salida real:**

```text
MountPoint           : C:
VolumeStatus         : FullyDecrypted
ProtectionStatus     : Off
EncryptionPercentage : 0
```

**Análisis:** El volumen C: estaba `FullyDecrypted`, protección `Off` y cifrado `0 %`. Existe una exposición potencial de confidencialidad ante pérdida física; no se encendió BitLocker y se exige respaldo y recuperación antes de considerar hacerlo.

## Paso 9 — Preparación del TPM (verificado)

**Comando ejecutado:**

```powershell
Get-Tpm |
    Select-Object TpmPresent,
                  TpmReady,
                  TpmEnabled,
                  TpmActivated |
    Format-List
```

**Salida real:**

```text
TpmPresent   : True
TpmReady     : True
TpmEnabled   : True
TpmActivated : True
```

**Análisis:** El TPM informó `Present`, `Ready`, `Enabled` y `Activated` en `True`. Esta preparación no demuestra que exista copia de seguridad, clave de recuperación ni compatibilidad completa de BitLocker.

## Paso 10 — Estado de Secure Boot (verificado)

**Comando ejecutado:**

```powershell
Confirm-SecureBootUEFI
```

**Salida real:**

```text
False
```

**Análisis:** `Confirm-SecureBootUEFI` devolvió `False`: Secure Boot estaba deshabilitado. No se modificó el firmware; una activación futura requiere comprobar recuperación y compatibilidad.

## Paso 11 — Modo de arranque UEFI (verificado)

**Comando ejecutado:**

```powershell
Get-ComputerInfo |
    Select-Object BiosFirmwareType |
    Format-List
```

**Salida real:**

```text
BiosFirmwareType : Uefi
```

**Análisis:** Windows indicó `BiosFirmwareType : Uefi`. Arranque UEFI no implica Secure Boot activo.

## Paso 12 — Formato del disco del sistema (verificado)

**Comando ejecutado:**

```powershell
Get-Partition -DriveLetter C |
    Get-Disk |
    Select-Object Number, PartitionStyle, IsBoot, IsSystem |
    Format-List
```

**Salida real:**

```text
Number         : 0
PartitionStyle : GPT
IsBoot         : True
IsSystem       : True
```

**Análisis:** El disco que contiene C: era el **Disco 0**, particionado **GPT** y mostraba `IsBoot=True` y `IsSystem=True` a nivel de objeto **disco**, no específicamente de partición. No se alteraron particiones.

## Paso 13 — Política de Script Block Logging en HKLM (verificado)

**Comando ejecutado:**

```powershell
$path = 'HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging'

if (Test-Path $path) {
    Get-ItemProperty -Path $path |
        Select-Object EnableScriptBlockLogging |
        Format-List
} else {
    'Policy registry key: Not configured'
}
```

**Salida real:**

```text
Policy registry key: Not configured
```

**Análisis:** La clave de política de máquina `ScriptBlockLogging` en HKLM no existía en la ruta examinada. No permite afirmar que todo registro de bloques de scripts esté desactivado.

## Paso 14 — Metadatos del canal PowerShell/Operational (verificado)

**Comando ejecutado:**

```powershell
Get-WinEvent -ListLog 'Microsoft-Windows-PowerShell/Operational' |
    Select-Object LogName,
                  IsEnabled,
                  RecordCount,
                  MaximumSizeInBytes,
                  LogMode |
    Format-List
```

**Salida real:**

```text
LogName            : Microsoft-Windows-PowerShell/Operational
IsEnabled          : True
RecordCount        : 808
MaximumSizeInBytes : 15728640
LogMode            : Circular
```

**Análisis:** El canal `Microsoft-Windows-PowerShell/Operational` estaba activo, con **808 eventos**, máximo **15 MiB** y retención circular. Esto no demuestra cobertura detallada de todos los scripts.

**Consulta de conteo propuesta en este punto.** El paso 15 conserva la consulta que se ejecutó y su resultado. La propuesta evita mostrar el contenido de los bloques de script.

```powershell
$log = 'Microsoft-Windows-PowerShell/Operational'
$events = Get-WinEvent -FilterHashtable @{ LogName = $log; Id = 4104 } -ErrorAction SilentlyContinue
[PSCustomObject]@{ EventId = 4104; Count = @($events).Count } | Format-List
```

## Paso 15 — Cantidad de eventos PowerShell 4104 (verificado)

**Comando ejecutado:**

```powershell
$count = (Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-PowerShell/Operational'
    Id = 4104
} -ErrorAction SilentlyContinue | Measure-Object).Count

[PSCustomObject]@{
    EventId = 4104
    Count = $count
} | Format-List
```

**Salida real:**

```text
EventId : 4104
Count   : 438
```

**Análisis:** Una consulta recuperó **438 eventos 4104** retenidos. El contenido de scripts no se consultó; la presencia de eventos no confirma cobertura exhaustiva.

## Paso 16 — Distribución de identificadores del canal PowerShell (verificado)

**Comando ejecutado:**

```powershell
Get-WinEvent -LogName 'Microsoft-Windows-PowerShell/Operational' -MaxEvents 1000 |
    Group-Object Id |
    Sort-Object Count -Descending |
    Select-Object Name, Count |
    Format-Table -AutoSize
```

**Salida real:**

```text
Name  Count
----  -----
4104    438
53504    63
12039    63
8196     63
40962    62
40961    62
8197     28
8194      9
8193      9
8195      9
4100      2
```

**Análisis:** Se identificaron **11 IDs** y **808 registros** en total; el identificador 4104 tenía 438. No se interpretaron contenidos ni se clasificó actividad maliciosa.

## Paso 17 — Política de Script Block Logging en HKCU (verificado)

**Comando ejecutado:**

```powershell
$path = 'HKCU:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging'

if (Test-Path $path) {
    Get-ItemProperty -Path $path |
        Select-Object EnableScriptBlockLogging |
        Format-List
} else {
    'HKCU policy registry key: Not configured'
}
```

**Salida real:**

```text
HKCU policy registry key: Not configured
```

**Análisis:** La clave `ScriptBlockLogging` en HKCU tampoco estaba configurada en la ruta inspeccionada. Se observaron eventos 4104 en otras comprobaciones: no debe confundirse ausencia de clave con ausencia total de telemetría.

## Paso 18 — Nivel de severidad de eventos 4104 (verificado)

**Comando ejecutado:**

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-PowerShell/Operational'
    Id = 4104
} |
    Group-Object LevelDisplayName |
    Select-Object Name, Count |
    Format-Table -AutoSize
```

**Salida real:**

```text
Name        Count
----        -----
Advertencia   439
```

**Análisis:** Una consulta posterior encontró **439 eventos 4104** de severidad `Advertencia` frente a 438 del conteo anterior. Son dos instantáneas distintas; la advertencia no es prueba de ejecución maliciosa.

## Paso 19 — Intervalo observado del historial PowerShell (verificado)

**Comando ejecutado:**

```powershell
$log = 'Microsoft-Windows-PowerShell/Operational'
$oldest = Get-WinEvent -LogName $log -Oldest -MaxEvents 1
$newest = Get-WinEvent -LogName $log -MaxEvents 1

[PSCustomObject]@{
    OldestEvent = $oldest.TimeCreated
    NewestEvent = $newest.TimeCreated
    RetainedDays = [math]::Round(($newest.TimeCreated - $oldest.TimeCreated).TotalDays, 2)
} | Format-List
```

**Salida real:**

```text
OldestEvent  : 13-11-2026 1:11:52
NewestEvent  : 18-11-2026 2:20:33
RetainedDays : 5,05
```

**Análisis:** El intervalo entre el evento PowerShell más antiguo y reciente era **5,05 días** (13/11/2026 01:11:52 a 18/11/2026 02:20:33). Es un intervalo observado, no retención mínima ni cobertura propia de ID 4104.

## Paso 20 — Política de Module Logging en HKLM (verificado)

**Comando ejecutado:**

```powershell
$path = 'HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ModuleLogging'

if (Test-Path $path) {
    Get-ItemProperty -Path $path |
        Select-Object EnableModuleLogging |
        Format-List
} else {
    'HKLM ModuleLogging policy key: Not configured'
}
```

**Salida real:**

```text
HKLM ModuleLogging policy key: Not configured
```

**Análisis:** La clave de política `ModuleLogging` en HKLM no estaba configurada en la ruta consultada. La configuración efectiva en otros ámbitos no se deduce de esta sola ruta.

## Paso 21 — Política de Module Logging en HKCU (verificado)

**Comando ejecutado:**

```powershell
$path = 'HKCU:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ModuleLogging'

if (Test-Path $path) {
    Get-ItemProperty -Path $path |
        Select-Object EnableModuleLogging |
        Format-List
} else {
    'HKCU ModuleLogging policy key: Not configured'
}
```

**Salida real:**

```text
HKCU ModuleLogging policy key: Not configured
```

**Análisis:** Tampoco existía la clave `ModuleLogging` del usuario actual en HKCU. Esto no constituye prueba definitiva de desactivación en todos los ámbitos.

El siguiente paso conserva la consulta de conteo 4103 y su resultado. Con `SilentlyContinue`, un conteo cero solo indica que la consulta no recuperó coincidencias.

## Paso 22 — Conteo de eventos Module Logging 4103 (verificado)

**Comando ejecutado:**

```powershell
$count = (Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-PowerShell/Operational'
    Id = 4103
} -ErrorAction SilentlyContinue | Measure-Object).Count

[PSCustomObject]@{
    EventId = 4103
    Count = $count
} | Format-List
```

**Salida real:**

```text
EventId : 4103
Count   : 0
```

**Análisis:** La consulta devolvió **0 eventos 4103** retenidos. Como se utilizó `-ErrorAction SilentlyContinue`, posibles errores no terminantes podrían ocultarse; es correcto indicar que no se recuperaron coincidencias, no que jamás existieron.

## Paso 23 — Política de Transcription en HKLM (verificado)

**Comando ejecutado:**

```powershell
$path = 'HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\Transcription'

if (Test-Path $path) {
    Get-ItemProperty -Path $path |
        Select-Object EnableTranscripting |
        Format-List
} else {
    'HKLM Transcription policy key: Not configured'
}
```

**Salida real:**

```text
HKLM Transcription policy key: Not configured
```

**Análisis:** La clave `Transcription` en HKLM no estaba configurada en la ruta examinada; no se analizaron archivos de transcripción.

## Paso 24 — Política de Transcription en HKCU (verificado)

**Comando ejecutado:**

```powershell
$path = 'HKCU:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\Transcription'

if (Test-Path $path) {
    Get-ItemProperty -Path $path |
        Select-Object EnableTranscripting |
        Format-List
} else {
    'HKCU Transcription policy key: Not configured'
}
```

**Salida real:**

```text
HKCU Transcription policy key: Not configured
```

**Análisis:** La clave `Transcription` en HKCU tampoco existía. Ambas rutas examinadas estaban ausentes, pero la transcripción interactiva y otros ámbitos no fueron evaluados.

## Paso 25 — Auditoría de inicios de sesión Windows (verificado)

**Comando ejecutado:**

```powershell
auditpol /get /subcategory:"{0cce9215-69ae-11d9-bed3-505054503030}"
```

**Salida real:**

```text
Directiva de auditoría del sistema
Categoría o subcategoría                  Configuración
Inicio/cierre de sesión
  Inicio de sesión                        Aciertos y errores
```

**Análisis:** `auditpol` informó **`Aciertos y errores`** para inicio de sesión, por lo que Windows está configurado para auditar ambos resultados; esto no verifica su retención o integridad total.

## Paso 26 — Eventos de autenticación en 24 horas (verificado)

**Comando ejecutado:**

```powershell
try {
    $events = Get-WinEvent -FilterHashtable @{
        LogName = 'Security'
        Id = 4624,4625
        StartTime = (Get-Date).AddHours(-24)
    } -ErrorAction Stop

    $events | Group-Object Id |
        Select-Object Name, Count |
        Format-Table -AutoSize
} catch {
    Write-Output ('Query status: ' + $_.Exception.Message)
}
```

**Salida real:**

```text
Name Count
---- -----
4624   235
```

**Análisis:** En la ventana móvil de 24 horas se recuperaron **235 eventos 4624** y **ningún grupo 4625**. Son registros de acceso, no 235 personas; ausencia de 4625 en la consulta no significa que nunca hayan existido errores.

## Paso 27 — Distribución de tipos de inicio de sesión 4624 (verificado)

**Comando ejecutado:**

```powershell
$from = (Get-Date).AddHours(-24)
$types = Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4624
    StartTime = $from
} -ErrorAction Stop | ForEach-Object {
    $xml = [xml]$_.ToXml()
    ($xml.Event.EventData.Data | Where-Object Name -eq 'LogonType').'#text'
}
$types | Group-Object | Sort-Object Count -Descending |
    Select-Object Name, Count | Format-Table -AutoSize
```

**Salida real:**

```text
Name Count
---- -----
5      229
11       4
7        2
```

**Análisis:** De los 235 registros 4624, **229 eran tipo 5 (Service)**, **4 tipo 11 (CachedInteractive)** y **2 tipo 7 (Unlock)**. El total coincide con la consulta anterior, pero las ventanas pueden avanzar.

## Paso 28 — Eventos de privilegios especiales 4672 (verificado)

**Comando ejecutado:**

```powershell
try {
    $count = (Get-WinEvent -FilterHashtable @{
        LogName = 'Security'
        Id = 4672
        StartTime = (Get-Date).AddHours(-24)
    } -ErrorAction Stop | Measure-Object).Count
    [PSCustomObject]@{ EventId = 4672; Count = $count } | Format-List
} catch {
    'Query status: ' + $_.Exception.Message
}
```

**Salida real:**

```text
EventId : 4672
Count   : 233
```

**Análisis:** Se recuperaron **233 registros 4672** en la ventana móvil de 24 horas. Reflejan asignación de privilegios especiales en nuevas sesiones, no uso probado de esos permisos ni 233 personas privilegiadas. No se correlacionaron con los 235 registros 4624 en esta consulta.

## Paso 29 — Priorización de hallazgos (análisis sin nuevos comandos)

| Prioridad | Evidencia | Riesgo o límite | Medida propuesta |
|---|---|---|---|
| Atención alta; requiere recuperación | C: sin cifrar, BitLocker Off, Secure Boot False; TPM listo y UEFI/GPT confirmados | Exposición potencial ante pérdida física y modificación del arranque | Preparar respaldos, medios de recuperación y custodia segura de claves; evaluar compatibilidad y aprobar cambios por separado |
| Media; retención | Security ampliado a 128 MiB bajo CHG-001; intervalo inmediato 1,96 días | Objetivo de siete días sin demostrar | Repetir mediciones en días posteriores y revisar capacidad/volumen de eventos |
| Media; auditoría | Creación de procesos sin auditoría | Cobertura potencial limitada de 4688 | Evaluar piloto acotado, privacidad, volumen y reversión; no autorizado todavía |
| Revisar; PowerShell | Claves HKLM/HKCU de Script Block, Module y Transcription ausentes; 4104 presentes, 4103 sin coincidencias | Cobertura efectiva desconocida | Evaluar políticas efectivas, objetivos de detección y privacidad antes de cambios |
| Monitoreo | Logon Success/Failure activo; 235 eventos 4624, 0 coincidencias 4625, 233 eventos 4672; predominan servicios | No se confirma legitimidad o abuso con agregados | Monitorizar tendencias e investigar con correlación controlada y autorización |
| Protección observada | Firewall ActiveStore Block/Allow, Defender con indicadores activos | Estado positivo limitado a comprobaciones seleccionadas | Verificar periódicamente sin modificar controles sin necesidad |

**Las prioridades no son puntuaciones CVSS ni una auditoría de cumplimiento.** No se atribuyó un incidente, explotación ni identidad real.

## Paso 30 — Conclusiones y cierre documental

**Etapa completada:** 28 comprobaciones de seguridad más dos etapas de interpretación y cierre. No equivale a un endurecimiento integral certificado.

**Cambio ejecutado:** únicamente **CHG-001**, ampliación de la capacidad máxima del registro Security de 20 a 128 MiB. La configuración fue verificada; la meta de siete días de retención sigue pendiente.

**Actividades pendientes:** preparar respaldos y recuperación antes de BitLocker/Secure Boot; reevaluar la retención cuando transcurra suficiente tiempo; proponer por separado auditoría de procesos o telemetría avanzada de PowerShell considerando privacidad, rendimiento y reversión.

**Integridad:** se publicaron comandos, indicadores, marcas de tiempo y agregados anonimizados. No hay evidencia confirmada de incidente ni pruebas de explotación. El análisis de privilegios 4672 no representa una elevación maliciosa confirmada.

## Evaluaciones futuras pendientes

Después de revisar la línea base, quedan por definir comprobaciones acotadas y de solo lectura sobre privilegios locales, estado de actualizaciones y controles del equipo. Los registros públicos excluyen inventarios completos, miembros de grupos administrativos, reglas detalladas, valores sensibles del registro y claves de recuperación.

## Normas para interpretar las evidencias

- **Verificado:** existe un comando realmente ejecutado y una salida suministrada.
- **Propuesto:** no implica implementado ni validado.
- **Desconocido:** no implica vulnerable.
- No publicar mensajes completos, usuarios, IP, contraseñas, claves BitLocker o datos corporativos sensibles.
- Distinguir fecha de observación, intervalo entre registros y obligación contractual de retención.

## Referencias técnicas

- [auditpol /get — Microsoft Learn](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/auditpol-get)
- [Get-WinEvent](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.diagnostics/get-winevent)
- [Get-NetFirewallProfile](https://learn.microsoft.com/en-us/powershell/module/netsecurity/get-netfirewallprofile)
- [Get-MpComputerStatus](https://learn.microsoft.com/en-us/powershell/module/defender/get-mpcomputerstatus)
- [Windows Security Baselines](https://learn.microsoft.com/en-us/windows/security/operating-system-security/device-management/windows-security-configuration-framework/windows-security-baselines)

*Las referencias respaldan métodos técnicos; los resultados reales proceden de los comandos documentados.*
