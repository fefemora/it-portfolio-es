# Triaje de autenticación SSH en Raspberry Pi

![Blue Team](https://img.shields.io/badge/Blue_Team-Authentication_Triage-059669?style=flat-square) ![OpenSSH](https://img.shields.io/badge/OpenSSH-journalctl-14283F?style=flat-square) ![Lab](https://img.shields.io/badge/Lab-Verified-64748B?style=flat-square)

[← Ciberseguridad](../README.md) · [Laboratorio DNS con Raspberry Pi](../../Networking/Raspberry-Pi-DNS-Lab/README.md) · [English version](https://github.com/fefemora/it-portfolio-en/blob/main/Cybersecurity/Raspberry-Pi-SSH-Authentication-Triage/README.md)

Mientras administraba una Raspberry Pi 3 Model B desde Windows con PuTTY, ingresé mal mi contraseña SSH. Revisé qué había quedado en el registro de OpenSSH y comparé los rechazos con un inicio de sesión exitoso posterior. Quería determinar si el acceso exitoso correspondía a la misma conexión o a una nueva.

## Revisión del registro

El 25 de noviembre de 2026 consulté las 30 entradas recientes del servicio SSH en Raspbian 13 (Trixie). Usé el formato ISO para conservar la hora local y el desplazamiento UTC−03:00.

```bash
sudo journalctl -u ssh.service -n 30 -o short-iso --no-pager
```

Extracto transcrito de la captura de PuTTY. Reemplacé el usuario y la IPv6 de origen por marcadores consistentes y uní las líneas que PuTTY había dividido visualmente.

```text
2026-11-25T16:32:41-03:00 pi3-lab sshd-session[2830]: Failed password for <lab-user> from <client-ipv6> port 57461 ssh2
2026-11-25T16:32:50-03:00 pi3-lab sshd-session[2830]: Failed password for <lab-user> from <client-ipv6> port 57461 ssh2
2026-11-25T16:32:55-03:00 pi3-lab sshd-session[2830]: Connection closed by authenticating user <lab-user> <client-ipv6> port 57461 [preauth]
2026-11-25T16:33:18-03:00 pi3-lab sshd-session[2833]: Accepted password for <lab-user> from <client-ipv6> port 63839 ssh2
```

## Línea de tiempo

| Hora local | Evidencia | Puerto de origen / PID |
|---|---|---|
| 16:32:41 y 16:32:50 | Dos rechazos de contraseña | `57461` / `2830` |
| 16:32:55 | Cierre antes de completar la autenticación | `57461` / `2830` |
| 16:33:18 | Contraseña aceptada y apertura de sesión confirmada por PAM en la entrada siguiente | `63839` / `2833` |

## Análisis y cierre

Correlacioné las entradas por hora, usuario, IPv6 de origen, puerto de origen y PID de `sshd-session`. Los dos rechazos comparten puerto y PID con el cierre marcado `[preauth]`. El acceso exitoso ocurrió 23 segundos después del cierre, con el mismo usuario e IPv6 y con un puerto y PID distintos. La secuencia muestra que el acceso se aceptó en una nueva conexión.

Clasifiqué este episodio como actividad legítima del laboratorio porque pude relacionarlo con mi error al ingresar la contraseña y mi acceso posterior desde PuTTY. El registro por sí solo permite reconstruir la secuencia de autenticación. La atribución al operador depende del contexto de la prueba y la revisión se limita a los eventos seleccionados.
