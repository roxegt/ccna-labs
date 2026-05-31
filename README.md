# CCNA Routing Labs

Colección de laboratorios prácticos de networking desarrollados en Cisco Packet Tracer. El foco está en protocolos de routing, partiendo desde configuración básica hasta redistribución entre protocolos.

Cada lab tiene su propia topología, configuraciones completas y sección de verificación.

## Herramientas

- Cisco Packet Tracer
- Cisco NetAcad

## Labs

### Bloque 1 — Rutas estáticas

| # | Lab | Temas |
|---|-----|-------|
| 01 | [Configuración básica de routers](./01-basic-router-config/) | hostname, interfaces, CDP |
| 02 | [Rutas estáticas simples](./02-static-routing/) | ip route, next-hop, exit interface |
| 03 | [Ruta estática por defecto](./03-default-route/) | 0.0.0.0/0, gateway of last resort |
| 04 | [Rutas estáticas flotantes](./04-floating-static/) | distancia administrativa, failover |
| 05 | [Sumarización de rutas estáticas](./05-static-summary/) | summarization manual, CIDR |

### Bloque 2 — RIPv2

| # | Lab | Temas |
|---|-----|-------|
| 06 | [RIPv2 básico](./06-ripv2-basic/) | network, no auto-summary |
| 07 | [RIPv2 con redistribución](./07-ripv2-redistribute/) | redistribute static |
| 08 | [Estático vs RIP](./08-static-vs-rip/) | distancia administrativa |

### Bloque 3 — OSPF Single Area

| # | Lab | Temas |
|---|-----|-------|
| 09 | [OSPF área 0 básico](./09-ospf-single-area/) | router-id, network, área backbone |
| 10 | [OSPF DR/BDR](./10-ospf-dr-bdr/) | elección DR, prioridad, passive-interface |
| 11 | [OSPF ruta por defecto](./11-ospf-default-route/) | default-information originate |
| 12 | [OSPF métricas y costo](./12-ospf-cost/) | bandwidth, auto-cost reference-bandwidth |
| 13 | [OSPF autenticación MD5](./13-ospf-auth/) | ip ospf authentication message-digest |

### Bloque 4 — OSPF Multi Area

| # | Lab | Temas |
|---|-----|-------|
| 14 | [OSPF multi-area básico](./14-ospf-multiarea/) | ABR, LSA tipo 1/2/3 |
| 15 | [OSPF rutas inter-área](./15-ospf-interarea/) | propagación entre áreas |
| 16 | [OSPF Stub y Totally Stub](./16-ospf-stub/) | reducción de LSAs |
| 17 | [OSPF NSSA](./17-ospf-nssa/) | not-so-stubby-area, LSA tipo 7 |

### Bloque 5 — EIGRP

| # | Lab | Temas |
|---|-----|-------|
| 18 | [EIGRP básico](./18-eigrp-basic/) | AS number, network, DUAL |
| 19 | [EIGRP sucesores](./19-eigrp-successors/) | successor, feasible successor |
| 20 | [EIGRP sumarización](./20-eigrp-summary/) | ip summary-address eigrp |

### Bloque 6 — Integración

| # | Lab | Temas |
|---|-----|-------|
| 21 | [Redistribución OSPF ↔ EIGRP](./21-redistribution/) | redistribute, seed metric |
| 22 | [Topología empresarial completa](./22-enterprise-topology/) | diseño jerárquico, troubleshooting |


## Progreso

![Labs completados](https://img.shields.io/badge/labs%20completados-0%2F22-red)
![Herramienta](https://img.shields.io/badge/herramienta-Packet%20Tracer-blue)
![Certificación](https://img.shields.io/badge/certificación-CCNA%20200--301-1ba0d7)


## Autor

Roxana Garat  
[GitHub](https://github.com/roxegt)
