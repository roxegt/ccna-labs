Lab 04 — Rutas estáticas flotantes
Una ruta estática flotante es una ruta de respaldo que solo se activa cuando la ruta principal falla. Se logra asignándole una distancia administrativa mayor que la ruta primaria, de modo que normalmente queda oculta en la tabla de rutas y aparece solo cuando es necesaria.
A diferencia de topologías con routers intermedios, este lab usa conexión directa entre R1 y R2 por dos enlaces seriales — así el failover es inmediato y real cuando cae el enlace principal.

Topología
PC1 ── [R1-Sucursal] ══════════════════ [R2-HQ] ── PC2
              Se0/0/0 ── 10.0.12.0/30 ── Se0/0/0   (enlace principal)
              Se0/0/1 ── 10.0.13.0/30 ── Se0/0/1   (enlace respaldo)
              Gi0/0                      Gi0/0
         192.168.1.0/24            192.168.2.0/24
DispositivoInterfazIPMáscaraR1-SucursalGi0/0192.168.1.1255.255.255.0R1-SucursalSe0/0/010.0.12.1255.255.255.252R1-SucursalSe0/0/110.0.13.1255.255.255.252R2-HQSe0/0/010.0.12.2255.255.255.252R2-HQSe0/0/110.0.13.2255.255.255.252R2-HQGi0/0192.168.2.1255.255.255.0PC1NIC192.168.1.10255.255.255.0PC2NIC192.168.2.10255.255.255.0

Concepto
La distancia administrativa (AD) indica qué tan confiable es una ruta. Valor más bajo = mayor preferencia.
Tipo de rutaAD por defectoDirectamente conectada0Estática normal1OSPF110RIP120
Una ruta flotante usa una AD mayor que la ruta principal. Mientras la ruta principal esté activa, la flotante no aparece en la tabla de rutas. Cuando la principal cae, la flotante toma su lugar automáticamente.
ip route <red> <máscara> <next-hop> <distancia-administrativa>

Configuración
R1-Sucursal
enable
configure terminal

hostname R1-Sucursal

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

banner motd #
=========================================
  ACCESO RESTRINGIDO - Solo personal autorizado
=========================================
#

interface GigabitEthernet0/0
 description LAN-SUCURSAL
 ip address 192.168.1.1 255.255.255.0
 no shutdown

interface Serial0/0/0
 description ENLACE-PRINCIPAL-R1-R2
 ip address 10.0.12.1 255.255.255.252
 clock rate 128000
 no shutdown

interface Serial0/0/1
 description ENLACE-RESPALDO-R1-R2
 ip address 10.0.13.1 255.255.255.252
 clock rate 128000
 no shutdown

ip route 192.168.2.0 255.255.255.0 10.0.12.2
ip route 192.168.2.0 255.255.255.0 10.0.13.2 5

end
write memory
R2-HQ
enable
configure terminal

hostname R2-HQ

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

banner motd #
=========================================
  ACCESO RESTRINGIDO - Solo personal autorizado
=========================================
#

interface GigabitEthernet0/0
 description LAN-HQ
 ip address 192.168.2.1 255.255.255.0
 no shutdown

interface Serial0/0/0
 description ENLACE-PRINCIPAL-R2-R1
 ip address 10.0.12.2 255.255.255.252
 no shutdown

interface Serial0/0/1
 description ENLACE-RESPALDO-R2-R1
 ip address 10.0.13.2 255.255.255.252
 no shutdown

ip route 192.168.1.0 255.255.255.0 10.0.12.1
ip route 192.168.1.0 255.255.255.0 10.0.13.1 5

end
write memory

Verificación
Estado normal — ruta flotante no visible
R1-Sucursal# show ip route
Solo aparece la ruta principal. La flotante está configurada pero oculta por tener AD mayor.
S    192.168.2.0/24 [1/0] via 10.0.12.2
Ping normal antes del fallo
PC1> ping 192.168.2.10
Simular falla — apagar enlace principal en ambos lados
R1-Sucursal(config)# interface Serial0/0/0
R1-Sucursal(config-if)# shutdown

R2-HQ(config)# interface Serial0/0/0
R2-HQ(config-if)# shutdown
Verificar que la flotante tomó el lugar
R1-Sucursal# show ip route
Ahora aparece la ruta con AD 5:
S    192.168.2.0/24 [5/0] via 10.0.13.2
Ping después del fallo — debe seguir funcionando
PC1> ping 192.168.2.10
Restaurar enlace principal
R1-Sucursal(config)# interface Serial0/0/0
R1-Sucursal(config-if)# no shutdown

R2-HQ(config)# interface Serial0/0/0
R2-HQ(config-if)# no shutdown
R1-Sucursal# show ip route
La tabla vuelve a mostrar la ruta principal. La flotante desaparece.
Tabla de comandos usados
ComandoQué muestrashow ip routeTabla de rutas — confirma qué ruta está activashow ip route staticTodas las estáticas incluyendo flotantes instaladasshow running-config | include ip routeVer todas las rutas configuradas incluyendo flotantes ocultas

Notas

La ruta flotante no aparece en show ip route mientras la principal esté activa, pero sí en show running-config — siempre está configurada, solo que inactiva.
Con rutas estáticas el failover solo se detecta cuando cae el enlace directamente conectado al router. Para detectar fallos en equipos intermedios se necesita un protocolo de routing dinámico como OSPF o EIGRP.
El valor de AD para la flotante puede ser cualquier número mayor que la ruta principal. AD 5 funciona, pero también es común usar valores como 200 para dejar margen respecto a protocolos dinámicos.
