# Diagnóstico de DNS local con Raspberry Pi 3

![Redes](https://img.shields.io/badge/Networking-DNS-14283F?style=flat-square) ![Laboratorio ejecutado](https://img.shields.io/badge/Lab-Verified-059669?style=flat-square)

[← Redes](../README.md) · [Caso simulado INC-003](../../IT-Support/Service-Desk-Incident-Management/incidents/INC-003-DNS-Resolution.md) · [English version](https://github.com/fefemora/it-portfolio-en/blob/main/Networking/Raspberry-Pi-DNS-Lab/README.md)

Monté un DNS local en una Raspberry Pi 3 Model B y lo consulté desde Windows. Quería comprobar cómo se ve, desde el cliente, un nombre ausente cuando el servidor DNS sigue respondiendo. Retiré temporalmente un registro A, observé la respuesta y confirmé que el nombre volvió a resolver después de restaurarlo.

## Entorno y configuración

- **Servidor** · Raspberry Pi 3 Model B, Raspbian GNU/Linux 13 (Trixie), `dnsmasq`, Wi-Fi (`wlan0`).
- **Recursos antes de instalar** · 767 MiB de RAM disponibles y 11 GiB libres en la microSD.
- **Cliente** · Windows, PowerShell y `nslookup`.

Comprobé que el puerto 53 estaba libre antes de instalar `dnsmasq`. La configuración inicial por interfaz también escuchaba en IPv6. Cambié `interface=wlan0` por `listen-address=192.168.1.17` y confirmé con `sudo ss -luntp 'sport = :53'` que el proceso quedó escuchando solo en esa IPv4, por UDP y TCP. El DNS del laboratorio quedó sin DHCP ni reenvío de consultas.

`/etc/dnsmasq.d/pi3-lab.conf`

```ini
listen-address=192.168.1.17
bind-dynamic
no-dhcp-interface=wlan0
no-resolv
no-hosts
local=/pi-lab.test/
host-record=intranet.pi-lab.test,192.168.1.17
```

## Pruebas desde Windows

Para las tres pruebas usé la misma consulta directa a la Pi.

```powershell
nslookup -type=A intranet.pi-lab.test 192.168.1.17
```

### 1. Línea base

El registro A devolvió la IPv4 configurada.

```text
Servidor:  intranet.pi-lab.test
Address:  192.168.1.17

Nombre:  intranet.pi-lab.test
Address:  192.168.1.17
```

### 2. Fallo controlado

Comenté solo la línea `host-record` y reinicié `dnsmasq`. El servidor respondió con `NXDOMAIN` (`Non-existent domain`).

```text
Servidor:  UnKnown
Address:  192.168.1.17

*** UnKnown no encuentra intranet.pi-lab.test: Non-existent domain
```

### 3. Recuperación

Volví a habilitar la línea `host-record`, reinicié `dnsmasq` y repetí la consulta. El registro A volvió a responder con la dirección esperada.

```text
Servidor:  intranet.pi-lab.test
Address:  192.168.1.17

Nombre:  intranet.pi-lab.test
Address:  192.168.1.17
```

## Qué comprobé

`NXDOMAIN` confirmó que el servidor DNS respondió durante el fallo controlado. En este caso, la causa fue el registro retirado. El texto `UnKnown` corresponde al nombre de servidor que `nslookup` intenta obtener por resolución inversa. La respuesta `NXDOMAIN` es el dato que usé para el diagnóstico.

La validación cubrió una consulta A directa desde un cliente Windows en la red local y el restablecimiento posterior. El caso corporativo INC-003 conserva su alcance de simulación.

## Seguimiento — Monitoreo automático con systemd

Automaticé la consulta al mismo servidor DNS con `dig` y un temporizador de `systemd`. La comprobación se ejecuta en la propia Raspberry Pi y registra el resultado en el journal local. Lee por separado el estado DNS y el código de salida de `dig`: `NXDOMAIN` es una respuesta válida del servidor, mientras que el código 9 indica que la consulta no recibió una respuesta DNS.

### Configuración instalada

Instalé `bind9-dnsutils` y guardé este script ejecutable en `/usr/local/bin/pi3-dns-check` con permisos `755`.

```sh
#!/bin/sh

server=192.168.1.17
name=intranet.pi-lab.test.
expected=192.168.1.17

reply=$(/usr/bin/dig -r "@$server" "$name" A \
    +time=2 +tries=1 +noall +comments +answer 2>&1)
result=$?

case "$result" in
    0) ;;
    9) echo "DNS_FAIL reason=NO_REPLY name=$name server=$server"; exit 1 ;;
    *) echo "DNS_FAIL reason=DIG_ERROR code=$result name=$name"; exit 1 ;;
esac

status=$(printf '%s\n' "$reply" | sed -n 's/^.*status: \([A-Z0-9]*\),.*$/\1/p')
if [ "$status" != NOERROR ]; then
    echo "DNS_FAIL reason=${status:-UNKNOWN_STATUS} name=$name server=$server"
    exit 1
fi

address=$(printf '%s\n' "$reply" | awk -v name="$name" '$1 == name && $4 == "A" {print $5}')
if [ "$address" != "$expected" ]; then
    echo "DNS_FAIL reason=UNEXPECTED_ANSWER name=$name expected=$expected"
    exit 1
fi

echo "DNS_OK name=$name server=$server answer=$address"
```

`/etc/systemd/system/pi3-dns-check.service`

```ini
[Unit]
Description=Raspberry Pi lab DNS check

[Service]
Type=oneshot
DynamicUser=yes
ExecStart=/usr/local/bin/pi3-dns-check
```

`/etc/systemd/system/pi3-dns-check.timer`

```ini
[Unit]
Description=Run the lab DNS check every minute

[Timer]
OnActiveSec=5s
OnUnitActiveSec=1min
AccuracySec=1s

[Install]
WantedBy=timers.target
```

Activé el temporizador con `sudo systemctl daemon-reload` y `sudo systemctl enable --now pi3-dns-check.timer`. El servicio ejecuta una consulta por activación y usa una cuenta dinámica de systemd.

### Pruebas y evidencia

Primero comenté el registro A de prueba, reinicié `dnsmasq` y esperé el resultado automático. Después restauré el registro y reinicié el servicio para comprobar que la consulta volvía a funcionar. Para probar la ausencia de respuesta, programé la restauración del servicio antes de detenerlo:

```bash
sudo systemd-run --unit=pi3-dns-restore --on-active=90s \
  --timer-property=AccuracySec=1s \
  /usr/bin/systemctl start dnsmasq.service &&
sudo systemctl stop dnsmasq.service
```

| Hora local (UTC−03:00) | Evidencia observada |
|---|---|
| 19:15:56 y 19:16:57 | `DNS_OK`, `answer=192.168.1.17` |
| 19:17:58 | `DNS_FAIL reason=NXDOMAIN` |
| 19:18:59 | `DNS_OK`, `answer=192.168.1.17` |
| 19:20:17 | `systemd` registró la detención de `dnsmasq.service`. |
| 19:21:00 | `DNS_FAIL reason=NO_REPLY` |
| 19:21:47 | El servicio transitorio de restauración volvió a iniciar `dnsmasq.service`. |
| 19:24:03 | `DNS_OK`, `answer=192.168.1.17` en el extracto posterior. |

La fecha de prueba fue el 25 de noviembre de 2026. El journal de `dnsmasq` muestra 90 segundos entre la detención y el arranque. El `DNS_OK` de las 19:24:03 es la comprobación posterior visible en el extracto y no establece la hora de la primera resolución exitosa tras el arranque.

Consulta de evidencia:

```bash
sudo journalctl -u pi3-dns-check.service \
  --since '2026-11-25 19:15:00' \
  --grep='DNS_(OK|FAIL)' -o short-iso --no-pager
```

La validación cubre una consulta A ejecutada desde el mismo equipo que aloja el resolvedor. Sirve para comprobar la respuesta del servicio local. La disponibilidad desde otro equipo de la red requiere una sonda externa.

## Seguimiento — Respaldo y restauración de DNS

Guardé la configuración del resolvedor, el script de comprobación y las unidades de servicio y temporizador de systemd en un archivo comprimido con rutas relativas. El respaldo se limitó a estos cuatro archivos del laboratorio:

- `etc/dnsmasq.d/pi3-lab.conf`
- `usr/local/bin/pi3-dns-check`
- `etc/systemd/system/pi3-dns-check.service`
- `etc/systemd/system/pi3-dns-check.timer`

1. **Respaldo en la Pi.** Creé `pi3-dns-20261125-200756.tar.gz` con `tar`, listé su contenido y verifiqué la suma SHA-256 con `sha256sum -c`. El resultado fue `pi3-dns-20261125-200756.tar.gz: OK`.
2. **Copia en otro equipo.** Transferí el archivo y el manifiesto a Windows por SFTP con PSCP y la clave de PuTTY. La comparación del manifiesto con `Get-FileHash -Algorithm SHA256` devolvió `True`.
3. **Restauración desde la copia de Windows.** Envié ese archivo de vuelta con otro nombre, comparé ambos archivos byte a byte con `cmp` y lo extraje en una carpeta de prueba nueva. Validé la configuración recuperada con `dnsmasq --test --conf-file` y el script con `sh -n`. El mensaje final solo se imprimía si todas las comprobaciones terminaban correctamente.

```text
dnsmasq: syntax check OK.
Restauración de prueba validada en: /home/<usuario>/pi3-dns-restore.7Btq9O
```

El nombre de usuario del extracto está anonimizado. La prueba verificó la integridad de la transferencia, la extracción y la sintaxis de DNS y del script. Los archivos restaurados quedaron en la carpeta de prueba y el servicio activo conservó su configuración. Arrancar el servicio y el temporizador con los archivos recuperados quedó fuera del alcance de esta prueba.
