# Pruebas de aceptación

Las capturas del README registran los resultados observados. La demostración de FortiGate se hace por GUI; los comandos se ejecutan en Kali.

## Direcciones

```bash
ip -br -4 addr
ip route
```

Se observó 10.23.88.21/25 y default via 10.23.88.1 proto dhcp.

## Con VPN activa

Confirmar ambos túneles Up en VPN → IPsec Tunnels.

```bash
ping -c 4 10.23.88.130
curl -k -I --connect-timeout 10 https://10.23.88.130/vpn.html
sudo traceroute -I -n -m 8 -w 2 10.23.88.130
```

Resultado observado: ping sin pérdidas, HTTP/1.1 200 OK y traceroute de tres saltos. Abrir también https://10.23.88.130/vpn.html en Firefox de Kali.

## Sin VPN y recuperación

FGT-USUARIOS → Network → Interfaces → desplegar port3 → VPN-SEDE-B → Status Disabled → OK. No detener el servidor ni el ISP.

Repetir ping y curl: se observaron 100 % de pérdida y curl (28) timeout. La prueba con curl evita confundir una página almacenada en caché con acceso nuevo.

Restaurar Status Enabled por GUI y esperar Up. Repetir curl: se observó 200 OK. Terminar con el túnel habilitado.

## NAT

Capturar el enlace ISP FastEthernet0/0 ↔ FGT-USUARIOS port3 en GNS3. Filtro Wireshark:

```text
icmp && ip.addr == 203.0.113.1
```

Desde Kali:

```bash
ping -c 4 203.0.113.1
```

Se observaron cuatro respuestas. En WAN, las solicitudes/respuestas utilizan 198.51.100.2 como dirección traducida del usuario. NAT se limita a este destino y servicio; la política VPN sigue sin NAT.

## Evidencias y entrega

Las imágenes incluidas fueron aportadas durante la configuración y pruebas del laboratorio, no extraídas del video. No se incluye un archivo PCAP original. Los fragmentos .cfg son documentación reconstruida, no exportaciones completas. Mantener los respaldos con secretos fuera del repositorio.
