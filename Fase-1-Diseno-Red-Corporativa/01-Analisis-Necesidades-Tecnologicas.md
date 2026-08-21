# Punto 1 — Análisis de Necesidades Tecnológicas

## 1.1 Objetivo y metodología del análisis

El objetivo de este análisis es identificar, a partir del contexto de negocio de Virtual Solutions (no de una lista de tecnologías predefinida), qué necesidades tecnológicas reales tiene la organización, para que cada componente del diseño posterior (Puntos 2 a 6) responda a una necesidad identificada aquí y no a una preferencia técnica sin justificación de negocio.

Metodología aplicada — 4 pasos, estándar en un levantamiento de requerimientos de infraestructura:

1. **Perfilamiento de la organización**: tamaño, estructura, distribución física, sector.
2. **Levantamiento por stakeholder**: qué necesita cada área/rol, no solo "la empresa" en abstracto.
3. **Consolidación y priorización**: agrupar necesidades transversales, priorizar con marco MoSCoW.
4. **Trazabilidad**: cada necesidad identificada debe poder rastrearse hasta una decisión concreta en los puntos 3-6 de este documento.

## 1.2 Perfil de la organización

| Atributo | Valor | Relevancia para el diseño |
|---|---|---|
| Tamaño | 184 usuarios | Define la escala de direccionamiento, cableado y capacidad de switching |
| Distribución física | 1 edificio, 4 niveles | Define el modelo jerárquico de 3 capas (Core-Distribución-Acceso) — ver [03-Diseno-Logico.md](03-Diseno-Logico.md) |
| Composición de plantilla | 60% perfil técnico (Desarrollo I/T) | Alta tolerancia a herramientas self-hosted, pero también mayor superficie de riesgo por conocimiento técnico interno — ver [02-Requerimientos-Costos-Trafico-Cultura.md](02-Requerimientos-Costos-Trafico-Cultura.md) §4 |
| Naturaleza del negocio | Maneja recursos monetarios (dato explícito del cliente) | Eleva el estándar de disponibilidad (Tier 4) y de control de acceso exigible |
| Conectividad actual | 2 enlaces de banda ancha, 10 Mbps c/u, sin infraestructura de red corporativa formal descrita | Punto de partida greenfield — no hay legado que migrar, lo que simplifica el diseño pero exige cubrir todo desde cero |

## 1.3 Levantamiento de necesidades por stakeholder

Un error común en el diseño de redes es tratar "la empresa" como un solo interesado homogéneo. Cada área tiene necesidades distintas que el diseño debe reconciliar:

| Stakeholder | Necesidad declarada o inferida | Implicación técnica |
|---|---|---|
| **Dirección / Gerencia** | Continuidad del negocio, control de costos, cumplimiento ante manejo de fondos | Alta disponibilidad (Tier 4), presupuesto justificado con memoria de cálculo (no cifras arbitrarias), políticas de seguridad auditables |
| **Administración/Finanzas** | Confidencialidad de datos financieros, acceso controlado | VLAN separada, política de clasificación de información, MFA |
| **Ventas** | Conectividad estable con clientes externos, herramientas de videoconferencia | VLAN propia, ancho de banda priorizado para videollamadas, salida a Internet confiable |
| **Desarrollo I/T** | Acceso ágil a herramientas de desarrollo (Git, CI/CD, entornos de prueba), sin fricción operativa excesiva | VLAN de mayor capacidad, tráfico este-oeste sin cuellos de botella, posibilidad de trabajar remoto |
| **Soporte I/T** | Visibilidad total de la red para diagnóstico, acceso administrativo a todos los equipos | VLAN de gestión dedicada, acceso privilegiado auditado, herramientas de monitoreo (Zabbix) |
| **Empleados en general** | Trabajar desde fuera de la oficina sin perder acceso a herramientas internas | VPN de acceso remoto, intranet accesible, telefonía IP con movilidad |
| **Clientes externos (indirecto)** | Que el sitio/servicios expuestos de la empresa estén siempre disponibles y no comprometan sus propios datos | DMZ, Web Server segmentado, políticas de terceros |

## 1.4 Necesidades tecnológicas identificadas (consolidado)

| # | Necesidad del negocio | Traducción técnica | Prioridad (MoSCoW) |
|---|---|---|---|
| 1 | Reducir costo de telefonía | VoIP interno con **Asterisk + FreePBX** (VM `vm-voip`), VLAN 60 dedicada con QoS | Must have |
| 2 | Continuidad de Internet ante falla de proveedor | 2 enlaces WAN de **Claro** y **Tigo** (proveedores con infraestructura independiente entre sí — evita compartir el mismo punto de falla), **failover automático** en R1 (no balanceo: el enunciado pide explícitamente "redundancia", no más ancho de banda agregado) | Must have |
| 3 | Alta disponibilidad de los sistemas financieros/core | Data Center Tier 4 | Must have |
| 4 | Confidencialidad y segmentación de datos por área | VLANs + matriz de control de acceso inter-VLAN | Must have |
| 5 | Habilitar trabajo remoto sin perder control de seguridad | VPN (WireGuard) + intranet colaborativa | Must have |
| 6 | Correo corporativo propio, sin depender de terceros | Servidor de correo on-premise, anti-spam | Must have |
| 7 | Reducir intervención manual / operación propensa a error humano | Automatización (Terraform + Ansible + IA) | Should have |
| 8 | Visibilidad proactiva de la salud de la red | Monitoreo centralizado (Zabbix) | Should have |
| 9 | Gestión de identidad centralizada y auditable | Política de cuentas nominales + revisión periódica de accesos | Should have |
| 10 | Escalabilidad ante crecimiento de personal | Direccionamiento con margen (VLANs `/24`), cableado con puertos de repuesto | Should have |
| 11 | Herramientas de colaboración/e-learning para equipos distribuidos | Nextcloud, videoconferencia self-hosted | Could have |
| 12 | Autenticación multifactor generalizada a todos los usuarios (no solo administrativos) | MFA universal | Could have — roadmap, no bloqueante para esta entrega |
| 13 | Redundancia de hipervisor (clúster multi-nodo) | Proxmox en clúster de 3 nodos con Ceph | Won't have (en este proyecto) — documentado como evolución futura, fuera del alcance del laboratorio |

La clasificación MoSCoW evita el error de tratar todas las necesidades como igualmente urgentes — permite justificar, por ejemplo, por qué el proyecto sí implementa VPN y correo propio (Must have) pero deja el clúster de Proxmox como roadmap (Won't have por ahora), sin que eso se lea como una omisión no analizada.

## 1.5 Análisis de brecha (estado actual → estado deseado)

| Dimensión | Estado actual (según el enunciado) | Estado deseado (este proyecto) | Brecha que cierra el diseño |
|---|---|---|---|
| Red de datos | No descrita — se asume inexistente o informal | Red jerárquica segmentada por VLAN, 184 usuarios | Diseño lógico y físico completos (Puntos 3-4) |
| Telefonía | Líneas telefónicas tradicionales (costo alto, mencionado explícitamente) | VoIP interno sobre la misma red de datos | VLAN 60 dedicada, ver [03-Diseno-Logico.md](03-Diseno-Logico.md) |
| Correo | No descrito | Servidor propio, on-premise, anti-spam | Fase 2 completa |
| Continuidad de Internet | 2 enlaces sin indicación de failover | Failover automático entre ISP | R1 con recursive routing/PCC, ver Fase 3 |
| Disponibilidad de sistemas críticos | No descrita | Data Center Tier 4 | Punto 6 de esta fase |
| Trabajo remoto | No descrito | VPN + intranet | Fase 3 |
| Seguridad | No descrita | Política formal de 15 controles lógicos + 7 físicos | Punto 5 de esta fase |

## 1.6 Riesgos de no atender estas necesidades

Justificación de por qué estas necesidades son "Must have" y no opcionales — qué pasa si no se atienden:

- **Sin segmentación VLAN**: un incidente de seguridad en cualquier equipo (ej. Desarrollo) tiene acceso potencial irrestricto a Administración/Finanzas — riesgo directo sobre el "manejo de recursos monetarios" que el cliente señaló como crítico.
- **Sin redundancia de Internet**: la caída de un solo proveedor detiene VoIP, correo saliente, VPN y acceso de clientes — para una empresa que depende de conectividad con clientes externos (Ventas), esto es interrupción de negocio, no solo un inconveniente técnico.
- **Sin Data Center Tier 4**: cualquier falla de energía o enfriamiento no planificada puede tumbar los sistemas financieros — inaceptable dado el perfil de negocio declarado.
- **Sin VPN formal**: los empleados remotos improvisarían accesos inseguros (RDP expuesto a Internet, uso de herramientas de terceros no controladas) — mayor riesgo que el que se busca mitigar.
- **Sin automatización**: la configuración manual repetida a mano en 4+ dispositivos de red es la fuente más común de errores de configuración en redes empresariales reales (inconsistencia entre reglas de firewall, VLANs mal aplicadas) — de ahí que, aunque es "Should have" y no "Must have", se implementó desde el día uno del proyecto (ver [06-Automatizacion-con-IA.md](../00-Documentacion-General/06-Automatizacion-con-IA.md)).
