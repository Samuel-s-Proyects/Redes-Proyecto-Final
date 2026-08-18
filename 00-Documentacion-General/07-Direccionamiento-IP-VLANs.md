# 07 — Direccionamiento IP y VLANs (fuente de verdad)

Bloque base asignado a toda la empresa: **10.10.0.0/16** (privado, RFC 1918). Se usa `/24` por VLAN de forma uniforme — con 184 endpoints totales hay margen de sobra en cada subred y mantiene el esquema simple de justificar (clase C por VLAN), consistente con lo visto en el curso. Nota de optimización con VLSM al final del documento.

## Tabla maestra de VLANs

| VLAN | Nombre | Área / Función | Dispositivos aprox. | Subred | Gateway | Rango DHCP | Rango estático |
|---|---|---|---|---|---|---|---|
| 10 | VLAN10-ADMIN | Administración | 14 | 10.10.10.0/24 | 10.10.10.1 | .100–.200 | .2–.99 |
| 20 | VLAN20-VENTAS | Ventas | 30 | 10.10.20.0/24 | 10.10.20.1 | .100–.200 | .2–.99 |
| 30 | VLAN30-DEVIT-A | Desarrollo I/T (Piso 3) | 55 | 10.10.30.0/24 | 10.10.30.1 | .100–.220 | .2–.99 |
| 31 | VLAN31-DEVIT-B | Desarrollo I/T (Piso 4) | 55 | 10.10.31.0/24 | 10.10.31.1 | .100–.220 | .2–.99 |
| 40 | VLAN40-SOPORTE | Soporte I/T | 12 | 10.10.40.0/24 | 10.10.40.1 | .100–.200 | .2–.99 |
| 50 | VLAN50-SERVERS | Servidores internos (correo, monitor, intranet, VPN, DHCP, proxy) | 12 físicos + VMs de servicio | 10.10.50.0/24 | 10.10.50.1 | — (todo estático) | .10–.199 |
| 60 | VLAN60-VOIP | Telefonía IP | 6 (crecimiento previsto) | 10.10.60.0/24 | 10.10.60.1 | .100–.200 (DHCP option 66/150 para provisioning) | .2–.99 |
| 70 | VLAN70-DMZ | Web Server (y correo si se publica directo) | — | 10.10.70.0/24 | 10.10.70.1 | — (todo estático) | .10–.50 |
| 80 | VLAN80-CLOUD-MGMT | Nube Privada — VMs internas + gestión SV1 | — | 10.10.80.0/24 | 10.10.80.1 (VR1) | — (todo estático) | .10–.50 |
| 90 | VLAN90-MGMT | Gestión de equipos de red (SNMP/SSH/Winbox/HTTPS) | — | 10.10.90.0/24 | 10.10.90.1 | — | .2–.50 |
| 200 | VLAN200-VPN | Pool de clientes VPN remotos (WireGuard) | dinámico | 10.10.200.0/24 | 10.10.200.1 | Asignado por WireGuard (no DHCP) | — |

## Enlace punto a punto Core ↔ Nube Privada

| Enlace | Subred | R1 | VR1 |
|---|---|---|---|
| R1 ↔ VR1 (OSPF, UTP Cat 6) | 10.10.254.0/30 | 10.10.254.1 | 10.10.254.2 |

## Router-IDs OSPF (loopbacks, buena práctica para estabilidad de OSPF)

| Router | Loopback / Router-ID |
|---|---|
| R1 | 10.10.255.1/32 |
| VR1 | 10.10.255.2/32 |

## Asignaciones estáticas clave (VLAN 50 / 70 / 80)

| VM | VLAN | IP |
|---|---|---|
| vm-dhcp | 50 | 10.10.50.10 |
| vm-proxy | 50 | 10.10.50.11 |
| vm-mail | 50 | 10.10.50.12 |
| vm-monitor (Zabbix) | 50 | 10.10.50.13 |
| vm-intranet (Nextcloud) | 50 | 10.10.50.14 |
| vm-vpn (WireGuard, si no va en R1) | 50 | 10.10.50.15 |
| vm-web (Nginx) | 70 | 10.10.70.10 |

## Reglas de ruteo entre VLANs (resumen — detalle de firewall en documentos 03/04)

- **Todas las VLANs de usuarios (10/20/30/31/40/60)** pueden llegar a VLAN 50 (Servidores) en los puertos de servicio específicos (SMTP/IMAP, HTTP Nextcloud, DHCP relay) — no acceso total.
- **DMZ (70)** no puede iniciar conexión hacia ninguna VLAN interna.
- **Cloud-Mgmt (80)** solo es alcanzable desde VLAN 90 (Gestión) y desde R1 vía OSPF — no expuesta a usuarios finales.
- **VPN (200)** entra con los mismos permisos que la VLAN de origen del usuario remoto (mapeo por perfil WireGuard).

## Nota sobre VLSM (optimización opcional, no obligatoria para la entrega)

Usar `/24` fijo por VLAN es simple y defendible, pero desperdicia direcciones en VLANs pequeñas (ej. VLAN 60 con 6 dispositivos usando 254 disponibles). Si se quiere sumar puntos extra mostrando dominio de VLSM, se puede resegmentar así manteniendo el mismo bloque `10.10.0.0/16`:

| VLAN | Tamaño real necesario | Máscara VLSM sugerida |
|---|---|---|
| VLAN 60 (VoIP, 6 equipos) | /28 (14 hosts) sobra | 10.10.60.0/28 |
| VLAN 90 (Gestión) | /28 sobra | 10.10.90.0/28 |
| VLAN 254 (enlace R1-VR1) | /30 (ya aplicado arriba) | 10.10.254.0/30 |

Se documenta como mejora opcional para no comprometer la claridad del esquema principal, que ya usa `/24` uniforme.
