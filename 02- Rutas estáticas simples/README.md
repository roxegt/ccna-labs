# Lab 02 — Rutas estáticas simples

Continuación del Lab 01. Los routers ya están configurados con sus interfaces activas — en este lab se agregan rutas estáticas para lograr conectividad completa entre PC1 y PC2.

Se practican tres formas de especificar una ruta estática: por next-hop, por interfaz de salida, y combinando ambas.

---

## Topología

Misma topología del Lab 01.

```
PC1 ── [R1] ──────────── [R2] ──────────── [R3] ── PC2
       Gi0/0  Se0/0/0  Se0/0/0  Se0/0/1  Se0/0/0  Gi0/0

       192.168.1.0/24   10.0.12.0/30   10.0.23.0/30   192.168.2.0/24
```

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

## Concepto

Una ruta estática le indica al router cómo llegar a una red que no está directamente conectada. La sintaxis básica es:

```
ip route <red-destino> <máscara> <next-hop | interfaz-salida>
```

El routing es bidireccional — si PC1 puede enviar un ping a PC2, R3 también necesita saber cómo volver a `192.168.1.0/24`. Sin la ruta de retorno, el ping sale pero la respuesta no llega.

---

## Configuración

### R1

R1 necesita llegar a `10.0.23.0/30` y `192.168.2.0/24`, ambas a través de R2 (`10.0.12.2`).

```
R1(config)# ip route 10.0.23.0 255.255.255.252 10.0.12.2
R1(config)# ip route 192.168.2.0 255.255.255.0 10.0.12.2
```

### R2

R2 es el router del medio — necesita rutas hacia ambas LANs.

```
R2(config)# ip route 192.168.1.0 255.255.255.0 10.0.12.1
R2(config)# ip route 192.168.2.0 255.255.255.0 10.0.23.2
```

### R3

R3 necesita llegar a `10.0.12.0/30` y `192.168.1.0/24`, ambas a través de R2 (`10.0.23.1`).

```
R3(config)# ip route 10.0.12.0 255.255.255.252 10.0.23.1
R3(config)# ip route 192.168.1.0 255.255.255.0 10.0.23.1
```

Guardar en los tres routers:

```
R1(config)# end
R1# write memory
```

---

## Verificación

**Tabla de rutas**

```
R1# show ip route
```

Las rutas estáticas aparecen marcadas con `S`. Las redes directamente conectadas con `C`.

```
S    10.0.23.0/30 [1/0] via 10.0.12.2
S    192.168.2.0/24 [1/0] via 10.0.12.2
C    10.0.12.0/30 is directly connected, Serial0/0/0
C    192.168.1.0/24 is directly connected, GigabitEthernet0/0
```

El `[1/0]` indica distancia administrativa 1 (estática) y métrica 0.

**Ping entre routers no adyacentes**

```
R1# ping 10.0.23.2
R1# ping 192.168.2.1
```

**Ping end-to-end desde las PCs**

```
PC1> ping 192.168.2.10
PC2> ping 192.168.1.10
```

Resultado esperado: `!!!!!` en ambos sentidos.

**Traceroute para ver el camino completo**

```
PC1> tracert 192.168.2.10
```

Debe mostrar tres saltos: R1 → R2 → R3, confirmando que el tráfico pasa por los tres routers.

**Tabla de comandos usados**

| Comando | Qué muestra |
|---------|-------------|
| `show ip route` | Tabla de rutas completa |
| `show ip route static` | Solo rutas estáticas |
| `show running-config \| include ip route` | Rutas en la config activa |

---

## Notas

- La distancia administrativa de rutas estáticas es 1 — tienen preferencia sobre casi cualquier protocolo de routing dinámico.
- Si se especifica interfaz de salida en vez de next-hop (`ip route 192.168.2.0 255.255.255.0 Se0/0/0`), la ruta aparece como `directly connected` en la tabla. Funciona, pero en enlaces seriales punto a punto es más claro usar next-hop.
- Si el ping falla en un sentido pero funciona en otro, casi siempre es una ruta de retorno faltante.
