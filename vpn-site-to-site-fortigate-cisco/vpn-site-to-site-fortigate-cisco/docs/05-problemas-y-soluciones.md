# 📓 Problemas encontrados y soluciones

[⬅️ Volver al README](../README.md)

Registro de los problemas reales que aparecieron durante el laboratorio.

| # | Síntoma | Causa | Diagnóstico | Solución |
|---|---|---|---|---|
| 1 | El ISP no hacía ping a ningún peer | Interfaces del ISP sin IP y en `shutdown` | `show ip interface brief` | Configurar IPs y `no shutdown` |
| 2 | Siguió sin haber ping al R-1 con el ISP configurado | El R-1 estaba conectado al puerto f0/0 del ISP en vez del f0/1 | `show cdp neighbors` mostraba el R-1 en `Fas 0/0` | Corregir el cableado en GNS3 (f0/1 ↔ R-1) |
| 3 | PC1 no recibía DHCP | El SW-2 perdió la VLAN 10 y el trunk | `show vlan brief` y `show interfaces trunk` vacíos | Reconfigurar VLAN y trunk, y guardar con `write memory` |
| 4 | Mensajes `DUPLEX_MISMATCH` en CDP | Autonegociación emulada en GNS3 | Aviso repetido en consola del switch | `duplex auto` / `speed auto`; es un aviso cosmético |
| 5 | El túnel subía, pero el servidor no respondía (`encaps` sin `decaps`) | El servidor tenía `10.7.1.130/28` en lugar de `10.7.1.2/28` | `ip a` en el servidor; el FortiGate respondía *Host de destino inaccesible* | Corregir IP y gateway del servidor |
| 6 | Configuración perdida al reiniciar nodos | No se había guardado | Interfaces en `administratively down` al arrancar | `write memory` en todos los equipos |
| 7 | No se podía seleccionar varias interfaces en una política | Limitación del GUI de esa versión | — | Crear una política por interfaz de entrada |
| 8 | Traceroute con muchos `* * *` | El destino aún no respondía | `ping` fallando al mismo destino | Resolver el problema 5; el trace termina en el servidor |

## Lecciones aprendidas

- Verificar **capa por capa**: cableado → IPs → enrutamiento → túnel → aplicación.
- `show cdp neighbors` es una forma rápida de detectar cables mal conectados.
- Un túnel puede estar activo (`QM_IDLE`) y aun así no pasar tráfico; los contadores `encaps` y `decaps` indican en qué sentido se pierde.
- Guardar la configuración (`write memory`) evita repetir trabajo.
