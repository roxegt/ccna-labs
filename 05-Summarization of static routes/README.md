# Lab 05 — Sumarización de rutas estáticas

En lugar de configurar una ruta estática por cada red, la sumarización permite agrupar varias redes contiguas en una sola ruta. Esto reduce el tamaño de la tabla de rutas y simplifica la administración.

---

## Topología

```
[R1] ──── Se0/0/0 ── 10.0.12.0/30 ── Se0/0/0 ──── [R2]
Gi0/0                                               Gi0/0
Gi0/1                                               Gi0/1
Gi0/2                                               Gi0/2

LANs R1:                              LANs R2:
192.168.0.0/24                        192.168.4.0/24
192.168.1.0/24                        192.168.5.0/24
192.168.2.0/24                        192.168.6.0/24
192.168.3.0/24                        192.168.7.0/24
```

| Dispositivo | Interfaz | IP | Máscara |
|---|---|---|---|
| R1 | Se0/0/0 | 10.0.12.1 | 255.255.255.252 |
| R1 | Lo0 | 192.168.0.1 | 255.255.255.0 |
| R1 | Lo1 | 192.168.1.1 | 255.255.255.0 |
| R1 | Lo2 | 192.168.2.1 | 255.255.255.0 |
| R1 | Lo3 | 192.168.3.1 | 255.255.255.0 |
| R2 | Se0/0/0 | 10.0.12.2 | 255.255.255.252 |
| R2 | Lo0 | 192.168.4.1 | 255.255.255.0 |
| R2 | Lo1 | 192.168.5.1 | 255.255.255.0 |
| R2 | Lo2 | 192.168.6.1 | 255.255.255.0 |
| R2 | Lo3 | 192.168.7.1 | 255.255.255.0 |

Se usan interfaces Loopback para simular múltiples LANs sin necesitar hardware adicional.

---

## Concepto

Sin sumarización, R1 necesitaría 4 rutas estáticas para llegar a las redes de R2:

```
ip route 192.168.4.0 255.255.255.0 10.0.12.2
ip route 192.168.5.0 255.255.255.0 10.0.12.2
ip route 192.168.6.0 255.255.255.0 10.0.12.2
ip route 192.168.7.0 255.255.255.0 10.0.12.2
```

Con sumarización, las 4 redes se agrupan en una sola ruta:

```
ip route 192.168.4.0 255.255.252.0 10.0.12.2
```

**Cómo calcular la ruta resumen:**

Las redes `192.168.4.0` a `192.168.7.0` en binario:

```
192.168.00000100.0  → 192.168.4.0
192.168.00000101.0  → 192.168.5.0
192.168.00000110.0  → 192.168.6.0
192.168.00000111.0  → 192.168.7.0
         ^^^^^^
         bits comunes: 192.168.000001 = 22 bits
```

Ruta resumen: `192.168.4.0/22` = máscara `255.255.252.0`

Lo mismo aplica para las redes de R1 (`192.168.0.0` a `192.168.3.0`):

```
192.168.00000000.0  → 192.168.0.0
192.168.00000001.0  → 192.168.1.0
192.168.00000010.0  → 192.168.2.0
192.168.00000011.0  → 192.168.3.0
```

Ruta resumen: `192.168.0.0/22` = máscara `255.255.252.0`

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

no ip domain-lookup

interface Serial0/0/0
 description ENLACE-R1-R2
 ip address 10.0.12.1 255.255.255.252
 clock rate 128000
 no shutdown

interface Loopback0
 description LAN-SIMULADA-0
 ip address 192.168.0.1 255.255.255.0
 no shutdown

interface Loopback1
 description LAN-SIMULADA-1
 ip address 192.168.1.1 255.255.255.0
 no shutdown

interface Loopback2
 description LAN-SIMULADA-2
 ip address 192.168.2.1 255.255.255.0
 no shutdown

interface Loopback3
 description LAN-SIMULADA-3
 ip address 192.168.3.1 255.255.255.0
 no shutdown

ip route 192.168.4.0 255.255.252.0 10.0.12.2

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

no ip domain-lookup

interface Serial0/0/0
 description ENLACE-R2-R1
 ip address 10.0.12.2 255.255.255.252
 no shutdown

interface Loopback0
 description LAN-SIMULADA-4
 ip address 192.168.4.1 255.255.255.0
 no shutdown

interface Loopback1
 description LAN-SIMULADA-5
 ip address 192.168.5.1 255.255.255.0
 no shutdown

interface Loopback2
 description LAN-SIMULADA-6
 ip address 192.168.6.1 255.255.255.0
 no shutdown

interface Loopback3
 description LAN-SIMULADA-7
 ip address 192.168.7.1 255.255.255.0
 no shutdown

ip route 192.168.0.0 255.255.252.0 10.0.12.1

end
write memory
```

---

## Verificación

**Tabla de rutas — una sola ruta cubre 4 redes**

```
R1# show ip route
```

Resultado esperado:

```
S    192.168.4.0/22 [1/0] via 10.0.12.2
```

Una sola entrada `S` cubre `192.168.4.0`, `192.168.5.0`, `192.168.6.0` y `192.168.7.0`.

**Ping hacia cada red de R2**

```
R1# ping 192.168.4.1
R1# ping 192.168.5.1
R1# ping 192.168.6.1
R1# ping 192.168.7.1
```

Los 4 pings deben funcionar con una sola ruta estática configurada.

**Comparación sin sumarización vs con sumarización**

```
R1# show running-config | include ip route
```

Solo debe aparecer una línea — eso es la sumarización en acción.

**Tabla de comandos usados**

| Comando | Qué muestra |
|---|---|
| `show ip route` | Tabla de rutas con ruta resumen |
| `show ip interface brief` | Estado de interfaces Loopback y Serial |
| `show running-config \| include ip route` | Confirma que solo hay una ruta configurada |

---

## Notas

- La sumarización solo funciona cuando las redes son contiguas y sus bits de red comparten un prefijo común.
- Una ruta resumen puede incluir redes que no existen — si el bloque `192.168.4.0/22` incluye una red que no está configurada en R2, el tráfico hacia esa red llega a R2 pero R2 no sabe cómo entregarlo. Hay que tener cuidado con esto en producción.
- Las interfaces Loopback son siempre `up/up` y nunca fallan — por eso son ideales para simular redes en labs.
- En redes reales la sumarización es fundamental para mantener tablas de rutas manejables, especialmente en redes con cientos de subredes.
