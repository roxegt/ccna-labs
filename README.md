# 🌐 CCNA Routing Labs 

Colección de laboratorios prácticos de networking enfocados en protocolos de routing, desarrollados en **Cisco Packet Tracer** como parte de mi preparación para la certificación **CCNA (200-301)**.

Cada lab incluye topología, configuraciones completas y verificación de resultados.

---

## 🛠️ Herramientas utilizadas

- [Cisco Packet Tracer](https://www.netacad.com/courses/packet-tracer) — simulador de redes
- [Cisco NetAcad](https://www.netacad.com/) — plataforma de aprendizaje

---

## 📁 Estructura del repositorio

```
ccna-routing-labs/
├── README.md
├── 01-static-routing/
│   ├── README.md
│   ├── topology.png
│   ├── lab.pkt
│   └── configs/
│       ├── R1.txt
│       └── R2.txt
├── 02-default-route/
├── ...
└── assets/
    └── topologies/
```

Cada carpeta contiene:

- `README.md` — objetivo, topología, comandos y verificación
- `topology.png` — diagrama de la red
- `lab.pkt` — archivo de Packet Tracer
- `configs/` — configuraciones exportadas de cada dispositivo

---

## 📋 Índice de laboratorios

### 🔵 Bloque 1 — Rutas Estáticas

| # | Lab | Temas |
|---|-----|-------|
| 01 | [Configuración básica de routers](./01-basic-router-config/) | hostname, interfaces, CDP, verificación |
| 02 | [Rutas estáticas simples](./02-static-routing/) | `ip route`, next-hop, exit interface |
| 03 | [Ruta estática por defecto](./03-default-route/) | `0.0.0.0/0`, gateway of last resort |
| 04 | [Rutas estáticas flotantes](./04-floating-static/) | distancia administrativa, failover |
| 05 | [Sumarización de rutas estáticas](./05-static-summary/) | summarization manual, CIDR |

### 🟡 Bloque 2 — RIPv2

| # | Lab | Temas |
|---|-----|-------|
| 06 | [RIPv2 básico](./06-ripv2-basic/) | `network`, `no auto-summary`, versión 2 |
| 07 | [RIPv2 con redistribución](./07-ripv2-redistribute/) | `redistribute static`, rutas externas |
| 08 | [Estático vs RIP](./08-static-vs-rip/) | distancia administrativa, preferencia de ruta |

### 🟠 Bloque 3 — OSPF Single Area

| # | Lab | Temas |
|---|-----|-------|
| 09 | [OSPF área 0 básico](./09-ospf-single-area/) | `router-id`, `network`, área backbone |
| 10 | [OSPF DR/BDR](./10-ospf-dr-bdr/) | elección DR, prioridad, `passive-interface` |
| 11 | [OSPF ruta por defecto](./11-ospf-default-route/) | `default-information originate` |
| 12 | [OSPF métricas y costo](./12-ospf-cost/) | `bandwidth`, `auto-cost reference-bandwidth` |
| 13 | [OSPF autenticación MD5](./13-ospf-auth/) | `ip ospf authentication message-digest` |

### 🔴 Bloque 4 — OSPF Multi Area

| # | Lab | Temas |
|---|-----|-------|
| 14 | [OSPF multi-area básico](./14-ospf-multiarea/) | área backbone, ABR, LSA tipo 1/2/3 |
| 15 | [OSPF rutas inter-área](./15-ospf-interarea/) | propagación de rutas entre áreas |
| 16 | [OSPF Stub y Totally Stub](./16-ospf-stub/) | reducción de LSAs, rutas por defecto |
| 17 | [OSPF NSSA](./17-ospf-nssa/) | `not-so-stubby-area`, LSA tipo 7 |

### 🟣 Bloque 5 — EIGRP

| # | Lab | Temas |
|---|-----|-------|
| 18 | [EIGRP básico](./18-eigrp-basic/) | AS number, `network`, algoritmo DUAL |
| 19 | [EIGRP sucesores](./19-eigrp-successors/) | successor, feasible successor, `show ip eigrp topology` |
| 20 | [EIGRP sumarización](./20-eigrp-summary/) | `ip summary-address eigrp` |

### ⚫ Bloque 6 — Integración y troubleshooting

| # | Lab | Temas |
|---|-----|-------|
| 21 | [Redistribución OSPF ↔ EIGRP](./21-redistribution/) | `redistribute`, seed metric, rutas externas |
| 22 | [Topología empresarial completa](./22-enterprise-topology/) | diseño jerárquico, troubleshooting end-to-end |

---

## ✅ Comandos de verificación frecuentes

```bash
# Tabla de rutas
show ip route
show ip route ospf
show ip route eigrp

# OSPF
show ip ospf neighbor
show ip ospf interface
show ip ospf database

# EIGRP
show ip eigrp neighbors
show ip eigrp topology
show ip eigrp interfaces

# General
ping <ip>
traceroute <ip>
show ip interface brief
```

---

## 📌 Progreso

![Labs completados](https://img.shields.io/badge/labs%20completados-0%2F22-red)
![Herramienta](https://img.shields.io/badge/herramienta-Packet%20Tracer-blue)
![Certificación](https://img.shields.io/badge/certificación-CCNA%20200--301-1ba0d7)

---

## 👤 Autor
Roxana Garat
 
