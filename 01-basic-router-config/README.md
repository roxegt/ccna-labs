# Lab 01 — Configuración básica de routers

Configuración inicial de tres routers en serie: hostname, contraseñas, interfaces y banner. El objetivo es dejar los dispositivos operativos y verificar conectividad entre interfaces adyacentes.

La conectividad end-to-end (PC1 → PC2) se completa en el Lab 02 con rutas estáticas.

---

## Topología

```
PC1 ── [R1] ──────────── [R2] ──────────── [R3] ── PC2
       Gi0/0  Se0/0/0  Se0/0/0  Se0/0/1  Se0/0/0  Gi0/0

       192.168.1.0/24   10.0.12.0/30   10.0.23.0/30   192.168.2.0/24
```

Los enlaces seriales usan /30 — práctica estándar para enlaces punto a punto (solo 2 hosts útiles por segmento).

| Dispositivo | Interfaz | IP           | Máscara         |
|-------------|----------|--------------|-----------------|
| R1          | Gi0/0    | 192.168.1.1  | 255.255.255.0   |
| R1          | Se0/0/0  | 10.0.12.1    | 255.255.255.252 |
| R2          | Se0/0/0  | 10.0.12.2    | 255.255.255.252 |
| R2          | Se0/0/1  | 10.0.23.1    | 255.255.255.252 |
| R3          | Se0/0/0  | 10.0.23.2    | 255.255.255.252 |
| R3          | Gi0/0    | 192.168.2.1  | 255.255.255.0   |
| PC1         | NIC      | 192.168.1.10 | 255.255.255.0   |
| PC2         | NIC      | 192.168.2.10 | 255.255.255.0   |

---

## Configuración

### R1

```
enable
configure terminal

hostname R1

enable secret cisco123
line console 0
 password cisco
 login
 logging synchronous
line vty 0 4
 password cisco
 login
exit

banner motd #
=========================================
  ACCESO RESTRINGIDO - Solo personal autorizado
=========================================
#

no ip domain-lookup

interface GigabitEthernet0/0
 description LAN-PC1
 ip address 192.168.1.1 255.255.255.0
 no shutdown

interface Serial0/0/0
 description ENLACE-R1-R2
 ip address 10.0.12.1 255.255.255.252
 clock rate 128000
 no shutdown

end
write memory
```

### R2

```
enable
configure terminal

hostname R2

enable secret cisco123
line console 0
 password cisco
 login
 logging synchronous
line vty 0 4
 password cisco
 login
exit

banner motd #
=========================================
  ACCESO RESTRINGIDO - Solo personal autorizado
=========================================
#

no ip domain-lookup

interface Serial0/0/0
 description ENLACE-R2-R1
 ip address 10.0.12.2 255.255.255.252
 no shutdown

interface Serial0/0/1
 description ENLACE-R2-R3
 ip address 10.0.23.1 255.255.255.252
 clock rate 128000
 no shutdown

end
write memory
```

### R3

```
enable
configure terminal

hostname R3

enable secret cisco123
line console 0
 password cisco
 login
 logging synchronous
line vty 0 4
 password cisco
 login
exit

banner motd #
=========================================
  ACCESO RESTRINGIDO - Solo personal autorizado
=========================================
#

no ip domain-lookup

interface Serial0/0/0
 description ENLACE-R3-R2
 ip address 10.0.23.2 255.255.255.252
 no shutdown

interface GigabitEthernet0/0
 description LAN-PC2
 ip address 192.168.2.1 255.255.255.0
 no shutdown

end
write memory
```

---

## Verificación

**Estado de interfaces**

```
R1# show ip interface brief
```

Todas las interfaces configuradas deben aparecer en `up/up`. Si el protocolo está `down`, revisar `clock rate` en el lado DCE del enlace serial.

**Conectividad entre adyacentes**

```
R1# ping 10.0.12.2
R2# ping 10.0.23.2
```

Resultado esperado: `!!!!!`

**Configuración guardada**

```
R1# show running-config
R1# show startup-config
```

Confirmar que hostname, interfaces, contraseñas y banner están presentes en ambas.

**Descubrimiento de vecinos con CDP**

```
R1# show cdp neighbors
R1# show cdp neighbors detail
```

R1 debe ver a R2 en `Se0/0/0`. Si no aparece, el enlace no está activo a nivel L2.

**Tabla de comandos usados**

| Comando | Qué muestra |
|---------|-------------|
| `show ip interface brief` | Estado de todas las interfaces |
| `show running-config` | Configuración en RAM |
| `show startup-config` | Configuración en NVRAM |
| `show cdp neighbors detail` | Vecinos: IP, plataforma, IOS |
| `show interfaces Serial0/0/0` | Errores, encapsulación, estado |
| `show version` | Versión IOS, uptime, memoria |

---

## Notas

- `enable secret` cifra con MD5, a diferencia de `enable password` que guarda en texto plano.
- `clock rate` va en el lado DCE del enlace. En Packet Tracer se puede ver cuál es con `show controllers Serial0/0/0`.
- `no ip domain-lookup` evita que el router intente resolver un comando mal escrito como nombre de dominio — sin esto, cada typo genera un delay de ~30 segundos.
- `write memory` es equivalente a `copy running-config startup-config`.
