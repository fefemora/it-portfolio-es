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
