# Proyecto Final — Redes de Computadoras 1
## Virtual Solutions — Propuesta técnica completa

El PDF original indica "Máximo Integrantes del grupo: 1", pero el catedrático **autorizó un grupo de 5 personas para este ciclo** (ver [11-Equipo-y-Responsabilidades.md](00-Documentacion-General/11-Equipo-y-Responsabilidades.md) para la división de trabajo y el respaldo de esa autorización). Propuesta real y ejecutable, pensada para implementarse con **la laptop de Samuel + un SSD externo portátil** como Nube Privada (Proxmox VE) y **equipo físico real de bajo costo comprado en Guatemala** para el Core del Data Center — sin GNS3.

Filosofía de la propuesta: **una sola fuente de verdad** (inventario de red en YAML/Markdown) desde la cual se genera todo lo demás — Terraform, Ansible, diagramas y documentación — usando IA (Claude Code) como "generador y verificador", para que el trabajo manual se reduzca a decisiones de diseño y revisión, no a escribir configuración a mano.

---

## Estructura de la carpeta

```
Proyecto-Redes-VirtualSolutions/
├── README.md                          ← estás acá: índice general
├── 00-Documentacion-General/          ← decisiones y arquitectura que aplican a todo el proyecto
├── Fase-1-Diseno-Red-Corporativa/      ← Entrega 1 — 23 de agosto de 2026
├── Fase-2-Servidor-Correo/             ← Entrega 2 — 19 de septiembre de 2026
├── Fase-3-LAN-WAN-VPN-Seguridad/       ← Entrega 2 — 19 de septiembre de 2026
├── Fase-4-Nube-Privada-SDN/            ← Entrega 3 — 17 de octubre de 2026
└── Responsables/                      ← un documento por persona: su módulo, sus decisiones, sus tareas
```

## Equipo

| Integrante | Responsabilidad principal | Su módulo explicado a fondo |
|---|---|---|
| **Samuel** | Correo (Fase 2) + configuración de máquinas (Ansible) + custodio del hardware | [Responsables/Samuel.md](Responsables/Samuel.md) |
| Sergio | Infraestructura como código / SDN (Terraform, Proxmox, SV1, VR1) — Fase 4 | [Responsables/Sergio.md](Responsables/Sergio.md) |
| Melany | Diseño de red corporativa y Data Center — Fase 1 + Políticas de seguridad | [Responsables/Melany.md](Responsables/Melany.md) |
| Luis | Core físico y networking — Fase 4 (parte física) + Presupuesto/equipo | [Responsables/Luis.md](Responsables/Luis.md) |
| Jeferson | LAN/WAN/VPN/Intranet/Monitoreo — Fase 3 + respaldo del ambiente | [Responsables/Jeferson.md](Responsables/Jeferson.md) |

Cada archivo en `Responsables/` es autocontenido: explica el módulo de esa persona, **cada decisión técnica que se tomó para llegar ahí y por qué**, qué diagramas le faltan pasar a una herramienta visual (si aplica), y sus próximos pasos concretos. Están escritos para copiar y mandar directo por chat.

Matriz RACI completa (quién es responsable/aprueba/consulta en cada documento) en [11-Equipo-y-Responsabilidades.md](00-Documentacion-General/11-Equipo-y-Responsabilidades.md).

## Cronograma real (ciclo 2026 — 3 entregas, no 4)

| Entrega | Fecha | Cubre | Estado |
|---|---|---|---|
| **Entrega 1** | **23 de agosto** | Fase 1 — Diseño de la red corporativa | 🔵 En curso |
| Entrega 2 | 19 de septiembre | Fase 2 (Correo) + Fase 3 (LAN/WAN/VPN/Seguridad) | Diseño ya definido, falta implementar |
| Entrega 3 | 17 de octubre | Fase 4 — Nube Privada / SDN | Diseño ya definido, falta implementar |

Aunque solo la Entrega 1 es inminente, **las Fases 2, 3 y 4 ya están completamente diseñadas y documentadas** — se armaron con visibilidad total desde el inicio para que nadie tenga que rediseñar nada de último momento, solo ejecutar (escribir el código de Terraform/Ansible, comprar equipo, configurar servicios) sobre decisiones que ya están tomadas y no van a cambiar.

---

## 00 — Documentación General (decisiones que aplican a todo el proyecto)

| Documento | Contenido |
|---|---|
| [Arquitectura-General.md](00-Documentacion-General/00-Arquitectura-General.md) | Visión completa, stack tecnológico elegido y por qué, diagrama general de la solución |
| [Terraform-Ansible-IaC.md](00-Documentacion-General/05-Terraform-Ansible-IaC.md) | Estructura del repo de Infraestructura como Código + estrategia de SSD portátil |
| [Automatizacion-con-IA.md](00-Documentacion-General/06-Automatizacion-con-IA.md) | Cómo usar Claude Code para generar y mantener todo lo anterior |
| [Direccionamiento-IP-VLANs.md](00-Documentacion-General/07-Direccionamiento-IP-VLANs.md) | Tabla maestra de VLANs y subredes (fuente de verdad del proyecto) |
| [Equipo-Fisico-Presupuesto.md](00-Documentacion-General/08-Equipo-Fisico-Presupuesto.md) | Lista de materiales con precios reales en Guatemala (Q) |
| [Equipo-y-Responsabilidades.md](00-Documentacion-General/11-Equipo-y-Responsabilidades.md) | Equipo, matriz RACI, colaboración remota, cronograma, pendientes |
| [Briefing-Equipo.md](00-Documentacion-General/12-Briefing-Equipo.md) | Versión resumida (una página) del briefing de cada persona |

## Fase 1 — Diseño de la Red Corporativa (2 pts) — Entrega 1, 23 de agosto

| Documento | Contenido |
|---|---|
| [Fase1-Diseno-Red-Corporativa.md](Fase-1-Diseno-Red-Corporativa/01-Fase1-Diseno-Red-Corporativa.md) | Análisis de necesidades, diseño lógico/físico, cableado, diagrama de planta, materiales |
| [Politicas-Seguridad.md](Fase-1-Diseno-Red-Corporativa/09-Politicas-Seguridad.md) | Políticas lógicas y físicas de seguridad |
| [Data-Center-Tier4.md](Fase-1-Diseno-Red-Corporativa/10-Data-Center-Tier4.md) | Diseño del Data Center según TIA-942 / Uptime Institute |

## Fase 2 — Servidor de Correo (4 pts) — Entrega 2, 19 de septiembre

| Documento | Contenido |
|---|---|
| [Fase2-Servidor-Correo.md](Fase-2-Servidor-Correo/02-Fase2-Servidor-Correo.md) | Correo on-premise open source, anti-spam, multi-dominio, diagrama de flujo |

## Fase 3 — LAN/WAN/VPN/Seguridad (2 pts) — Entrega 2, 19 de septiembre

| Documento | Contenido |
|---|---|
| [Fase3-LAN-WAN-VPN-Seguridad.md](Fase-3-LAN-WAN-VPN-Seguridad/03-Fase3-LAN-WAN-VPN-Seguridad.md) | LAN/WAN/Extranet/VPN/Intranet, monitoreo, DMZ (con diagrama de zonas), alta disponibilidad |

## Fase 4 — Nube Privada / SDN (7 pts) — Entrega 3, 17 de octubre

| Documento | Contenido |
|---|---|
| [Fase4-Nube-Privada-SDN.md](Fase-4-Nube-Privada-SDN/04-Fase4-Nube-Privada-SDN.md) | Implementación SDN, DHCP/Web/Proxy, Core físico, OSPF, checklist de pruebas |

---

## Diagramas pendientes — inventario completo (solo falta pasarlos a herramienta visual)

Todos los diagramas de este proyecto ya están diseñados y completos como código (Mermaid), embebidos en sus documentos. **No queda ninguna decisión de diseño pendiente** — el trabajo que falta es puramente de forma: copiar la estructura a una herramienta visual profesional (recomendado: **draw.io / diagrams.net**, gratis, permite pegar Mermaid y convertirlo a forma editable con íconos de red).

| Diagrama | Dónde está el Mermaid ya definido | Responsable de pasarlo a visual |
|---|---|---|
| Arquitectura general de la solución | [00-Arquitectura-General.md](00-Documentacion-General/00-Arquitectura-General.md) §2 | Melany |
| Planta por piso (4 niveles) | [Fase1](Fase-1-Diseno-Red-Corporativa/01-Fase1-Diseno-Red-Corporativa.md) §4.1 | Melany |
| Elevación de rack del Data Center | [Fase1](Fase-1-Diseno-Red-Corporativa/01-Fase1-Diseno-Red-Corporativa.md) §4.2 (tabla → convertir a rack visual) | Melany |
| Flujo de correo (intra/inter-dominio + anti-spam) | [Fase2](Fase-2-Servidor-Correo/02-Fase2-Servidor-Correo.md) §2.1 | Samuel |
| Zonas DMZ / Internet / LAN interna | [Fase3](Fase-3-LAN-WAN-VPN-Seguridad/03-Fase3-LAN-WAN-VPN-Seguridad.md) §4 | Jeferson |
| Topología física + SDN (R1, switch, VR1, SV1) | [Fase4](Fase-4-Nube-Privada-SDN/04-Fase4-Nube-Privada-SDN.md) §2 | Sergio |

No hace falta un "diagramador" único dedicado — cada quien pasa a visual el diagrama de su propia fase, porque ya la conoce a fondo y es quien la va a presentar. Luis no tiene diagrama propio; su rol es validar que la parte física de los diagramas de Melany y Sergio quede fiel a lo que realmente compra e instala.

---

## Resumen de la empresa (contexto del caso)

**Virtual Solutions** — edificio de 4 niveles.

| Área | Usuarios |
|---|---|
| Administración | 14 |
| Ventas | 30 |
| Desarrollo I/T | 110 |
| Soporte I/T | 12 |
| Servidores (físicos/virtuales) | 12 |
| Telefonía IP | 6 |
| **Total endpoints** | **184** |

- Internet: 2 enlaces de 10 Mbps, proveedores distintos, con redundancia.
- Data Center **Tier 4** (alta disponibilidad, sin punto único de falla).
- Requiere VoIP en LAN, correo, servidores web, VPN, trabajo remoto, e-learning/colaboración.
- Segmentación obligatoria por VLAN (ingeniería separada de administrativo, con justificación).
- Nube Privada SDN 100% open source: Web Server, DHCP, Proxy, con switch virtual SV1 y router virtual VR1, conectada a un Router físico Core R1.

## Stack tecnológico elegido (resumen — detalle en [Arquitectura-General.md](00-Documentacion-General/00-Arquitectura-General.md))

| Componente | Tecnología | Motivo |
|---|---|---|
| Hipervisor / Nube Privada | **Proxmox VE** (en SSD externo portátil) | 100% open source, API completa (Terraform-able), corre bien en laptops de 8-16GB RAM |
| Switch virtual (SV1) | **Open vSwitch (OVS)** | Estándar de facto para SDN open source, soporta VLAN/trunk |
| Router virtual (VR1) | **VyOS** | Router Linux open source, soporta OSPF/BGP dinámico, se automatiza con Ansible |
| Core físico (R1) | **MikroTik RouterOS (hEX RB750Gr3)** | Barato en Guatemala (~Q600), soporta OSPF, VLANs, firewall |
| Switch físico | **TP-Link TL-SG105** (no administrable) | Cumple el requisito de la Fase 4 sin pagar de más (~Q214) |
| IaC | **Terraform** (provider `bpg/proxmox`) + **Ansible** | Terraform provisiona VMs/red, Ansible configura servicios dentro |
| Correo | **Postfix + Dovecot + rspamd** | Open source, on-premise, anti-spam integrado |
| Monitoreo | **Zabbix** | Monitoreo de red + servidores en una sola plataforma open source |
| Intranet/colaboración | **Nextcloud** | Archivos, calendario, trabajo en equipo remoto |
| VPN acceso remoto | **WireGuard** | Simple, rápido, open source, ideal para trabajo a distancia |
| IPAM / documentación de red | **NetBox** | Fuente de verdad de IPs/VLANs, se integra con Terraform |

## Próximo paso sugerido

1. Cada quien lee su documento en `Responsables/` y el/los documento(s) técnico(s) que le corresponden.
2. Melany pasa sus 3 diagramas a draw.io esta semana (Entrega 1 es la más próxima).
3. Luis coordina la compra del equipo físico — mientras antes, más tiempo de prueba real.
4. Samuel prepara el SSD externo con Proxmox VE instalado.
5. Sergio y Jeferson arrancan el `network-inventory.yaml` en conjunto (la tabla de VLANs de Jeferson es el insumo).
