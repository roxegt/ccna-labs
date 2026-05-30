# Lab 01 — Configuración Básica de Routers

## 🎯 Objetivo

Configurar tres routers interconectados en serie aplicando configuración inicial completa: hostname, contraseñas, interfaces, mensajes de advertencia y conectividad básica. Verificar la conectividad end-to-end mediante ping y traceroute.

---

## 🖧 Topología

```
PC1 ── [R1] ──────────── [R2] ──────────── [R3] ── PC2
       Gi0/0  Se0/0/0  Se0/0/0  Se0/0/1  Se0/0/0  Gi0/0
       
       192.168.1.0/24   10.0.12.0/30   10.0.23.0/30   192.168.2.0/24
```

### Tabla de direccionamiento

| Dispositivo | Interfaz | Dirección IP | Máscara |
|-------------|----------|--------------|---------|
| R1 | Gi0/0 | 192.168.1.1 | 255.255.255.0 |
| R1 | Se0/0/0 | 10.0.12.1 | 255.255.255.252 |
| R2 | Se0/0/0 | 10.0.12.2 | 255.255.255.252 |
| R2 | Se0/0/1 | 10.0.23.1 | 255.255.255.252 |
| R3 | Se0/0/0 | 10.0.23.2 | 255.255.255.252 |
| R3 | Gi0/0 | 192.168.2.1 | 255.255.255.0 |
| PC1 | NIC | 192.168.1.10 | 255.255.255.0 |
| PC2 | NIC | 192.168.2.10 | 255.255.255.0 |

> 💡 Las interfaces seriales usan /30 (2 hosts útiles) — práctica estándar para enlaces punto a punto.

---

## ⚙️ Configuración

### R1

```bash
enable
configure terminal

! Identificación
hostname R1

! Seguridad de acceso
enable secret cisco123
line console 0
 password cisco
 login
 logging synchronous
line vty 0 4
 password cisco
 login
exit

! Banner
banner motd #
=========================================
  ACCESO RESTRINGIDO - Solo personal autorizado
=========================================
#

! Deshabilitar búsqueda DNS (evita delays por typos)
no ip domain-lookup

! Interfaces
interface GigabitEthernet0/0
 description LAN-PC1
 ip address 192.168.1.1 255.255.255.0
 no shutdown

interface Serial0/0/0
 description ENLACE-R1-R2
 ip address 10.0.12.1 255.255.255.252
 clock rate 128000
 no shutdown

! Guardar configuración
end
write memory
```

### R2

```bash
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

```bash
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

## ✅ Verificación

### 1. Estado de interfaces

```bash
R1# show ip interface brief
```

Resultado esperado:
```
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0    192.168.1.1     YES manual up                    up
Serial0/0/0           10.0.12.1       YES manual up                    up
```

> ⚠️ Si el Status es `down` → revisar `no shutdown`. Si Protocol es `down` → revisar `clock rate` en el DCE.

---

### 2. Conectividad entre routers adyacentes

```bash
! Desde R1 hacia R2
R1# ping 10.0.12.2

! Desde R2 hacia R3
R2# ping 10.0.23.2
```

Resultado esperado: `!!!!!` (5/5 paquetes exitosos)

---

### 3. Verificar configuración guardada

```bash
R1# show running-config
R1# show startup-config
```

Confirmar que aparecen hostname, interfaces, contraseñas y banner.

---

### 4. Verificar banner y autenticación

```bash
! Cerrar sesión y volver a entrar
R1# exit
```

Debe aparecer el mensaje de advertencia antes del prompt de login.

---

### 5. CDP — descubrimiento de vecinos

```bash
R1# show cdp neighbors
R1# show cdp neighbors detail
```

Resultado esperado en R1:
```
Device ID   Local Intrfce   Holdtme   Capability   Platform   Port ID
R2          Ser 0/0/0       120       R            ...        Ser 0/0/0
```

> CDP confirma conectividad Layer 2 con el vecino directamente conectado. Si no aparece, el enlace no está activo.

---

### 6. Traceroute end-to-end

```bash
! Desde PC1 hacia PC2 (requiere rutas estáticas — ver Lab 02)
PC1> tracert 192.168.2.10
```

> 📌 En este lab la conectividad end-to-end (PC1 → PC2) **no funcionará aún** porque no hay rutas configuradas. Eso se resuelve en el Lab 02. Lo que sí debe funcionar es la conectividad entre interfaces directamente conectadas.

---

### 7. Resumen de comandos show utilizados

| Comando | Propósito |
|---------|-----------|
| `show ip interface brief` | Estado rápido de todas las interfaces |
| `show running-config` | Configuración activa en RAM |
| `show startup-config` | Configuración guardada en NVRAM |
| `show cdp neighbors` | Vecinos directamente conectados |
| `show cdp neighbors detail` | Detalle: IP, plataforma, versión IOS |
| `show version` | Versión de IOS, uptime, memoria |
| `show interfaces Serial0/0/0` | Detalle de interfaz: errores, encapsulación, estado |

---

## 📝 Conceptos clave

- **enable secret** usa MD5 — más seguro que `enable password`
- **clock rate** se configura en el lado DCE del enlace serial (en Packet Tracer, R1 y R2 en sus respectivos puertos DCE)
- **logging synchronous** evita que los mensajes del sistema interrumpan lo que estás escribiendo
- **no ip domain-lookup** evita que el router intente resolver typos como nombres de dominio (genera delays de ~30s)
- **write memory** = `copy running-config startup-config` — ambos guardan la config

---

## 📂 Archivos del lab

```
01-basic-router-config/
├── README.md
├── topology.png       ← captura de Packet Tracer
├── lab.pkt            ← archivo simulación
└── configs/
    ├── R1.txt
    ├── R2.txt
    └── R3.txt
```
