# Lab 01 — Configuración Básica de Routers

## 🎯 Objetivo

Configurar tres routers interconectados en serie aplicando configuración inicial completa: hostname, contraseñas, interfaces, mensajes de advertencia y conectividad básica. Verificar la conectividad end-to-end mediante ping y traceroute.


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


## ⚙️ Configuración

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

banner motd #
=========================================
  ACCESO RESTRINGIDO - Solo personal autorizado
=========================================
#

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
end
write memory
```

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

banner motd #
=========================================
  ACCESO RESTRINGIDO - Solo personal autorizado
=========================================
#
###

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

## Verificación

### 1. Estado de interfaces

R1# show ip interface brief

Resultado esperado

Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0    192.168.1.1     YES manual up                    up
Serial0/0/0           10.0.12.1       YES manual up                    up

### 2. Conectividad entre routers adyacentes


! Desde R1 hacia R2
R1# ping 10.0.12.2

! Desde R2 hacia R3
R2# ping 10.0.23.2

### 3. Verificar configuración guardada

R1# show running-config
R1# show startup-config


Confirmar que aparecen hostname, interfaces, contraseñas y banner.

### 4. Verificar banner y autenticación

! Cerrar sesión y volver a entrar
R1# exit


### 5. CDP — descubrimiento de vecinos


R1# show cdp neighbors
R1# show cdp neighbors detail

Resultado esperado en R1:

Device ID   Local Intrfce   Holdtme   Capability   Platform   Port ID
R2          Ser 0/0/0       120       R            ...        Ser 0/0/0


### 6. Traceroute end-to-end


! Desde PC1 hacia PC2 (requiere rutas estáticas — ver Lab 02)
PC1> tracert 192.168.2.10


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

