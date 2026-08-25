# 07 — Direccionamiento IP y VLANs (fuente de verdad)

Bloque base asignado a toda la empresa: **10.10.0.0/16** (privado, RFC 1918). Se usa `/24` por VLAN de forma uniforme — con 184 endpoints totales hay margen de sobra en cada subred. Nota de optimización con VLSM al final del documento.

## Justificación del bloque de direccionamiento y su alcance frente a 184 endpoints

Con solo 184 endpoints, el tamaño del bloque elegido (`10.10.0.0/16`, y más ampliamente el rango privado `10.0.0.0/8` del que se deriva) es deliberadamente mayor de lo que el conteo actual de usuarios necesitaría en una lectura estrictamente literal. Esa holgura es intencional, no un descuido, por tres razones concretas:

1. **Elección del rango privado (`10.0.0.0/8`) sobre las otras dos opciones de RFC 1918**: existen tres bloques privados disponibles — `10.0.0.0/8` (16.7 millones de direcciones), `172.16.0.0/12` (1 millón) y `192.168.0.0/16` (65,536, el rango que casi todo router doméstico/SOHO trae configurado de fábrica). Se descarta `192.168.0.0/16` específicamente porque es el más propenso a colisión: si un empleado se conecta por VPN desde su casa y su propio router también usa `192.168.1.0/24`, el cliente VPN y la red corporativa "compiten" por la misma subred y el enrutamiento falla. `10.0.0.0/8` es, en la práctica, el rango con menor probabilidad de chocar con una red doméstica o de un tercero.
2. **Espacio privado no es un recurso escaso — a diferencia de IPv4 pública**: reservar más espacio del que se usa hoy no tiene costo, porque nadie más puede "quedarse sin" direcciones privadas por que una organización tome un bloque grande. De los 16.7 millones de direcciones de `10.0.0.0/8`, este proyecto usa activamente 65,536 (`10.10.0.0/16`) y, dentro de ese `/16`, solo una fracción de las 12 subredes `/24` (3,072 direcciones utilizables en total) están realmente asignadas a los 184 endpoints — la relación exacta se muestra en la tabla siguiente.
3. **Margen de crecimiento sin rediseño**: al quedar prácticamente todo `10.0.0.0/8` libre por fuera del `/16` en uso, la empresa puede crecer (una segunda sede, una nueva VLAN, interconexión con otra oficina) usando el mismo espacio de direccionamiento sin jamás necesitar renumerar lo que ya existe ni invadir el rango `172.16.0.0/12` o `192.168.0.0/16`.

**Visibilidad real del segmento — cuánto se usa vs. cuánto se reservó**:

| Nivel | Direcciones disponibles | Direcciones realmente necesarias (184 endpoints) | % en uso |
|---|---|---|---|
| Por VLAN (`/24`, ej. VLAN 20 – Ventas, 30 usuarios) | 254 utilizables | 30 | ≈ 12% |
| Bloque en uso (`10.10.0.0/16`, 12 VLANs) | 3,072 utilizables (12 × 254) | 184 | ≈ 6% |
| Rango privado completo (`10.0.0.0/8`) | ~16.7 millones | 184 | < 0.002% |

El margen visible en esta tabla no es sobre-dimensionamiento sin criterio: es el mismo principio que ya se aplica al resto del diseño (capacidad de switches, cableado, direccionamiento) — dejar cabida documentada para crecimiento, sin que eso se confunda con "capacidad ilimitada" que no requiere monitoreo (ver nota de escalabilidad en el Punto 1 del proyecto).

**Aclaración conceptual — por qué esto NO es "direccionamiento clase C"**: todo el bloque `10.0.0.0/8` pertenece, por definición, a la **Clase A** (el primer octeto 10 cae en el rango 1–126 que identifica Clase A) — esto no cambia sin importar qué máscara se le aplique después. El "class-based addressing" (Clases A/B/C/D/E, definido por el primer octeto) fue reemplazado por **CIDR (Classless Inter-Domain Routing, RFC 1518/1519, 1993)** precisamente para eliminar la rigidez de que una organización solo pudiera pedir bloques de tamaño fijo /8, /16 o /24. Que cada VLAN de este proyecto use una máscara `/24` (255.255.255.0) es una decisión de **subnetting classless** — el tamaño de subred se elige por necesidad de hosts, no por la clase original del bloque — no una propiedad "clase C" del espacio de direcciones. Confundir "máscara /24" con "Clase C" es un error conceptual común (la clase describe el bloque original antes de subnetear; la máscara describe cómo se subneteó *después*, algo que las clases nunca contemplaron). En este proyecto: bloque `10.10.0.0/16` = subred de la Clase A privada `10.0.0.0/8` (RFC 1918), subneteada de forma classless en 12 subredes `/24` mediante VLSM/CIDR.

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
