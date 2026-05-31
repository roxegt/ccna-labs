# Lab 04 — Rutas estáticas flotantes

Una ruta estática flotante es una ruta de respaldo que solo se activa cuando la ruta principal falla. Se logra asignándole una distancia administrativa mayor que la ruta primaria, de modo que normalmente queda oculta en la tabla de rutas y aparece solo cuando es necesaria.


## Topología

Dos caminos entre R1 y R3 — uno principal por R2 y uno de respaldo directo.


| Dispositivo | Interfaz | IP          | Máscara         |
|-------------|----------|-------------|-----------------|
| R1          | Gi0/0    | 192.168.1.1 | 255.255.255.0   |
| R1          | Se0/0/0  | 10.0.12.1   | 255.255.255.252 |
| R1          | Se0/0/1  | 10.0.13.1   | 255.255.255.252 |
| R2          | Se0/0/0  | 10.0.12.2   | 255.255.255.252 |
| R2          | Se0/0/1  | 10.0.23.1   | 255.255.255.252 |
| R3          | Se0/0/0  | 10.0.13.2   | 255.255.255.252 |
| R3          | Se0/0/1  | 10.0.23.2   | 255.255.255.252 |
| R3          | Gi0/0    | 192.168.2.1 | 255.255.255.0   |
| PC1         | NIC      | 192.168.1.10| 255.255.255.0   |
| PC2         | NIC      | 192.168.2.10| 255.255.255.0   |


## Concepto

La distancia administrativa (AD) indica qué tan confiable es una ruta. Valor más bajo = mayor preferencia.

| Tipo de ruta | AD por defecto |
|---|---|
| Directamente conectada | 0 |
| Estática normal | 1 |
| OSPF | 110 |
| RIP | 120 |

Una ruta flotante usa una AD mayor que la ruta principal. Mientras la ruta principal esté activa, la flotante no aparece en la tabla de rutas. Cuando la principal cae, la flotante toma su lugar automáticamente.

ip route <red> <máscara> <next-hop> <distancia-administrativa>

## Configuración

### R1

Ruta principal hacia `192.168.2.0/24` por R2 (AD 1 por defecto):

R1(config)# ip route 192.168.2.0 255.255.255.0 10.0.12.2
R1(config)# ip route 10.0.23.0 255.255.255.252 10.0.12.2

Ruta flotante hacia `192.168.2.0/24` por enlace directo a R3 (AD 5):

R1(config)# ip route 192.168.2.0 255.255.255.0 10.0.13.2 5
R1(config)# ip route 10.0.23.0 255.255.255.252 10.0.13.2 5

Interfaz de respaldo:

R1(config)# interface Serial0/0/1
R1(config-if)# description RESPALDO-R1-R3
R1(config-if)# ip address 10.0.13.1 255.255.255.252
R1(config-if)# clock rate 128000
R1(config-if)# no shutdown

### R2

R2(config)# ip route 192.168.1.0 255.255.255.0 10.0.12.1
R2(config)# ip route 192.168.2.0 255.255.255.0 10.0.23.2

### R3

Ruta principal por R2 (AD 1):

R3(config)# ip route 192.168.1.0 255.255.255.0 10.0.23.1
R3(config)# ip route 10.0.12.0 255.255.255.252 10.0.23.1

Ruta flotante por enlace directo a R1 (AD 5):

R3(config)# ip route 192.168.1.0 255.255.255.0 10.0.13.1 5
R3(config)# ip route 10.0.12.0 255.255.255.252 10.0.13.1 5

R3(config)# interface Serial0/0/0
R3(config-if)# description RESPALDO-R3-R1
R3(config-if)# ip address 10.0.13.2 255.255.255.252
R3(config-if)# no shutdown

Guardar en todos:

end
write memory

## Verificación

**Estado normal — ruta flotante no visible**

R1# show ip route

Solo debe aparecer la ruta principal hacia `192.168.2.0/24` via `10.0.12.2`. La flotante no aparece porque tiene AD mayor.

**Ping normal antes del fallo**

PC1> ping 192.168.2.10

**Simular falla — bajar la interfaz principal en R1**

R1(config)# interface Serial0/0/0
R1(config-if)# shutdown

**Verificar que la flotante tomó el lugar**

R1# show ip route

Ahora debe aparecer la ruta hacia `192.168.2.0/24` via `10.0.13.2` con AD 5.

**Ping después del fallo — debe seguir funcionando**

PC1> ping 192.168.2.10

Si el ping sigue respondiendo, la ruta flotante está funcionando correctamente.

**Restaurar el enlace principal**

R1(config)# interface Serial0/0/0
R1(config-if)# no shutdown

R1# show ip route

La tabla vuelve a mostrar la ruta principal via R2. La flotante desaparece nuevamente.

**Tabla de comandos usados**

| Comando | Qué muestra |
|---------|-------------|
| `show ip route` | Tabla de rutas — confirma qué ruta está activa |
| `show ip route static` | Todas las estáticas, incluyendo flotantes instaladas |
| `show running-config \| include ip route` | Ver todas las rutas configuradas incluyendo las flotantes ocultas |
| `interface Serial0/0/0` + `shutdown` | Simular falla del enlace principal |
