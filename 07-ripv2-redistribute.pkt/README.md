# Lab 07 — RIPv2 con redistribución

La redistribución permite que un protocolo de routing anuncie rutas que aprendió de otra fuente — en este caso, RIP anunciará rutas estáticas a sus vecinos. Esto es útil cuando parte de la red usa rutas estáticas y otra parte usa routing dinámico.

## Topología


| Dispositivo | Interfaz | IP | Máscara |
|---|---|---|---|
| R-ISP | Se0/0/0 | 10.0.1.2 | 255.255.255.252 |
| R-ISP | Lo0 | 209.165.200.1 | 255.255.255.0 |
| R1 | Se0/0/1 | 10.0.1.1 | 255.255.255.252 |
| R1 | Gi0/0 | 192.168.1.1 | 255.255.255.0 |
| R1 | Se0/0/0 | 10.0.12.1 | 255.255.255.252 |
| R2 | Se0/0/0 | 10.0.12.2 | 255.255.255.252 |
| R2 | Se0/0/1 | 10.0.23.1 | 255.255.255.252 |
| R2 | Gi0/0 | 192.168.2.1 | 255.255.255.0 |
| R3 | Se0/0/0 | 10.0.23.2 | 255.255.255.252 |
| PC1 | NIC | 192.168.1.10 | 255.255.255.0 |
| PC2 | NIC | 192.168.2.10 | 255.255.255.0 |

## Concepto

R1 tiene una ruta estática hacia R-ISP (`209.165.200.0/24`). R2 y R3 aprenden las redes internas por RIP, pero no saben nada del ISP.

Con `redistribute static`, R1 toma sus rutas estáticas y las inyecta en el proceso RIP — así R2 y R3 pueden llegar al ISP sin necesitar rutas estáticas propias.

Las rutas redistribuidas aparecen en la tabla con `R E` — R de RIP y E de external (externa al dominio RIP).


## Configuración

### R-ISP

enable
configure terminal

hostname R-ISP

no ip domain-lookup

interface Serial0/0/0
 description ENLACE-ISP-R1
 ip address 10.0.1.2 255.255.255.252
 no shutdown

interface Loopback0
 description SIMULACION-INTERNET
 ip address 209.165.200.1 255.255.255.0
 no shutdown

ip route 192.168.1.0 255.255.255.0 10.0.1.1
ip route 192.168.2.0 255.255.255.0 10.0.1.1

end
write memory

### R1

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

interface GigabitEthernet0/0
 description LAN-PC1
 ip address 192.168.1.1 255.255.255.0
 no shutdown

interface Serial0/0/0
 description ENLACE-R1-R2
 ip address 10.0.12.1 255.255.255.252
 clock rate 128000
 no shutdown

interface Serial0/0/1
 description ENLACE-R1-ISP
 ip address 10.0.1.1 255.255.255.252
 no shutdown

ip route 0.0.0.0 0.0.0.0 10.0.1.2

router rip
 version 2
 no auto-summary
 network 192.168.1.0
 network 10.0.0.0
 redistribute static metric 1
 default-information originate

end
write memory

### R2

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

interface Serial0/0/1
 description ENLACE-R2-R3
 ip address 10.0.23.1 255.255.255.252
 clock rate 128000
 no shutdown

interface GigabitEthernet0/0
 description LAN-PC2
 ip address 192.168.2.1 255.255.255.0
 no shutdown

router rip
 version 2
 no auto-summary
 network 10.0.0.0
 network 192.168.2.0

end
write memory

### R3

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

no ip domain-lookup

interface Serial0/0/0
 description ENLACE-R3-R2
 ip address 10.0.23.2 255.255.255.252
 no shutdown

router rip
 version 2
 no auto-summary
 network 10.0.0.0

end
write memory


## Verificación

**Rutas redistribuidas en R2 y R3**

R2# show ip route rip
R3# show ip route rip

R2 y R3 deben ver la ruta hacia `209.165.200.0/24` y la default route marcadas como `R` o `R*`:

R*   0.0.0.0/0 [120/1] via 10.0.12.1
R    209.165.200.0/24 [120/1] via 10.0.12.1

**Verificar redistribución en R1**

R1# show ip protocols

Debe aparecer `Redistributing: static` en la sección de RIP.

**Ping desde R3 hacia el ISP**

R3# ping 209.165.200.1

R3 no tiene ninguna ruta estática configurada — llega al ISP únicamente por la redistribución.

**Ping end-to-end**

PC1> ping 192.168.2.10
PC1> ping 209.165.200.1

**Tabla de comandos usados**

| Comando | Qué muestra |
|---|---|
| `show ip route rip` | Rutas aprendidas por RIP incluyendo redistribuidas |
| `show ip protocols` | Confirma que la redistribución está activa |
| `show ip route` | Tabla completa — buscar `R*` para la default redistributed |


## Notas

- `redistribute static metric 1` inyecta las rutas estáticas en RIP con métrica 1. Sin definir métrica, RIP no redistribuye la ruta.
- `default-information originate` redistribuye específicamente la ruta por defecto (`0.0.0.0/0`) hacia los vecinos RIP.
- Las rutas redistribuidas aparecen como externas en otros protocolos. En RIP simplemente se marcan con `R` igual que las internas.
- R-ISP tiene rutas estáticas de retorno hacia las LANs internas — sin esto los pings llegan pero las respuestas no vuelven.
