# 12 — Briefing para cada integrante del equipo

Versión resumida (1 página por persona) — para el detalle completo de decisiones, "por qué", y tareas, cada quien tiene su propio documento en `Responsables/`, que es la fuente que se mantiene actualizada. Samuel: podés copiar la sección de cada persona y mandársela directo (WhatsApp/Discord).

## Contexto compartido (para los 5)

Proyecto final de Redes 1 — diseñar e implementar la red de **Virtual Solutions** (184 usuarios, 4 pisos, Data Center Tier 4, Nube Privada SDN open source). El catedrático autorizó equipo de 5 (el PDF original decía máximo 1, ya quedó resuelto).

**Calendario real de este ciclo (2026):**

| Entrega | Fecha | Cubre |
|---|---|---|
| Entrega 1 | 23 de agosto | Fase 1 — Diseño de la red (repartida entre los 5, ver tabla abajo) |
| Entrega 2 | 19 de septiembre | Fase 2 (Correo) + Fase 3 (LAN/WAN/VPN/Seguridad) |
| Entrega 3 | 17 de octubre | Fase 4 — Nube Privada / SDN |

El costo del equipo físico (~Q1,480–1,890) se reparte en partes iguales entre los 5 (~Q296–378 c/u) — detalle en [08-Equipo-Fisico-Presupuesto.md](08-Equipo-Fisico-Presupuesto.md).

## La Fase 1 se repartió entre los 5 — cada quien con el punto más relacionado a su fase futura

| Punto de Fase 1 | Responsable | Su fase técnica después |
|---|---|---|
| 1-2. Necesidades + Requerimientos/Costos/Tráfico/Cultura | Melany | Coordina toda la Fase 1 |
| 3. Diseño lógico | Jeferson | Fase 3 (LAN/WAN/VPN) |
| 4. Diseño físico + materiales/presupuesto | Luis | Fase 4 físico + presupuesto |
| 5. Políticas de Seguridad | Samuel | Fase 2 (correo) + gobernanza |
| 6. Data Center Tier 4 | Sergio | Fase 4 SDN (su nube vive ahí) |

Por qué a cada quien le tocó su punto: [11-Equipo-y-Responsabilidades.md](11-Equipo-y-Responsabilidades.md) §1.1.

---

## Para Melany
**Fase 1, Puntos 1-2 (Entrega 1, 23 de agosto) + coordinación general de la Fase 1**

Documento completo: [Responsables/Melany.md](../Responsables/Melany.md)

Tus documentos: [01-Analisis-Necesidades-Tecnologicas.md](../Fase-1-Diseno-Red-Corporativa/01-Analisis-Necesidades-Tecnologicas.md), [02-Requerimientos-Costos-Trafico-Cultura.md](../Fase-1-Diseno-Red-Corporativa/02-Requerimientos-Costos-Trafico-Cultura.md), y el índice [00-Introduccion-y-Metodologia.md](../Fase-1-Diseno-Red-Corporativa/00-Introduccion-y-Metodologia.md). Tu diagrama: arquitectura general ([00-Arquitectura-General.md](00-Arquitectura-General.md) §2).

Como coordinadora, además leé los puntos 3-6 (de tus compañeros) para poder amarrar la presentación conjunta.

---

## Para Sergio
**Fase 1, Punto 6 (Entrega 1, 23 de agosto) + Fase 4 parte SDN/Terraform (Entrega 3, 17 de octubre)**

Documento completo: [Responsables/Sergio.md](../Responsables/Sergio.md)

Tu Fase 1: [06-Data-Center-Tier4.md](../Fase-1-Diseno-Red-Corporativa/06-Data-Center-Tier4.md) — memoria de cálculo de UPS/HVAC, diferencia real Tier3 vs Tier4. Sin diagrama propio pendiente (es memoria de cálculo, no topología).

Tu Fase 4: [04-Fase4-Nube-Privada-SDN.md](../Fase-4-Nube-Privada-SDN/04-Fase4-Nube-Privada-SDN.md) (SV1/VR1) + [05-Terraform-Ansible-IaC.md](05-Terraform-Ansible-IaC.md) (Terraform) — todavía falta escribir el código real (`network-inventory.yaml` completo y los módulos `.tf`), lo que hay hoy es ejemplo/esqueleto. Diagrama de topología pendiente de pasar a draw.io, sin apuro.

---

## Para Luis
**Fase 1, Punto 4 (Entrega 1, 23 de agosto) + Fase 4 Core físico + Presupuesto (Entrega 3, 17 de octubre — pero la COMPRA no espera)**

Documento completo: [Responsables/Luis.md](../Responsables/Luis.md)

Tu Fase 1: [04-Diseno-Fisico.md](../Fase-1-Diseno-Red-Corporativa/04-Diseno-Fisico.md) — memoria de cálculo real del cableado (405 puntos, 66 cajas, no una estimación al ojo). 2 diagramas tuyos: planta por piso y elevación de rack.

Tu Fase 4 + presupuesto: [04-Fase4-Nube-Privada-SDN.md](../Fase-4-Nube-Privada-SDN/04-Fase4-Nube-Privada-SDN.md) §3.6-3.7 (Core físico) y [08-Equipo-Fisico-Presupuesto.md](08-Equipo-Fisico-Presupuesto.md) (sos el dueño). **Coordinar la compra del equipo cuanto antes** — router MikroTik, switch TP-Link Easy Smart (VLAN, no el TL-SG105 simple), cable, SSD externo — cobrar la parte de cada quien (~Q296-378 c/u).

---

## Para Jeferson
**Fase 1, Punto 3 (Entrega 1, 23 de agosto) + Fase 3 completa (Entrega 2, 19 de septiembre) + Respaldo del ambiente**

Documento completo: [Responsables/Jeferson.md](../Responsables/Jeferson.md)

Tu Fase 1: [03-Diseno-Logico.md](../Fase-1-Diseno-Red-Corporativa/03-Diseno-Logico.md) — los 4 switches de la red completa (distribución + 3 IDF) con sus VLANs. Diagrama tuyo, es el más citado de la fase.

Tu Fase 3: [03-Fase3-LAN-WAN-VPN-Seguridad.md](../Fase-3-LAN-WAN-VPN-Seguridad/03-Fase3-LAN-WAN-VPN-Seguridad.md) + tabla maestra [07-Direccionamiento-IP-VLANs.md](07-Direccionamiento-IP-VLANs.md) — la fuente de verdad de todo el proyecto, revisala bien antes de que Sergio construya Terraform sobre esa base.

**Responsabilidad extra**: sos el punto de respaldo del ambiente — coordiná con Samuel la copia del SSD/repositorio.

---

## Para Samuel (vos)
**Fase 1, Punto 5 (Entrega 1, 23 de agosto) + Fase 2 correo (Entrega 2, 19 de septiembre) + configuración de máquinas + custodio del hardware**

Documento completo: [Responsables/Samuel.md](../Responsables/Samuel.md)

Tu Fase 1: [05-Politicas-Seguridad.md](../Fase-1-Diseno-Red-Corporativa/05-Politicas-Seguridad.md) — 15 políticas lógicas + 7 físicas, marco ISO 27001/NIST. Sin diagrama propio pendiente.

Tu Fase 2 + rol transversal: [02-Fase2-Servidor-Correo.md](../Fase-2-Servidor-Correo/02-Fase2-Servidor-Correo.md) (código ya escrito en `infra/ansible/roles/mailserver/`) + todos los roles de Ansible que configuran lo que corre dentro de las VMs de Sergio + custodio del hardware físico + aprobador de PRs que tocan `terraform/`/`ansible/`. Detalle completo en la matriz RACI de [11-Equipo-y-Responsabilidades.md](11-Equipo-y-Responsabilidades.md).
