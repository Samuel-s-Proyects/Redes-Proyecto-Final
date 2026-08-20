# 11 — Equipo y Responsabilidades

## 0. Nota importante sobre el tamaño del equipo

El PDF original ("Proyecto Final Redes-2025") indica en la página 5: **"Máximo Integrantes del grupo: 1."** El equipo confirma que el catedrático **autorizó grupo de 5 personas para este ciclo**, lo cual reemplaza esa restricción. Se recomienda **guardar la autorización por escrito** (correo, mensaje, anuncio del curso) como respaldo, por si en algún momento se pide justificar el tamaño del equipo ante coordinación académica.

## 1. Integrantes

| # | Nombre | Rol principal |
|---|---|---|
| 1 | **Samuel** | Configuración de máquinas (Ansible) + Fase 2 (Correo on-premise) — infraestructura física base |
| 2 | Sergio | Infraestructura como código / SDN (Terraform, Proxmox, SV1, VR1) — Fase 4 (parte virtual) |
| 3 | Melany | Diseño de red corporativa y Data Center — Fase 1 + Políticas de seguridad |
| 4 | Luis | Core físico y networking — Fase 4 (parte física) + Presupuesto/equipo |
| 5 | Jeferson | LAN/WAN/VPN/Intranet/Monitoreo — Fase 3 |

## 1.1 División de la Fase 1 (Entrega 1) entre los 5

La Fase 1 tiene 6 puntos exigidos por el enunciado (ver [índice](../Fase-1-Diseno-Red-Corporativa/00-Introduccion-y-Metodologia.md)) — se repartieron entre los 5, cada uno con el punto que mejor conecta con lo que ya va a hacer más adelante en el proyecto, para que no esté aprendiendo desde cero un tema que después abandona:

| Punto de Fase 1 | Responsable | Por qué le toca a esa persona |
|---|---|---|
| 1-2. Necesidades + Requerimientos/Costos/Tráfico/Cultura | **Melany** | Es el análisis que sostiene todo el resto del diseño — Melany coordina la Fase 1 completa, tiene sentido que sea quien primero entienda el negocio a fondo |
| 3. Diseño lógico (VLANs, switches, trunk, direccionamiento) | **Jeferson** | Es la continuación exacta de lo que hace en Fase 3 (tabla de VLANs, LAN/WAN) — mismo tema, más adelante en el tiempo |
| 4. Diseño físico (cableado, planta, materiales, presupuesto) | **Luis** | Es la continuación de lo que hace en Fase 4 (Core físico, compra de equipo, presupuesto) — ya va a manejar materiales y costos, aquí lo hace a escala de toda la empresa |
| 5. Políticas de Seguridad | **Samuel** | Encaja con su rol de aprobador de PRs/gobernanza del proyecto y con la seguridad del correo (Fase 2) — ya piensa en controles de acceso y cifrado en su propia fase |
| 6. Diseño de Data Center (Tier 4) | **Sergio** | Su Nube Privada (Fase 4) vive físicamente en el Data Center que aquí se diseña — entender los requisitos de energía/enfrionamiento/disponibilidad del sitio donde corren sus VMs es directamente relevante para su propio módulo |

Melany sigue siendo la coordinadora general de la Fase 1 (arma el documento 00 de índice/metodología y da seguimiento a que los 6 puntos queden consistentes entre sí), pero cada quien es el experto y defensor de su propio punto — igual que ya se acordó para las Fases 2-4 (cada quien defiende lo suyo).

## 2. Matriz de responsabilidades (RACI) por documento

**R** = Responsable de escribir/implementar · **A** = Aprueba/revisa antes de entregar · **C** = Consultado (aporta pero no es dueño) · **I** = Informado

| Documento / Componente | Samuel | Sergio | Melany | Luis | Jeferson |
|---|---|---|---|---|---|
| [00 Arquitectura General](00-Arquitectura-General.md) | A | C | C | C | C |
| **Fase 1** — [00 Índice/Metodología](../Fase-1-Diseno-Red-Corporativa/00-Introduccion-y-Metodologia.md) | I | I | **R** (coordina) | I | I |
| **Fase 1** — [01 Necesidades](../Fase-1-Diseno-Red-Corporativa/01-Analisis-Necesidades-Tecnologicas.md) + [02 Requerim./Costos/Tráfico/Cultura](../Fase-1-Diseno-Red-Corporativa/02-Requerimientos-Costos-Trafico-Cultura.md) | I | I | **R** | C | I |
| **Fase 1** — [03 Diseño Lógico](../Fase-1-Diseno-Red-Corporativa/03-Diseno-Logico.md) | I | I | C | I | **R** |
| **Fase 1** — [04 Diseño Físico](../Fase-1-Diseno-Red-Corporativa/04-Diseno-Fisico.md) | I | I | C | **R** | I |
| **Fase 1** — [05 Políticas de Seguridad](../Fase-1-Diseno-Red-Corporativa/05-Politicas-Seguridad.md) | **R** | I | C | I | I |
| **Fase 1** — [06 Data Center Tier 4](../Fase-1-Diseno-Red-Corporativa/06-Data-Center-Tier4.md) | I | **R** | C | I | I |
| [02 Fase 2 — Servidor de Correo](../Fase-2-Servidor-Correo/02-Fase2-Servidor-Correo.md) | **R** | I | I | I | C |
| [03 Fase 3 — LAN/WAN/VPN/Seguridad](../Fase-3-LAN-WAN-VPN-Seguridad/03-Fase3-LAN-WAN-VPN-Seguridad.md) | C | I | I | I | **R** |
| [04 Fase 4 — Nube Privada/SDN](../Fase-4-Nube-Privada-SDN/04-Fase4-Nube-Privada-SDN.md) | C (Ansible de las VMs) | **R** (Terraform/SDN) | I | **R** (Core físico/OSPF) | I |
| [05 Terraform + Ansible (IaC)](05-Terraform-Ansible-IaC.md) | **R** (roles Ansible) | **R** (módulos Terraform) | I | C | C |
| [06 Automatización con IA](06-Automatizacion-con-IA.md) | A | C | C | C | C |
| [07 Direccionamiento IP/VLANs](07-Direccionamiento-IP-VLANs.md) | C | C | C | I | **R** |
| [08 Equipo Físico/Presupuesto](08-Equipo-Fisico-Presupuesto.md) | C | I | I | **R** | C |
| Hardware físico (Proxmox host, R1, switch, cableado) | **R** (custodio del equipo) | I | I | **R** (compra/configuración de R1) | I |
| Pruebas de conectividad end-to-end (Fase 4, sección 6) | C | C | I | C | I → **R compartido entre Sergio y Luis**, Samuel valida desde el lado de las VMs |
| Consolidación final / entrega | **A** | C | C | C | C |

Nota: Samuel queda como **aprobador final** de la mayoría de documentos porque es quien tiene acceso físico al hardware y corre el ambiente real — es el único que puede validar en la práctica que lo que cada quien diseñó realmente funciona quando se integra.

## 3. División concreta de la Fase 4 (la más grande, 7 pts)

Como es la fase con más peso y más piezas, se divide explícitamente en 3 bloques:

| Bloque | Responsable | Incluye |
|---|---|---|
| **Infraestructura virtual (Terraform)** | Sergio | Módulo `sdn/` (bridge OVS = SV1, VM VyOS = VR1), módulo `compute/` (VMs Web/DHCP/Proxy), `network-inventory.yaml` |
| **Configuración de servicios (Ansible)** | Samuel | Roles `webserver`, `dhcp`, `proxy`, `vyos_router` — lo que corre *dentro* de cada VM que Sergio provisiona |
| **Core físico y enrutamiento** | Luis | Compra/configuración de R1 (MikroTik), switch físico, cableado Cat 6, script `.rsc` de OSPF y firewall de R1 |

Este orden de dependencia importa: **Sergio provisiona → Samuel configura → Luis conecta el lado físico y valida OSPF** — es el flujo natural de `terraform apply` → `ansible-playbook` → prueba de conectividad física.

## 4. Flujo de colaboración (esto faltaba definir — ya resuelto aquí)

### 4.1 Repositorio de código
- Un solo repositorio Git privado (GitHub/GitLab) con la estructura de [05-Terraform-Ansible-IaC.md](05-Terraform-Ansible-IaC.md).
- Rama `main` protegida; cada quien trabaja en su rama (`feature/fase1-diseno`, `feature/fase4-terraform`, etc.) y hace Pull Request.
- **Samuel aprueba los PRs que tocan `terraform/` o `ansible/`** (porque es quien puede probarlos contra el hardware real antes de aceptarlos); el resto de documentos los puede aprobar el dueño de la fase correspondiente.

### 4.2 Acceso remoto al ambiente de Samuel
Como el Proxmox físico vive en casa de Samuel, el resto del equipo necesita forma de probar su parte sin estar presencialmente ahí:
- Se recomienda instalar **Tailscale** (gratis, basado en WireGuard, un solo comando) en el servidor Proxmox de Samuel — crea una red privada tipo VPN entre las laptops de los 5 sin necesidad de abrir puertos en el router de la casa.
- Con eso, Sergio puede aplicar Terraform, Samuel/Jeferson pueden correr Ansible, y Luis puede entrar por SSH/Winbox al MikroTik, todo remoto.
- **Nota**: esta VPN de colaboración (Tailscale) es una herramienta de trabajo en equipo, distinta del WireGuard que se implementa como parte del proyecto en la Fase 3 (ese es para los empleados de Virtual Solutions, no para el equipo de desarrollo) — no mezclar ambos en la documentación de entrega.

### 4.3 Comunicación
- Canal de chat único (WhatsApp/Discord) para coordinación diaria.
- Reunión corta semanal (15-20 min) para revisar avance por fase — se sugiere sincronizar con el ritmo de PRs del repo (revisar qué se mergeó esa semana).

## 5. Riesgo: Samuel es punto único de falla del hardware físico

Solo Samuel tiene acceso directo al Proxmox/router/switch físicos. Mitigación:
- **Todo el ambiente es reproducible por código** (ver documento 05) — si el hardware de Samuel falla, cualquier otro integrante con acceso a un equipo con virtualización puede levantar el ambiente desde cero con `terraform apply` + `ansible-playbook`, usando el mismo repo. Esto es, en la práctica, el respaldo del proyecto.
- Se recomienda que **al menos Sergio tenga también Proxmox instalado en algún equipo propio** (aunque sea con recursos limitados) para poder probar sus módulos de Terraform sin depender 100% de la disponibilidad de Samuel.
- El SSD externo con el ambiente completo (ver [05](05-Terraform-Ansible-IaC.md) sección de portabilidad) debe tener **una copia de respaldo** (imagen del disco) en manos de una segunda persona del equipo — **Jeferson** es el punto de respaldo designado.

## 6. Decisiones del equipo (confirmadas)

| Punto | Decisión |
|---|---|
| Costo del equipo físico (~Q1,380–1,790) | **Partes iguales entre los 5** (~Q276–358 c/u) |
| Quién expone cada fase | **Cada quien defiende la fase de la que es responsable** — cada uno se vuelve el experto de su área (ver matriz RACI, sección 2) |
| Punto de respaldo del ambiente (además de Samuel) | **Jeferson** — tiene copia de respaldo del SSD/imagen del ambiente |
| Confirmar autorización del grupo de 5 por escrito | Pendiente — guardar el correo/mensaje del catedrático (ver sección 0) |
| Plan de contingencia si alguien no entrega a tiempo | **No definido por ahora** — se revisará más adelante si hace falta |

## 7. Cronograma real de entregas (ciclo 2026)

El catedrático definió **3 entregas** este ciclo (no 4 como en el PDF de 2025) — la Entrega 2 fusiona lo que originalmente eran Fase 2 y Fase 3:

| Entrega | Fecha 2026 | Cubre | Estado |
|---|---|---|---|
| **Entrega 1** | **23 de agosto** | Fase 1 — Diseño de la red corporativa | 🔵 En curso — es la que se está preparando ahora |
| Entrega 2 | 19 de septiembre | Fase 2 (Correo) **+** Fase 3 (LAN/WAN/VPN/Seguridad) fusionadas | Pendiente |
| Entrega 3 | 17 de octubre | Fase 4 — Nube Privada / SDN | Pendiente |

Los documentos de esta carpeta (01 a 04) se mantienen separados por fase técnica tal como los pidió originalmente el enunciado — la fusión de Fase 2+3 en la Entrega 2 es solo una decisión de **calendario de entrega**, no cambia el contenido técnico de cada fase.

## 8. Pendientes que aún faltan por decidir

- [ ] Confirmar con el catedrático — y guardar evidencia escrita — de la autorización del grupo de 5 (ver sección 0)
- [ ] Plan de contingencia si alguien no entrega su parte a tiempo (se dejó pendiente a propósito, revisar antes de la Entrega 2)
