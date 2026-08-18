# 12 — Briefing para cada integrante del equipo

Samuel: podés copiar la sección de cada persona y mandársela directo (WhatsApp/Discord) — cada una es autocontenida.

## Contexto compartido (para los 5)

Proyecto final de Redes 1 — diseñar e implementar la red de **Virtual Solutions** (184 usuarios, 4 pisos, Data Center Tier 4, Nube Privada SDN open source). El catedrático autorizó que lo hagamos en equipo de 5 (el PDF original decía máximo 1, ya quedó resuelto). Toda la propuesta técnica vive en esta carpeta compartida, organizada por documentos numerados — cada quien es dueño de su fase y la presenta/defiende personalmente ante el catedrático.

**Calendario real de este ciclo (2026):**

| Entrega | Fecha | Cubre |
|---|---|---|
| Entrega 1 | 23 de agosto | Fase 1 — Diseño de la red |
| Entrega 2 | 19 de septiembre | Fase 2 (Correo) + Fase 3 (LAN/WAN/VPN/Seguridad) |
| Entrega 3 | 17 de octubre | Fase 4 — Nube Privada / SDN |

El costo del equipo físico (~Q1,380–1,790) se reparte en partes iguales entre los 5 (~Q280–360 c/u) — detalle en [08-Equipo-Fisico-Presupuesto.md](08-Equipo-Fisico-Presupuesto.md).

---

## Para Melany — Fase 1: Diseño de la Red Corporativa
**Entrega 1 — 23 de agosto (la más próxima)**

### Qué hice
Armé el diseño completo de la Fase 1 en [01-Fase1-Diseno-Red-Corporativa.md](../Fase-1-Diseno-Red-Corporativa/01-Fase1-Diseno-Red-Corporativa.md): análisis de necesidades tecnológicas, análisis de requerimientos/costos/tráfico/cultura organizacional, diseño lógico, diseño físico (distribución por piso, cableado estructurado según norma TIA/EIA-568, diagrama de planta por piso, elevación del rack del Data Center), listado de materiales con precios reales de Guatemala, y referencias a las políticas de seguridad y al diseño del Data Center Tier 4.

### Documentos que son 100% tuyos (sos la responsable/experta)
- [01-Fase1-Diseno-Red-Corporativa.md](../Fase-1-Diseno-Red-Corporativa/01-Fase1-Diseno-Red-Corporativa.md) — el documento principal de tu fase
- [09-Politicas-Seguridad.md](../Fase-1-Diseno-Red-Corporativa/09-Politicas-Seguridad.md) — políticas lógicas y físicas
- [10-Data-Center-Tier4.md](../Fase-1-Diseno-Red-Corporativa/10-Data-Center-Tier4.md) — diseño del Data Center según estándares

### Documentos de apoyo (los usás pero no son tuyos para editar)
- [07-Direccionamiento-IP-VLANs.md](07-Direccionamiento-IP-VLANs.md) — tabla de VLANs (la arma Jeferson, vos la referenciás)
- [08-Equipo-Fisico-Presupuesto.md](08-Equipo-Fisico-Presupuesto.md) — presupuesto (lo lidera Luis)

### Qué te toca hacer ahora
1. Leer los 3 documentos tuyos completos y hacerlos propios — vas a ser quien los defienda.
2. Revisar que la justificación de segmentación por VLAN (en [00-Arquitectura-General.md](00-Arquitectura-General.md) sección 3) te convenza para poder explicarla con tus palabras.
3. Avisar si algo del diseño físico (distribución por piso, cableado) no te cuadra con lo que vos hubieras diseñado — todavía se puede ajustar antes del 23 de agosto.
4. Coordinar con Luis si el presupuesto de materiales (doc 08, sección 4 — rollout de producción) necesita ajuste.

**Es la entrega más próxima — tu fase queda como prioridad del equipo esta semana.**

---

## Para Sergio — Fase 4 (parte SDN/Terraform)
**Entrega 3 — 17 de octubre (todavía hay tiempo, pero es la más pesada)**

### Qué hice
Documenté la arquitectura completa de la Nube Privada en [04-Fase4-Nube-Privada-SDN.md](../Fase-4-Nube-Privada-SDN/04-Fase4-Nube-Privada-SDN.md) (Proxmox + Open vSwitch/SV1 + VyOS/VR1, sin GNS3) y armé la estructura completa de Infraestructura como Código en [05-Terraform-Ansible-IaC.md](05-Terraform-Ansible-IaC.md) — incluyendo el diseño del archivo `network-inventory.yaml` (la fuente de verdad de todo el proyecto) y ejemplos de módulos Terraform con el provider `bpg/proxmox`.

### Documentos que son tuyos
- [04-Fase4-Nube-Privada-SDN.md](../Fase-4-Nube-Privada-SDN/04-Fase4-Nube-Privada-SDN.md), secciones 3.4 y 3.5 (SV1 y VR1) y 5 (automatización)
- [05-Terraform-Ansible-IaC.md](05-Terraform-Ansible-IaC.md) — la parte de Terraform (secciones 2-4)

### Qué falta que hagas (esto todavía NO está escrito, son ejemplos/esqueleto)
1. Escribir el `network-inventory.yaml` real y completo (el ejemplo en doc 05 es solo una muestra).
2. Escribir los módulos Terraform completos (`sdn/`, `compute/`, `network-vlans/`) — el código de doc 05 es un punto de partida, no el archivo final.
3. Coordinar con Samuel: vos provisionás las VMs (Terraform), él las configura por dentro (Ansible) — necesitás pasarle las IPs/outputs de Terraform.

### No es urgente todavía, pero
El servidor de Samuel corre en un SSD externo portátil — para que puedas probar tu código Terraform sin depender de tener acceso físico al hardware, van a instalar **Tailscale** para que entres remoto (ver [05](05-Terraform-Ansible-IaC.md) sección 8.3). Avisale a Samuel cuando quieras empezar a probar para que te dé acceso.

---

## Para Luis — Fase 4 (Core físico) + Presupuesto
**Entrega 3 — 17 de octubre, pero la COMPRA de equipo debe pasar antes**

### Qué hice
Documenté el diseño del Core físico en [04-Fase4-Nube-Privada-SDN.md](../Fase-4-Nube-Privada-SDN/04-Fase4-Nube-Privada-SDN.md) (Router R1 MikroTik, switch físico, OSPF, conexión Cat 6 hacia VR1) y armé el presupuesto completo con precios reales de Guatemala en [08-Equipo-Fisico-Presupuesto.md](08-Equipo-Fisico-Presupuesto.md) (router ~Q600, switch ~Q214, cable, SSD externo, etc.).

### Documentos que son tuyos
- [04-Fase4-Nube-Privada-SDN.md](../Fase-4-Nube-Privada-SDN/04-Fase4-Nube-Privada-SDN.md), secciones 3.6 y 3.7 (Core físico, switch, host de prueba) y checklist de pruebas de conectividad (sección 6)
- [08-Equipo-Fisico-Presupuesto.md](08-Equipo-Fisico-Presupuesto.md) — sos el dueño del presupuesto

### Qué te toca hacer ahora (esto sí es urgente, aunque la entrega sea en octubre)
1. **Coordinar la compra del equipo** (router MikroTik, switch TP-Link, cable Cat 6, SSD externo + enclosure, adaptador USB-Ethernet — lista completa en doc 08) — cobrar la parte de cada quien (~Q280-360, partes iguales ya decidido).
2. Una vez llegue el router, sos quien arma el script `.rsc` de configuración de R1 (OSPF + firewall) — con ayuda de Sergio para que calce con lo que él configura del lado de VR1.
3. El día que se pruebe conectividad física, sos vos + Samuel los que validan el checklist de la sección 6 del documento 04.

**Aunque la entrega formal es en octubre, entre más rápido se compre el equipo, más tiempo de prueba real tiene el equipo — no lo dejes para último momento.**

---

## Para Jeferson — Fase 3 (LAN/WAN/VPN/Intranet/Monitoreo) + Respaldo del ambiente
**Entrega 2 — 19 de septiembre (junto con la parte de Samuel)**

### Qué hice
Documenté toda la Fase 3 en [03-Fase3-LAN-WAN-VPN-Seguridad.md](../Fase-3-LAN-WAN-VPN-Seguridad/03-Fase3-LAN-WAN-VPN-Seguridad.md) (LAN/WAN/Extranet/VPN con WireGuard/Intranet con Nextcloud/monitoreo con Zabbix/DMZ/alta disponibilidad) y armé la tabla maestra de direccionamiento en [07-Direccionamiento-IP-VLANs.md](07-Direccionamiento-IP-VLANs.md) — la fuente de verdad de VLANs y subredes que usan todos los demás documentos.

### Documentos que son tuyos
- [03-Fase3-LAN-WAN-VPN-Seguridad.md](../Fase-3-LAN-WAN-VPN-Seguridad/03-Fase3-LAN-WAN-VPN-Seguridad.md) — el documento completo
- [07-Direccionamiento-IP-VLANs.md](07-Direccionamiento-IP-VLANs.md) — la tabla de VLANs/subredes

### Qué te toca hacer ahora
1. Revisar la tabla de VLANs (doc 07) — si algo del direccionamiento no te convence, es más fácil ajustarlo ahora que cuando Sergio ya tenga Terraform escrito sobre esa base (todo el proyecto depende de este archivo).
2. Definir con Samuel quién configura qué de Nextcloud/Zabbix/WireGuard: vos diseñaste el "qué" (doc 03), falta el "cómo" en Ansible — coordinar antes de la Entrega 2.
3. Tu Entrega 2 (19 de septiembre) va junto con la de Samuel (Fase 2, correo) — avísense mutuamente el avance para que la entrega salga como un solo paquete coherente.

### Responsabilidad extra: respaldo del ambiente
Sos el punto de respaldo del proyecto — deberías tener una copia de la imagen del SSD externo de Samuel (o al menos del repositorio de código) para que, si algo le pasa a su hardware, el equipo no pierda el trabajo. Coordinen cuándo hacer esa copia.

---

## Para Samuel (vos) — Correo + Configuración de máquinas + Custodio del hardware

Este es tu resumen, más que nada para que quede completo el documento: Fase 2 (correo on-premise, [02-Fase2-Servidor-Correo.md](../Fase-2-Servidor-Correo/02-Fase2-Servidor-Correo.md)) + todos los roles de Ansible que configuran lo que corre dentro de las VMs de Sergio + sos quien tiene el hardware físico y aprueba los PRs que tocan `terraform/`/`ansible/` antes de aplicarlos de verdad. Detalle completo de tu parte en la matriz RACI de [11-Equipo-y-Responsabilidades.md](11-Equipo-y-Responsabilidades.md).
