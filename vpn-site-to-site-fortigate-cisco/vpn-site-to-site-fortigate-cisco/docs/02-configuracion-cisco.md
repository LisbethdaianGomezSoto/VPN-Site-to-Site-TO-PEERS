# 📗 Configuración de los equipos Cisco

[⬅️ Volver al README](../README.md)

Todos los scripts están en [`configs/scripts/`](../configs/scripts/) y las configuraciones finales en [`configs/running-config/`](../configs/running-config/). 

Cada script empieza con la **configuración base** (nombre, usuario, SSH y llaves RSA) y continúa con la parte del laboratorio. `<CONTRASEÑA>` 

## 1. ISP

El ISP solo tiene las dos IPs públicas. **No** tiene rutas hacia las redes privadas.

```
! ============================================================
!  ISP (Cisco 2691 · IOS 12.4) - Proveedor con IPs públicas
!  Nota: <CONTRASEÑA> sustituye la clave real del laboratorio.
! ============================================================

! ---------- Configuración base ----------
enable
configure terminal
no ip domain-lookup
hostname ISP
banner motd #Lisbeth Gómez 2025-0701#
username Admin privilege 15 secret <CONTRASEÑA>
enable secret <CONTRASEÑA>
line console 0
login local
exit
ip domain-name red.local
crypto key generate rsa
2048
ip ssh version 2
line vty 0 4
login local
transport input ssh
exit
end
write memory

! ---------- Interfaces hacia los peers ----------
en
config t
interface f0/0
ip address 200.7.1.1 255.255.255.252
no shutdown
interface f0/1
ip address 200.7.1.5 255.255.255.252
no shutdown
end
write memory
```

Verificación: `show ip interface brief`, `show cdp neighbors`, `ping 200.7.1.2` y `ping 200.7.1.6`.

## 2. SW-2

VLAN 10 para los usuarios y un trunk 802.1Q hacia R-1.

```
! ============================================================
!  SW-2 (Cisco IOSv L2 · IOS 15.2) - Switch de usuarios
!  Nota: <CONTRASEÑA> sustituye la clave real del laboratorio.
! ============================================================

! ---------- Configuración base ----------
enable
configure terminal
no ip domain-lookup
hostname SW-2
banner motd #Lisbeth Gómez 2025-0701#
username Admin privilege 15 secret <CONTRASEÑA>
enable secret <CONTRASEÑA>
line console 0
login local
exit
ip domain-name red.local
crypto key generate rsa
2048
ip ssh version 2
line vty 0 4
login local
transport input ssh
exit
end
write memory

! ---------- VLAN 10 y puertos ----------
en
config t
vlan 10
interface Gi0/0
switchport trunk encapsulation dot1q
switchport mode trunk
interface Gi0/1
switchport mode access
switchport access vlan 10
end
write memory
```

Verificación: `show vlan brief` y `show interfaces trunk`.

## 3. R-1 (red, DHCP, NAT y VPN)

```
! ============================================================
!  R-1 (Cisco 2691 · IOS 12.4) - Red, DHCP, NAT y VPN IPsec
!  Peer: FortiGate 200.7.1.2
!  Nota: <CONTRASEÑA> sustituye la clave real del laboratorio.
! ============================================================

! ---------- Configuración base ----------
enable
configure terminal
no ip domain-lookup
hostname R-1
banner motd #Lisbeth Gómez 2025-0701#
username Admin privilege 15 secret <CONTRASEÑA>
enable secret <CONTRASEÑA>
line console 0
login local
exit
ip domain-name red.local
crypto key generate rsa
2048
ip ssh version 2
line vty 0 4
login local
transport input ssh
exit
end
write memory

! ---------- Interfaces ----------
config t
interface f0/0
 ip address 200.7.1.6 255.255.255.252
 ip nat outside
 no shutdown
interface f0/1
 no shutdown
interface f0/1.10
 encapsulation dot1Q 10
 ip address 10.7.1.129 255.255.255.128
 ip nat inside

! ---------- Ruta por defecto ----------
ip route 0.0.0.0 0.0.0.0 200.7.1.5

! ---------- DHCP VLAN 10 ----------
ip dhcp excluded-address 10.7.1.129 10.7.1.135
ip dhcp pool VLAN10
 network 10.7.1.128 255.255.255.128
 default-router 10.7.1.129
 dns-server 8.8.8.8

! ---------- NAT: excluir el tráfico de la VPN ----------
access-list 100 deny   ip 10.7.1.128 0.0.0.127 10.7.1.0 0.0.0.15
access-list 100 permit ip 10.7.1.128 0.0.0.127 any
ip nat inside source list 100 interface f0/0 overload

! ---------- VPN IPsec con DES ----------
crypto isakmp policy 10
 encryption des
 hash sha
 authentication pre-share
 group 2
 lifetime 86400
crypto isakmp key VPN2025-0701 address 200.7.1.2

crypto ipsec transform-set TS esp-des esp-sha-hmac
 mode tunnel

access-list 101 permit ip 10.7.1.128 0.0.0.127 10.7.1.0 0.0.0.15

crypto map CMAP 10 ipsec-isakmp
 set peer 200.7.1.2
 set transform-set TS
 set pfs group2
 match address 101

interface f0/0
 crypto map CMAP
end
write memory
```

### Puntos importantes

- La **ACL 101** define el tráfico interesante y debe ser el **espejo exacto** de los selectores de fase 2 del FortiGate.
- La **ACL 100** excluye del NAT el tráfico que debe viajar por la VPN; sin esa exclusión, el túnel no funciona.
- La clave precompartida y las propuestas (DES / SHA1 / DH 2) deben coincidir con el FortiGate.
- En la running-config del R-1 no aparecen `encryption des` ni `hash sha` dentro de `crypto isakmp policy 10`: son los valores **por defecto** de IOS y el router no los muestra. La política sí está activa (`show crypto isakmp policy`).

### Verificación

```
show crypto map
show crypto isakmp policy
show crypto isakmp sa
show crypto ipsec sa | include pkts
show ip dhcp pool
```
