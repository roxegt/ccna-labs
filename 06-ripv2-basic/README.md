# Lab 06 — RIPv2 básico

RIP (Routing Information Protocol) es un protocolo de routing dinámico de vector distancia. En vez de configurar rutas manualmente como en los labs anteriores, los routers intercambian información de routing automáticamente y construyen sus tablas solos.

RIPv2 es la versión mejorada — soporta CIDR, VLSM y autenticación, a diferencia de RIPv1 que solo trabaja con clases.


## Topología


| Dispositivo | Interfaz | IP | Máscara |
|---|---|---|---|
| R1 | Gi0/0 | 192.168.1.1 | 255.255.255.0 |
| R1 | Se0/0/0 | 10.0.12.1 | 255.255.255.252 |
| R2 | Se0/0/0 | 10.0.12.2 | 255.255.255.252 |
| R2 | Se0/0/1 | 10.0.23.1 | 255.255.255.252 |
| R3 | Se0/0/0 | 10.0.23.2 | 255.255.255.252 |
| R3 | Gi0/0 | 192.168.2.1 | 255.255.255.0 |
| PC1 | NIC | 192.168.1.10 | 255.255.255.0 |
| PC2 | NIC | 192.168.2.10 | 255.255.255.0 |


## Concepto

RIP usa el número de saltos (hops) como métrica. Cada router entre origen y destino cuenta como 1 salto. El máximo es 15 — una red a 16 saltos se considera inalcanzable.

El comando `network` le indica a RIP qué interfaces participan en el proceso de routing. RIP anunciará esas redes a sus vecinos y escuchará actualizaciones por esas mismas interfaces.

`no auto-summary` es obligatorio en RIPv2 — sin esto el router resume automáticamente las redes a su clase natural (comportamiento de RIPv1) y puede causar problemas con subredes.


## Configuración

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

router rip
 version 2
 no auto-summary
 network 192.168.1.0
 network 10.0.0.0

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

router rip
 version 2
 no auto-summary
 network 10.0.0.0

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


## Verificación

**Rutas aprendidas por RIP**

R1# show ip route rip

R1 debe ver las redes de R2 y R3 marcadas con `R`:

R    10.0.23.0/30 [120/1] via 10.0.12.2
R    192.168.2.0/24 [120/2] via 10.0.12.2

El `[120/1]` indica distancia administrativa 120 (RIP) y métrica 1 salto. La red `192.168.2.0` tiene métrica 2 porque está a 2 saltos de R1.

**Vecinos y actualizaciones RIP**

R1# debug ip rip


Muestra las actualizaciones RIP en tiempo real — se pueden ver los anuncios que envía y recibe cada router. Para detenerlo:

R1# undebug all

**Protocolo RIP activo**

R1# show ip protocols

Muestra la versión de RIP, redes anunciadas, vecinos y temporizadores.

**Ping end-to-end**

PC1> ping 192.168.2.10
PC2> ping 192.168.1.10

**Traceroute**

PC1> tracert 192.168.2.10

Debe mostrar 3 saltos: R1 → R2 → R3.

**Tabla de comandos usados**

| Comando | Qué muestra |
|---|---|
| `show ip route rip` | Solo rutas aprendidas por RIP |
| `show ip protocols` | Versión, redes, vecinos, temporizadores |
| `debug ip rip` | Actualizaciones RIP en tiempo real |
| `show ip route` | Tabla completa con rutas RIP marcadas con R |


## Notas

- RIP envía actualizaciones completas de su tabla cada 30 segundos a la dirección multicast `224.0.0.9` (RIPv2) o broadcast (RIPv1).
- La distancia administrativa de RIP es 120 — mucho mayor que las rutas estáticas (1), por eso si coexisten una estática y una RIP hacia la misma red, el router prefiere la estática.
- `no auto-summary` es fundamental en RIPv2. Sin él, `10.0.12.0/30` se resume a `10.0.0.0/8` y se pierde la información de subred.
- RIP tiene un máximo de 15 saltos — no es adecuado para redes grandes. Para eso existen OSPF y EIGRP.
