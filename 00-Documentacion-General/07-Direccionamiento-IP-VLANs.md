# 07 — Direccionamiento IP y VLANs (fuente de verdad)

Bloque base asignado a toda la empresa: **172.16.0.0/16** (privado, RFC 1918, rango 172.16.0.0/12). Cada una de las 12 VLANs y el enlace punto a punto usan una máscara **VLSM ajustada a su necesidad real de hosts** — no un `/24` uniforme — siguiendo la memoria de cálculo de esta sección.

## 1. Por qué un bloque tamaño Clase B, y por qué VLSM (no `/24` uniforme)

### 1.1 Metodología de dimensionamiento

Para cada VLAN, el tamaño de subred se calcula así, sin dejar margen adicional más allá de lo que exige el redondeo a una potencia de 2:

1. **Host necesarios** = conteo real (personas o dispositivos), sin inflar el número.
2. **Bits de host** = el menor `n` tal que `2ⁿ − 3 ≥ Host necesarios`.
3. **Bloque** = `2ⁿ` direcciones. **Prefijo** = `32 − n`.
4. **Disponibles** = `Bloque − 3` — se restan **red, broadcast y gateway** (no solo red y broadcast), porque en este diseño cada subred de usuarios necesita su propio gateway dedicado.

**Excepción del enlace punto a punto** (Core ↔ Router Virtual): un enlace de 2 puntos no tiene un "gateway" separado de sus propios extremos, así que ahí se aplica la convención estándar de solo `−2` (red + broadcast), no `−3` — se mantiene `/30`, 2 direcciones utilizables, una por router.

### 1.2 Por qué el bloque base es tamaño Clase B y no Clase C

Un error común es pensar que basta con mirar el tamaño de **cada VLAN por separado** para decidir la clase del bloque base. El criterio correcto es la **suma de todos los bloques VLSM que hay que alojar simultáneamente, sin que se traslapen**:

| VLAN / enlace | Bloque asignado |
|---|---|
| Ventas, Desarrollo I/T A, Desarrollo I/T B | 64 + 64 + 64 |
| Administración, Servidores, Gestión | 32 + 32 + 32 |
| Soporte I/T, Telefonía IP | 16 + 16 |
| DMZ, Cloud-Mgmt, VPN-pool | 8 + 8 + 8 |
| Enlace Core ↔ Router Virtual | 4 |
| **Suma total** | **348 direcciones** |

348 direcciones **no caben** dentro de un solo bloque de 256 direcciones (el tamaño de un bloque tipo Clase C, `/24`) — por lo tanto, siguiendo el mismo criterio que determina cuándo escalar de un rango tipo Clase C a uno tipo Clase B, este proyecto necesita un bloque base de al menos tamaño `/16` (65,536 direcciones, tamaño Clase B). `172.16.0.0/16` cumple esto con amplio margen (utilización real ≈ 0.5%), dejando espacio documentado para crecimiento sin tener que renumerar nada — el mismo principio que ya se aplica al resto del diseño (capacidad de switches, cableado).

### 1.3 Aclaración conceptual — qué significa realmente "Clase B" aquí

Técnicamente, desde que existe **CIDR (Classless Inter-Domain Routing, RFC 1518/1519, 1993)**, la "clase" de un bloque (A/B/C, definida antes por el primer octeto) ya no impone ninguna regla real sobre qué máscara se le puede aplicar — es perfectamente válido subnetear un bloque Clase A en piezas de tamaño /29, o un bloque Clase C en una sola subred de /24. Lo que hace este proyecto al hablar de "necesitar un bloque tamaño Clase B" es usar la nomenclatura de tamaño de bloque (256 = tamaño de un bloque Clase C, 65,536 = tamaño de un bloque Clase B) como una forma abreviada y estándar en la industria de referirse a "cuánto espacio de direcciones necesito reservar", no como una regla que obligue a elegir literalmente un rango que alguna vez perteneció a esa clase. `172.16.0.0/16` es, en términos estrictos de RFC 1918, parte del rango privado `172.16.0.0/12` (que agrupa los bloques que clásicamente se llamaban Clase B) — el nombre coincide con el tamaño de bloque que realmente se necesita, lo cual es intencional y no una coincidencia.

## 2. Tabla maestra de VLANs — máscaras VLSM ajustadas

| VLAN | Nombre | Área / Función | Host reales | Bloque | Prefijo | Subred | Gateway | Rango usable | Broadcast | Disponibles |
|---|---|---|---|---|---|---|---|---|---|---|
| 10 | VLAN10-ADMIN | Administración | 14 | 32 | /27 | 172.16.0.192/27 | 172.16.0.193 | .194–.222 | 172.16.0.223 | 29 |
| 20 | VLAN20-VENTAS | Ventas | 30 | 64 | /26 | 172.16.0.0/26 | 172.16.0.1 | .2–.62 | 172.16.0.63 | 61 |
| 30 | VLAN30-DEVIT-A | Desarrollo I/T (Piso 3) | 55 | 64 | /26 | 172.16.0.64/26 | 172.16.0.65 | .66–.126 | 172.16.0.127 | 61 |
| 31 | VLAN31-DEVIT-B | Desarrollo I/T (Piso 4) | 55 | 64 | /26 | 172.16.0.128/26 | 172.16.0.129 | .130–.190 | 172.16.0.191 | 61 |
| 40 | VLAN40-SOPORTE | Soporte I/T | 12 | 16 | /28 | 172.16.1.64/28 | 172.16.1.65 | .66–.78 | 172.16.1.79 | 13 |
| 50 | VLAN50-SERVERS | Servidores internos (8 VMs de servicio + hosts físicos) | 20 | 32 | /27 | 172.16.1.0/27 | 172.16.1.1 | .2–.30 | 172.16.1.31 | 29 |
| 60 | VLAN60-VOIP | Telefonía IP | 6 | 16 | /28 | 172.16.1.80/28 | 172.16.1.81 | .82–.94 | 172.16.1.95 | 13 |
| 70 | VLAN70-DMZ | Servidor Web (y correo si se publica directo) | 2 | 8 | /29 | 172.16.1.96/29 | 172.16.1.97 | .98–.102 | 172.16.1.103 | 5 |
| 80 | VLAN80-CLOUD-MGMT | Nube Privada — VR1 + SV1 | 2 | 8 | /29 | 172.16.1.104/29 | 172.16.1.105 (VR1) | .106–.110 | 172.16.1.111 | 5 |
| 90 | VLAN90-MGMT | Gestión de equipos de red (12 acceso + 2 distribución + 2 routers Core) | 16 | 32 | /27 | 172.16.1.32/27 | 172.16.1.33 | .34–.62 | 172.16.1.63 | 29 |
| 200 | VLAN200-VPN | Pool de clientes VPN remotos (WireGuard, 5 perfiles piloto) | 5 | 8 | /29 | 172.16.1.112/29 | 172.16.1.113 | .114–.118 | 172.16.1.119 | 5 |

**Rangos DHCP / estático por VLAN**:

| VLAN | Rango estático | Rango DHCP |
|---|---|---|
| 10 Admin | .194–.198 | .199–.222 |
| 20 Ventas | .2–.10 | .11–.62 |
| 30 DevIT-A | .66–.74 | .75–.126 |
| 31 DevIT-B | .130–.138 | .139–.190 |
| 40 Soporte | .66–.69 | .70–.78 |
| 50 Servers | .2–.30 (todo estático) | — |
| 60 VoIP | — | .82–.94 (DHCP option 66/150 para aprovisionamiento) |
| 70 DMZ | .98–.102 (todo estático) | — |
| 80 Cloud-Mgmt | .106–.110 (todo estático) | — |
| 90 Mgmt | .34–.62 (todo estático) | — |
| 200 VPN-pool | — | Asignado por WireGuard (no DHCP) |

## 3. Enlace punto a punto Core ↔ Nube Privada

| Enlace | Subred | R1 | VR1 |
|---|---|---|---|
| R1 ↔ VR1 (OSPF, UTP Cat 6) | 172.16.1.120/30 | 172.16.1.121 | 172.16.1.122 |

Bloque de 4 direcciones (`/30`), 2 utilizables — convención estándar para un enlace de 2 puntos, sin restar una tercera dirección de "gateway" porque ninguno de los dos routers necesita una puerta de enlace distinta de su propia interfaz.

## 4. Router-IDs OSPF (loopbacks, buena práctica para estabilidad de OSPF)

| Router | Loopback / Router-ID |
|---|---|
| R1 | 172.16.254.1/32 |
| VR1 | 172.16.254.2/32 |

Usar direcciones de loopback dedicadas (`/32`) como Router-ID es una práctica estándar de estabilidad en OSPF: a diferencia de una interfaz física, un loopback nunca cae, lo que evita que la adyacencia OSPF se reinicie si una interfaz física específica tiene una falla intermitente.

## 5. Asignaciones estáticas clave (VLAN 50 / 70 / 80)

| VM | VLAN | IP |
|---|---|---|
| vm-dhcp | 50 | 172.16.1.2 |
| vm-proxy | 50 | 172.16.1.3 |
| vm-mail | 50 | 172.16.1.4 |
| vm-monitor (Zabbix) | 50 | 172.16.1.5 |
| vm-intranet (Nextcloud) | 50 | 172.16.1.6 |
| vm-vpn (WireGuard, si no va en R1) | 50 | 172.16.1.7 |
| vm-voip | 50 | 172.16.1.8 |
| vm-web (Nginx) | 70 | 172.16.1.98 |
| SV1 (switch virtual, gestión) | 80 | 172.16.1.106 |

Quedan direcciones libres (172.16.1.9–.30 en Servidores) para los hosts físicos de virtualización adicionales del roadmap de producción, sin necesidad de re-dimensionar la subred.

## 6. Reglas de ruteo entre VLANs (resumen — detalle de firewall en documentos 03/04)

- **Todas las VLANs de usuarios (10/20/30/31/40/60)** pueden llegar a VLAN 50 (Servidores) en los puertos de servicio específicos (SMTP/IMAP, HTTP Nextcloud, DHCP relay) — no acceso total.
- **DMZ (70)** no puede iniciar conexión hacia ninguna VLAN interna.
- **Cloud-Mgmt (80)** solo es alcanzable desde VLAN 90 (Gestión) y desde R1 vía OSPF — no expuesta a usuarios finales.
- **VPN (200)** entra con los mismos permisos que la VLAN de origen del usuario remoto (mapeo por perfil WireGuard).
