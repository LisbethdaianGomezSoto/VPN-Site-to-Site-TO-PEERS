# 📕 Pruebas y verificación

[⬅️ Volver al README](../README.md)

## Plan de pruebas

| # | Prueba | Resultado esperado |
|---|---|---|
| 1 | `ip dhcp` en PC1 | IP `10.7.1.136/25`, gateway `10.7.1.129` |
| 2 | `ping 10.7.1.2` desde PC1 | Respuesta (`ttl=62`) |
| 3 | `trace 10.7.1.2` desde PC1 | Termina en `10.7.1.2` |
| 4 | `https://10.7.1.2` | Página del servidor |
| 5 | `show crypto isakmp sa` en R-1 | `QM_IDLE` |
| 6 | `show crypto ipsec sa` en R-1 | `encaps` y `decaps` subiendo juntos |
| 7 | VPN deshabilitada | `ping` y `trace` fallan |

## 1. DHCP
<img src="../images/evidencias/01-pc1-dhcp.png" width="500">

## 2. Ping y traceroute con la VPN activa
<img src="../images/evidencias/02-ping-traceroute-con-vpn.png" width="650">

### Cómo interpretar el traceroute

```
1   10.7.1.129   -> R-1 (gateway de los usuarios)
2   * * *        -> FortiGate (no responde al TTL expirado)
3   10.7.1.2     -> servidor (Destination port unreachable = llegó al destino)
```

- El túnel IPsec **no aparece como un salto adicional**: el tráfico viaja encapsulado en ESP y el ISP nunca ve las redes privadas.
- El `ttl=62` del ping confirma dos saltos de enrutamiento (R-1 y FortiGate).
- El mensaje *Destination port unreachable* lo genera el **servidor** y es la forma normal en que termina un traceroute: significa que se llegó al destino.

## 3. HTTPS
<img src="../images/evidencias/03-https-servidor.png" width="700">

## 4. Estado del túnel

```
R-1# show crypto isakmp sa
dst             src             state          conn-id slot status
200.7.1.6       200.7.1.2       QM_IDLE              1    0 ACTIVE
```

```
R-1# clear crypto sa counters
R-1# show crypto ipsec sa | include pkts
    #pkts encaps: 5, #pkts encrypt: 5, #pkts digest: 5
    #pkts decaps: 5, #pkts decrypt: 5, #pkts verify: 5
```

<img src="../images/fortigate-gui/09-monitor-ipsec.png" width="700">

## 5. Prueba negativa: sin VPN no hay comunicación

Se deshabilitan las políticas de la VPN en el FortiGate (o se apaga la interfaz del ISP hacia R-1) y se repite la prueba desde PC1:

<img src="../images/evidencias/04-sin-vpn-ping-traceroute-falla.png" width="500">

El `ping` expira y el `trace` no avanza más allá del primer salto. Al reactivar la VPN, la comunicación se restablece. Esto demuestra que **el servidor solo es alcanzable a través del túnel**.

## 6. Captura del tráfico cifrado (opcional)

En GNS3: clic derecho sobre el enlace **ISP ↔ R-1 → Start capture**. Al hacer ping desde PC1 y filtrar por `esp` en Wireshark, solo se ven paquetes **ESP** entre `200.7.1.6` y `200.7.1.2`; las IPs privadas y el contenido no son visibles.
