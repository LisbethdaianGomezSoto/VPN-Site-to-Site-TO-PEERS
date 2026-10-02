# 📸 Checklist de capturas

[⬅️ Volver al README](../README.md)

Estado de las capturas del repositorio. Los archivos nuevos se guardan en la carpeta indicada con el nombre sugerido.

## FortiGate (GUI) — `images/fortigate-gui/`

| Archivo | Contenido | Estado |
|---|---|---|
| `01-ipsec-fase1-red.png` | Túnel VPN-R1: red y gateway remoto | ✅ Incluida |
| `02-ipsec-fase1-propuesta.png` | Fase 1: DES, SHA1, DH 2 | ✅ Incluida |
| `03-ipsec-autenticacion-psk.png` | PSK, IKEv1, Main mode | ✅ Incluida |
| `04-ipsec-fase2-selectores.png` | Fase 2: selectores, PFS, lifetime | ✅ Incluida |
| `05-interfaces.png` | Port1, Port2 y Port4 | ✅ Incluida |
| `06-rutas-estaticas.png` | Ruta por defecto, ruta VPN y blackhole | ✅ Incluida |
| `07-objetos-direccion.png` | SRV_NET y USERS_NET | ✅ Incluida |
| `08-politicas-firewall.png` | Políticas de firewall | ✅ Incluida |
| `09-monitor-ipsec.png` | IPsec Monitor con el túnel en verde | ✅ Incluida |
| `10-forward-traffic.png` | Log & Report → Forward Traffic con sesiones por VPN-R1 | ⬜ Pendiente |
| `11-vpn-events.png` | Log & Report → VPN Events (negociación) | ⬜ Pendiente |
| `12-monitor-ipsec-con-trafico.png` | IPsec Monitor con bytes entrantes y salientes mayores que 0 | ⬜ Pendiente |
| `13-politicas-con-bytes.png` | Políticas con contadores de la VPN mayores que 0 | ⬜ Pendiente |

## Cisco — `images/cisco/`

| Archivo | Comando | Estado |
|---|---|---|
| `isp-interfaces.png` | `show ip interface brief` (ISP) | ⬜ Pendiente |
| `isp-cdp.png` | `show cdp neighbors` (ISP) | ⬜ Pendiente |
| `sw2-vlan-trunk.png` | `show vlan brief` y `show interfaces trunk` | ⬜ Pendiente |
| `r1-interfaces.png` | `show ip interface brief` (R-1) | ⬜ Pendiente |
| `r1-dhcp.png` | `show ip dhcp pool` y `show ip dhcp binding` | ⬜ Pendiente |
| `r1-crypto-map.png` | `show crypto map` | ⬜ Pendiente |
| `r1-isakmp-sa.png` | `show crypto isakmp sa` (QM_IDLE) | ⬜ Pendiente |
| `r1-ipsec-sa.png` | `show crypto ipsec sa \| include pkts` | ⬜ Pendiente |

## Evidencias — `images/evidencias/`

| Archivo | Contenido | Estado |
|---|---|---|
| `01-pc1-dhcp.png` | PC1 recibe IP por DHCP | ✅ Incluida |
| `02-ping-traceroute-con-vpn.png` | Ping y traceroute funcionando | ✅ Incluida |
| `03-https-servidor.png` | Acceso HTTPS al servidor | ✅ Incluida |
| `04-sin-vpn-ping-traceroute-falla.png` | Ping y traceroute fallando con la VPN caída | ✅ Incluida |
| `05-captura-wireshark-esp.png` | Wireshark en el enlace ISP–R-1 filtrando `esp` | ⬜ Pendiente |
| `06-pc-admin-ping.png` | PC-ADMIN con ping a Port4 y al servidor | ⬜ Pendiente |
| `07-web-server-ip.png` | `ip a` e `ip route` del servidor | ⬜ Pendiente |
