# Documento complementario — Arquitectura Final de Producción y su Equivalencia con el Laboratorio (Fase 4)

## 0. Objetivo de este documento

Los 6 documentos anteriores (00 a 06) diseñan la red que Virtual Solutions construiría **de verdad**, para sus 184 usuarios en 4 pisos. Este documento responde la pregunta que conecta esa Fase 1 con la Fase 4: **exactamente qué de todo ese diseño se implementa físicamente en el laboratorio, qué se virtualiza, y qué queda únicamente documentado** — para que quede clarísimo, dispositivo por dispositivo, que es "el mismo proyecto" a dos escalas, no dos proyectos distintos que casualmente comparten nombre.

## 1. Arquitectura de producción — resumen consolidado

| Capa | Elemento | Cantidad | Ubicación |
|---|---|---|---|
| WAN | Enlaces de Internet (Claro + Tigo) | 2 × 10 Mbps | Entran a R1 |
| Core | Router (redundante) | 2 | Data Center (Piso 2) |
| Distribución | Switch 24p+4×10G SFP+, redundante (2N) | 2 | Data Center (Piso 2) |
| Acceso | Switches 48p PoE+ | 10 (repartidos, ver [04-Diseno-Fisico.md](04-Diseno-Fisico.md) §1.1) | Pisos 1, 2, 3, 4 |
| Servidores | Hosts de virtualización | hasta 3 (clúster HA, roadmap) | Data Center (Piso 2) |
| Telefonía | Teléfonos IP | 6 (piloto, repartidos) | Pisos 1, 2, 3, 4 |
| Telefonía | PBX (Asterisk + FreePBX) | 1 VM | Data Center (Piso 2) |
| Cableado | Puntos de red (drops) | 366 | Los 4 pisos |
| Energía | UPS 2N + generador | Data Center Tier 4 | Data Center (Piso 2) |
| Red lógica | VLANs | 12 (ver [07-Direccionamiento-IP-VLANs.md](../00-Documentacion-General/07-Direccionamiento-IP-VLANs.md)) | Toda la red |

Esta es la fotografía completa de "si Virtual Solutions nos contratara para construirlo de verdad". El presupuesto total (sin energía del Data Center) es **≈ Q186,391** ([08-Equipo-Fisico-Presupuesto.md](../00-Documentacion-General/08-Equipo-Fisico-Presupuesto.md) §4).

## 2. Tabla de equivalencia — cada componente, dispositivo por dispositivo

| Componente | En producción (esta fase) | En el laboratorio (Fase 4) | Qué se demuestra en vivo |
|---|---|---|---|
| Conectividad WAN | 2 ISP reales (Claro + Tigo), failover automático | 1 enlace real de Internet (el de la casa) — el mecanismo de failover se configura en R1 aunque no haya un segundo ISP pagado para la demo | La lógica de failover en R1 (recursive routing/distancia administrativa), documentada y configurada, aunque solo se valide con un enlace real |
| Router Core (R1) | 2 routers en alta disponibilidad (VRRP/CARP) | 1 MikroTik hEX físico real, comprado | El mismo RouterOS, mismo OSPF, mismo firewall — sin el segundo router de respaldo |
| Switch de distribución (MDF) | 2× MikroTik CRS326-24S+2Q+RM, redundante (2N) | Es el **mismo rol** que cumple el único switch físico del laboratorio (TP-Link Easy Smart, VLAN 802.1Q) | Trunk 802.1Q + VLAN tagging — el mecanismo, no la capacidad total de puertos ni la redundancia |
| Switches de acceso (Pisos 1, 2, 3, 4) | 10 switches 48p PoE+, repartidos por piso (2-2-3-3) | **No existen en el laboratorio** — ni físicos ni virtuales | Nada — no se prueban físicamente; el laboratorio prueba el concepto con 2 hosts en el único switch comprado |
| Cableado estructurado | 366 puntos, certificados, con backbone de fibra | 3-4 patch cords sueltos | Conectividad punto a punto básica entre R1, el switch y los hosts de prueba |
| Teléfonos IP | 6 teléfonos físicos, PoE, repartidos en los 4 pisos ([04-Diseno-Fisico.md](04-Diseno-Fisico.md) §1.3) | No se compra hardware de teléfono para el laboratorio — fuera del checklist mínimo de pruebas de Fase 4 | Si se quiere ir más allá del mínimo: un softphone contra `vm-voip` demostraría el registro SIP, sin comprar teléfonos físicos |
| PBX / Telefonía (Asterisk + FreePBX) | 1 VM centralizada en el Data Center | Misma VM (`vm-voip`), dentro de Proxmox — noveno servicio del stack, ver [infra/network-inventory.yaml](../../infra/network-inventory.yaml) | Igual que las demás VMs de servicio: se prueba con `ansible-playbook` una vez tenga su rol escrito |
| **Nube Privada (SV1, VR1, Web, DHCP, Proxy)** | **No aplica** — no es parte del diseño físico del edificio, es un requisito aparte del enunciado | **100% real**, corre en Proxmox VE sobre el SSD externo portátil | **Esto sí se demuestra en vivo por completo** — es el corazón de la Fase 4, no una simulación de la LAN del edificio |
| Servidores físicos ("12 Servidores Físicos/Virtuales" del enunciado) | Hosts de virtualización dedicados, hasta 3 en clúster HA a futuro (roadmap, ver [06-Data-Center-Tier4.md](06-Data-Center-Tier4.md) §8) | 1 laptop + SSD externo portátil, 1 solo nodo | Las VMs corren igual (mismas herramientas, Terraform/Ansible), solo en hardware mucho más modesto y sin redundancia de cómputo |
| Data Center físico (Tier 4 completo) | 2N energía, 2N enfriamiento, extinción gaseosa, mantrap biométrico, piso elevado | 1 UPS pequeño simbólico, sin generador, sin extinción, sin mantrap | Nada de la infraestructura física del DC se construye — es 100% diseño documentado con memoria de cálculo |
| VLANs y direccionamiento IP | 12 VLANs, tabla completa, máscaras VLSM ajustadas por VLAN (bloque base `172.16.0.0/16`) | Las **mismas** VLANs, en el mismo `network-inventory.yaml` — no se inventa un esquema aparte para el laboratorio | VLAN tagging real, con 2 hosts físicos en VLANs distintas conectados al mismo switch |
| Enrutamiento dinámico (OSPF) | OSPF Area 0 en todo el Core, sin rutas estáticas | OSPF Area 0 entre R1 y VR1, sin rutas estáticas | Esto sí se demuestra en vivo, con el protocolo real — la escala es menor (2 routers, no una topología de campus completa) pero el mecanismo es idéntico |
| Políticas de seguridad (lógicas) | 15 políticas completas ([05-Politicas-Seguridad.md](05-Politicas-Seguridad.md)) | Se aplican las que tienen sentido a la escala del laboratorio (MFA en accesos administrativos, gestión de cambios vía PR, segmentación VLAN) | Las políticas que dependen de escala real (revisión semestral de accesos de 184 personas, por ejemplo) quedan como diseño, no se "actúan" en el laboratorio |

## 3. Qué implementa exactamente la Fase 4 (checklist desde la óptica de producción)

Tomando la arquitectura de producción como referencia, esto es precisamente lo que **sí** se construye en el laboratorio — ni más, ni menos:

- [x] 1 router físico real (R1) con OSPF y firewall — representa al Core de producción, sin el segundo router redundante.
- [x] 1 switch físico real con VLAN 802.1Q — representa la función de switching de la red completa (no cada switch de piso individualmente).
- [x] 2 hosts físicos de prueba, cada uno en una VLAN distinta — demuestra que la segmentación diseñada en el Punto 3 realmente funciona en hardware, no solo en el diagrama.
- [x] Nube Privada 100% funcional (SV1, VR1, y las VMs de servicio) — este componente se construye **completo**, sin reducción de escala, porque es un requisito propio de la Fase 4, no una réplica a menor tamaño de otra cosa.
- [x] Las mismas VLANs y el mismo direccionamiento IP que el diseño de producción — sin un esquema "de juguete" aparte.
- [ ] Los 10 switches de acceso de producción — **no se construyen, ni se virtualizan**.
- [ ] El cableado estructurado completo de 366 puntos — **no se instala**.
- [ ] La infraestructura física del Data Center Tier 4 (energía 2N, enfriamiento, extinción) — **no se construye**.
- [ ] Los teléfonos IP físicos — **no se compran** (fuera del mínimo exigido por el checklist de pruebas de Fase 4).

Detalle completo de la implementación de Fase 4 (con este mismo checklist ampliado) en [Fase4-Nube-Privada-SDN.md](../Fase-4-Nube-Privada-SDN/04-Fase4-Nube-Privada-SDN.md) §6.

## 4. La idea que hay que quedarse: dos preguntas, un mismo proyecto

- **Fase 1 pregunta**: *"¿Cómo se vería y cuánto costaría esta red si Virtual Solutions la construyera completa, hoy?"* — Se responde con diseño de ingeniería: diagramas, memoria de cálculo, BOM con marca/modelo/precio real.
- **Fase 4 pregunta**: *"¿Funciona el mecanismo?"* — Se responde con un laboratorio real pero deliberadamente mínimo, más la Nube Privada completa (que es un requisito aparte, no una versión reducida de la LAN del edificio).

Es el mismo proyecto en el sentido de que **comparten exactamente el mismo diseño lógico** (mismas VLANs, mismo direccionamiento, mismo protocolo de enrutamiento, misma filosofía de segmentación) — lo que cambia entre las dos fases es la **escala física**, no la arquitectura. Por eso el `network-inventory.yaml` es una sola fuente de verdad para ambas: el laboratorio no inventa su propio esquema, usa un subconjunto exacto del de producción.
