CCNA Routing Labs
Colección de laboratorios prácticos de networking desarrollados en Cisco Packet Tracer. El foco está en protocolos de routing, partiendo desde configuración básica hasta redistribución entre protocolos.
Cada lab tiene su propia topología, configuraciones completas y sección de verificación.

Herramientas

Cisco Packet Tracer
Cisco NetAcad


Estructura
Cada carpeta sigue el mismo formato:
01-basic-router-config/
├── README.md        — objetivo, topología, comandos y verificación
├── topology.png     — diagrama de la red
├── lab.pkt          — archivo de Packet Tracer
└── configs/
    ├── R1.txt
    ├── R2.txt
    └── R3.txt

Labs
Bloque 1 — Rutas estáticas
#LabTemas01Configuración básica de routershostname, interfaces, CDP02Rutas estáticas simplesip route, next-hop, exit interface03Ruta estática por defecto0.0.0.0/0, gateway of last resort04Rutas estáticas flotantesdistancia administrativa, failover05Sumarización de rutas estáticassummarization manual, CIDR
Bloque 2 — RIPv2
#LabTemas06RIPv2 básiconetwork, no auto-summary07RIPv2 con redistribuciónredistribute static08Estático vs RIPdistancia administrativa
Bloque 3 — OSPF Single Area
#LabTemas09OSPF área 0 básicorouter-id, network, área backbone10OSPF DR/BDRelección DR, prioridad, passive-interface11OSPF ruta por defectodefault-information originate12OSPF métricas y costobandwidth, auto-cost reference-bandwidth13OSPF autenticación MD5ip ospf authentication message-digest
Bloque 4 — OSPF Multi Area
#LabTemas14OSPF multi-area básicoABR, LSA tipo 1/2/315OSPF rutas inter-áreapropagación entre áreas16OSPF Stub y Totally Stubreducción de LSAs17OSPF NSSAnot-so-stubby-area, LSA tipo 7
Bloque 5 — EIGRP
#LabTemas18EIGRP básicoAS number, network, DUAL19EIGRP sucesoressuccessor, feasible successor20EIGRP sumarizaciónip summary-address eigrp
Bloque 6 — Integración
#LabTemas21Redistribución OSPF ↔ EIGRPredistribute, seed metric22Topología empresarial completadiseño jerárquico, troubleshooting

Autor
Roxana Garat
GitHub
