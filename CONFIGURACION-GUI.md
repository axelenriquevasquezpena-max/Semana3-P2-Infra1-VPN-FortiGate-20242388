# Configuración documentada — FortiGate por GUI

Resumen de los valores aplicados y observados durante las pruebas. No es un archivo importable ni un respaldo completo.

## Interfaces y servicios

En Network → Interfaces:

| Equipo | Interfaz | Ajuste |
|---|---|---|
| Usuarios | port1 | DHCP de administración, 192.168.6.135/24 observada |
| Usuarios | port2 | Puerto padre sin IP; enlace al trunk del switch |
| Usuarios | VLAN10-USUARIOS | VLAN ID 10 sobre port2; 10.23.88.1/25; PING |
| Usuarios | port3 / WAN-ISP | 198.51.100.2/30; rol WAN; PING |
| WEB | port1 | DHCP de administración, 192.168.6.136/24 observada |
| WEB | port2 / LAN-WEB | 10.23.88.129/28; rol LAN; PING; DHCP servidor desactivado |
| WEB | port3 / WAN-ISP | 203.0.113.2/30; rol WAN; PING |

DHCP en VLAN10-USUARIOS: habilitado; rango 10.23.88.2–10.23.88.126; máscara 255.255.255.128; gateway Same as Interface IP; DNS Same as System DNS; concesión 604800 segundos. Resolución DNS externa no forma parte de las pruebas.

## Rutas

Network → Static Routes:

| Equipo | Destino | Salida |
|---|---|---|
| Usuarios | 0.0.0.0/0 | Gateway 198.51.100.1, port3; distancia 1 |
| WEB | 0.0.0.0/0 | Gateway 203.0.113.1, port3; distancia 1 |
| Usuarios | 10.23.88.128/28 | VPN-SEDE-B; distancia 10 |
| WEB | 10.23.88.0/25 | VPN-SEDE-A; distancia 10 |

Las rutas VPN se crearon seleccionando la interfaz del túnel y sin introducir un gateway WAN manual. La GUI mostraba posteriormente la IP del peer en la columna Gateway IP.

Durante el diagnóstico, la tabla activa tenía una ruta por defecto por port1 con distancia 5. Se dio preferencia a las rutas WAN con distancia 1; después se verificaron VPN Up y conectividad extremo a extremo. No confundir una ruta configurada (Enabled) con una ruta elegida en la tabla activa. Se intentó desactivar Retrieve default gateway from server en la administración; ese cambio por sí solo no eliminó la ruta observada durante la sesión.

## VPN IPsec

VPN → IPsec Tunnels → Create New → Custom:

| Parámetro | FGT-USUARIOS | FGT-WEB |
|---|---|---|
| Nombre | VPN-SEDE-B | VPN-SEDE-A |
| Peer estático | 203.0.113.2 | 198.51.100.2 |
| Interfaz | port3 | port3 |
| Autenticación | Clave compartida privada | Misma clave privada |
| IKE | v1, Main | v1, Main |
| Propuesta fase 1 | DES-SHA256 | DES-SHA256 |
| DH fase 1 | 14 | 14 |
| Vida fase 1 | 86400 s | 86400 s |
| Selector local fase 2 | 10.23.88.0/25 | 10.23.88.128/28 |
| Selector remoto fase 2 | 10.23.88.128/28 | 10.23.88.0/25 |
| Propuesta fase 2 | DES-SHA256 | DES-SHA256 |
| PFS / DH | Activado / 14 | Activado / 14 |
| Vida fase 2 | 43200 s | 43200 s |

Replay Detection y Auto-negotiate activados en ambos; Autokey Keep Alive aparece activado y bloqueado por la GUI. Puertos y protocolo de selectores: All. NAT Traversal Enable; DPD On Demand; Local ID vacío; XAUTH desactivado. Las políticas reducen los servicios permitidos a HTTPS y PING. DES-SHA256 quedó guardado y ambos túneles se verificaron Up.

## Políticas

Objetos de tipo Subnet: RED-USUARIOS = 10.23.88.0/25; SERVIDOR-WEB = 10.23.88.130/32; ISP-PRUEBA = 203.0.113.1/32.

| Equipo / regla | Entrada → salida | Origen → destino | Servicio | NAT |
|---|---|---|---|---|
| Usuarios: USUARIOS-A-WEB-VPN | VLAN10-USUARIOS → VPN-SEDE-B | RED-USUARIOS → SERVIDOR-WEB | HTTPS, PING | No |
| WEB: VPN-USUARIOS-A-WEB | VPN-SEDE-A → LAN-WEB | RED-USUARIOS → SERVIDOR-WEB | HTTPS, PING | No |
| Usuarios: USUARIOS-ISP-NAT | VLAN10-USUARIOS → WAN-ISP | RED-USUARIOS → ISP-PRUEBA | PING | Sí, dirección de salida |

Action ACCEPT, Schedule always, habilitadas, registro All Sessions; perfiles de seguridad desactivados y SSL Inspection no-inspection en estas pruebas. El resto queda sujeto a denegación implícita.

Para mantenimiento se configuró además ADMIN-SSH-WEB en FGT-WEB: port1 → port2, origen 192.168.6.1/32, destino 10.23.88.130/32, servicio SSH, ACCEPT, sin NAT. Es acceso auxiliar del administrador, no del usuario de VLAN 10. Windows utilizó una ruta de host hacia WEB por 192.168.6.136. El acceso de Kali sigue limitado a las reglas VPN descritas.

## Servidor

Ubuntu / Nginx; ens33 = 10.23.88.130/28, gateway 10.23.88.129. El servidor HTTPS existente escucha en 443 y usa root /var/www/p2. Se agregó vpn.html, sin reemplazar la aplicación PHP previa, que dependía de otra base de datos. Se verificó 200 OK en /vpn.html; no se afirma que la raíz PHP esté reparada.
