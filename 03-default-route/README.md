# Lab 03 — Ruta estática por defecto

En este lab se configura una ruta por defecto (default route) en lugar de rutas estáticas individuales para cada red. Es el método más común en routers de borde que envían todo el tráfico desconocido hacia un ISP o gateway central.

---

## Topología

Se agrega un router ISP a la topología anterior. R1 representa la red interna y R-ISP simula la salida a internet.

```
PC1 ── [R1] ──────────── [R2] ──────────── [R3] ── PC2
       Gi0/0  Se0/0/0  Se0/0/0  Se0/0/1  Se0/0/0  Gi0/0
                                                    
       192.168.1.0/24   10.0.12.0/30   10.0.23.0/30   192.168.2.0/24

[R1] ── Se0/0/1 ── [R-ISP]
        10.0.1.0/30       209.165.200.0/24 (red "internet")
```

| Dispositivo | Interfaz | IP            | Máscara         |
|-------------|----------|---------------|-----------------|
| R1          | Gi0/0    | 192.168.1.1   | 255.255.255.0   |
| R1          | Se0/0/0  | 10.0.12.1     | 255.255.255.252 |
| R1          | Se0/0/1  | 10.0.1.1      | 255.255.255.252 |
| R2          | Se0/0/0  | 10.0.12.2     | 255.255.255.252 |
| R2          | Se0/0/1  | 10.0.23.1     | 255.255.255.252 |
| R3          | Se0/0/0  | 10.0.23.2     | 255.255.255.252 |
| R3          | Gi0/0    | 192.168.2.1   | 255.255.255.0   |
| R-ISP       | Se0/0/0  | 10.0.1.2      | 255.255.255.252 |
| R-ISP       | Lo0      | 209.165.200.1 | 255.255.255.0   |
| PC1         | NIC      | 192.168.1.10  | 255.255.255.0   |
| PC2         | NIC      | 192.168.2.10  | 255.255.255.0   |

La interfaz Loopback en R-ISP simula una red de internet sin necesitar hardware adicional.

---

## Concepto

Una ruta por defecto coincide con cualquier destino que no esté en la tabla de rutas. Se define con la red `0.0.0.0` y máscara `0.0.0.0`:

```
ip route 0.0.0.0 0.0.0.0 <next-hop>
```

Aparece en la tabla de rutas como `S*` — la `S` indica estática y el `*` indica que es candidata a gateway of last resort.

Sin default route, si un router no conoce el destino simplemente descarta el paquete. Con default route, lo envía al gateway definido y ese router decide qué hacer.

---

## Configuración

### R-ISP

```
enable
configure terminal

hostname R-ISP

interface Serial0/0/0
 description ENLACE-ISP-R1
 ip address 10.0.1.2 255.255.255.252
 no shutdown

interface Loopback0
 description SIMULACION-INTERNET
 ip address 209.165.200.1 255.255.255.0
 no shutdown

end
write memory
```

### R1 — agregar interfaz y default route

```
R1(config)# interface Serial0/0/1
R1(config-if)# description ENLACE-R1-ISP
R1(config-if)# ip address 10.0.1.1 255.255.255.252
R1(config-if)# no shutdown
R1(config-if)# exit

R1(config)# ip route 0.0.0.0 0.0.0.0 10.0.1.2
```

### R2 y R3 — default route apuntando a R1

En lugar de agregar rutas individuales hacia la red ISP, R2 y R3 usan una default route hacia R1.

```
R2(config)# ip route 0.0.0.0 0.0.0.0 10.0.12.1

R3(config)# ip route 0.0.0.0 0.0.0.0 10.0.23.1
```

R-ISP necesita ruta de retorno hacia las redes internas:

```
R-ISP(config)# ip route 192.168.1.0 255.255.255.0 10.0.1.1
R-ISP(config)# ip route 192.168.2.0 255.255.255.0 10.0.1.1
R-ISP(config)# ip route 10.0.12.0 255.255.255.252 10.0.1.1
R-ISP(config)# ip route 10.0.23.0 255.255.255.252 10.0.1.1
```

Guardar en todos los routers:

```
end
write memory
```

---

## Verificación

**Gateway of last resort**

```
R1# show ip route
```

Buscar la línea:

```
Gateway of last resort is 10.0.1.2 to network 0.0.0.0
S*   0.0.0.0/0 [1/0] via 10.0.1.2
```

La `S*` confirma que la default route está activa y es el gateway of last resort.

**Ping hacia la red ISP**

```
PC1> ping 209.165.200.1
R1# ping 209.165.200.1
```

**Ping end-to-end interno — verificar que sigue funcionando**

```
PC1> ping 192.168.2.10
```

Las rutas estáticas del Lab 02 siguen activas. La default route solo se usa cuando no hay una ruta más específica.

**Traceroute hacia ISP**

```
PC1> tracert 209.165.200.1
```

Debe mostrar: R1 → R-ISP. PC1 no necesita saber que existe R-ISP — simplemente envía todo a R1 y R1 decide.

**Tabla de comandos usados**

| Comando | Qué muestra |
|---------|-------------|
| `show ip route` | Gateway of last resort y ruta `S*` |
| `show ip route static` | Solo rutas estáticas incluyendo default |
| `ping 209.165.200.1` | Conectividad hacia red ISP |
| `tracert 209.165.200.1` | Camino hasta el ISP |

---

## Notas

- La máscara `0.0.0.0` es la menos específica posible — por eso coincide con cualquier destino.
- Si existe una ruta más específica en la tabla, el router la prefiere sobre la default. Esto se llama longest prefix match.
- En redes reales, la default route hacia el ISP se aprende por BGP o la configura manualmente el administrador.
- Una sola default route reemplaza múltiples rutas estáticas hacia redes desconocidas — simplifica mucho la tabla de rutas en routers de borde.
