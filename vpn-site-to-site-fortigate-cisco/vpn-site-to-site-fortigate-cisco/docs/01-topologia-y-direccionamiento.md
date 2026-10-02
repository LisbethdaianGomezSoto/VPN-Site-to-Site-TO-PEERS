# 📘 Topología y direccionamiento

[⬅️ Volver al README](../README.md)

<div align="center">
<img src="../images/topologia/topologia-gns3.png" alt="Topología GNS3" width="900">
</div>

## Dispositivos

| Dispositivo | Rol | Plataforma |
|---|---|---|
| FortiG-ESTE | Firewall y peer VPN del Sitio A (configuración por GUI) | FortiGate-VM64-KVM · FortiOS 7.0.9 |
| R-1 | Router y peer VPN del Sitio B | Cisco 2691 · IOS 12.4(25d) |
| ISP | Proveedor con IPs públicas | Cisco 2691 · IOS 12.4 |
| SW-2 | Switch de acceso de los usuarios (VLAN 10) | Cisco IOSv L2 · IOS 15.2 |
| web-server | Servidor web HTTPS | Ubuntu + Apache2 (Docker en GNS3) |
| PC-ADMIN | Equipo de administración del FortiGate | VM / PC |
| PC1 | Usuario final (DHCP) | VPCS |

## Conexiones

| Dispositivo | Puerto | Conectado a |
|---|---|---|
| ISP | f0/0 | FortiGate Port1 |
| ISP | f0/1 | R-1 f0/0 |
| FortiGate | Port2 | web-server eth0 |
| FortiGate | Port4 | PC-ADMIN e0 |
| R-1 | f0/1 | SW-2 Gi0/0 (trunk 802.1Q) |
| SW-2 | Gi0/1 | PC1 e0 (acceso VLAN 10) |

## Plan de direccionamiento

Se usaron los últimos dígitos de la matrícula (**0701 → 7.1**).

| Red | Prefijo | Dispositivo | Interfaz | IP |
|---|---|---|---|---|
| Enlace FortiGate–ISP | `200.7.1.0/30` | ISP | f0/0 | `200.7.1.1` |
| | | FortiGate | Port1 | `200.7.1.2` |
| Enlace ISP–R-1 | `200.7.1.4/30` | ISP | f0/1 | `200.7.1.5` |
| | | R-1 | f0/0 | `200.7.1.6` |
| Servidor (**/28**) | `10.7.1.0/28` | FortiGate | Port2 | `10.7.1.1` |
| | | web-server | eth0 | `10.7.1.2` |
| Administración (/28) | `10.7.1.16/28` | FortiGate | Port4 | `10.7.1.17` |
| | | PC-ADMIN | e0 | `10.7.1.18` |
| Usuarios VLAN 10 (**/25**) | `10.7.1.128/25` | R-1 | f0/1.10 | `10.7.1.129` |
| | | PC1 | e0 | DHCP (`10.7.1.136`) |

## VLAN y DHCP

- **VLAN 10** (usuarios): trunk 802.1Q entre SW-2 Gi0/0 y R-1 f0/1 (*router-on-a-stick*).
- **DHCP en R-1:** pool `VLAN10`, red `10.7.1.128/25`, gateway `10.7.1.129`, DNS `8.8.8.8`, excluidas `10.7.1.129–10.7.1.135`.

## NAT

| Equipo | NAT | Observación |
|---|---|---|
| R-1 | `ip nat inside source list 100 interface f0/0 overload` | La ACL 100 excluye el tráfico hacia `10.7.1.0/28`, que debe ir por la VPN |
| FortiGate | NAT activado en las políticas hacia Port1 | NAT **desactivado** en las políticas de la VPN |

## Rutas

| Equipo | Destino | Salida |
|---|---|---|
| ISP | Redes conectadas | — (no conoce redes privadas) |
| R-1 | `0.0.0.0/0` | `200.7.1.5` (ISP) |
| FortiGate | `0.0.0.0/0` | `200.7.1.1` (ISP) |
| FortiGate | `10.7.1.128/25` | interfaz `VPN-R1` (distancia 10) |
| FortiGate | `10.7.1.128/25` | *Blackhole* (distancia 254) |
