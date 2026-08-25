# Módulo de Sergio — Fase 1 Punto 6 (Data Center Tier 4) + Infraestructura como Código / SDN (Fase 4)

**Entrega 1 — 23 de agosto de 2026 (tu punto de Fase 1) — Entrega 3 — 17 de octubre de 2026 (tu Fase 4, la más pesada del proyecto pero con más tiempo por delante).**

## Qué es tu módulo

Tu Fase 1 y tu Fase 4 están más conectadas de lo que parece: tu Nube Privada (VR1, SV1, las VMs de servicio) **vive físicamente** en el Data Center que diseñás en el Punto 6 — entender los requisitos de energía, enfriamiento y disponibilidad del sitio donde corre tu infraestructura es directamente relevante para tu propio módulo (ver por qué en [11-Equipo-y-Responsabilidades.md](../00-Documentacion-General/11-Equipo-y-Responsabilidades.md) §1.1). Documentos que son tuyos:

1. **Fase 1, Punto 6 — Data Center Tier 4** (Entrega 1, la más próxima): [06-Data-Center-Tier4.md](../Fase-1-Diseno-Red-Corporativa/06-Data-Center-Tier4.md).
2. **Fase 4, parte virtual** (Entrega 3): todo lo que corre dentro de Proxmox — el bridge Open vSwitch (SV1), la VM de VyOS (VR1), y el provisionamiento de las VMs de servicio (Web, DHCP, Proxy) usando **Terraform** — [Fase4-Nube-Privada-SDN.md](../Fase-4-Nube-Privada-SDN/04-Fase4-Nube-Privada-SDN.md) secciones 3.4 y 3.5, y [Terraform-Ansible-IaC.md](../00-Documentacion-General/05-Terraform-Ansible-IaC.md) secciones 2 a 4.

La división con Samuel en Fase 4 es clara: **vos creás las VMs (Terraform), él configura lo que corre dentro (Ansible)**. Con Luis coordinás el lado físico: tu VR1 se conecta al R1 físico que él compra y configura.

## Tu punto de Fase 1 (Punto 6 — Data Center Tier 4) — es tu entrega más próxima

[06-Data-Center-Tier4.md](../Fase-1-Diseno-Red-Corporativa/06-Data-Center-Tier4.md) es un diseño de ingeniería real con memoria de cálculo, no solo "poner UPS y ya" — cubre energía, enfriamiento, extinción de incendios, tierra física y control de acceso, todo con estándares citados (Uptime Institute, TIA-942, NFPA 2001/75/76, ASHRAE TC9.9, TIA-607-C).

### Decisiones clave de este punto y por qué se tomaron

**Qué distingue realmente a Tier 4 de Tier 3**: no es "más redundancia" en general — es **tolerancia a fallas** (el sitio debe sobrevivir una falla no planificada de cualquier componente, en cualquier momento, incluso durante mantenimiento), mientras que Tier 3 solo garantiza mantenimiento concurrente. Esta distinción está en doc. 06 §1 con una tabla comparativa — es la pregunta más común que te van a hacer sobre este punto.

**Memoria de cálculo del UPS (no una cifra inventada)**: se estima la carga IT crítica (2 routers Core + 2 switches de distribución redundantes + 10 switches de acceso + servidores + PBX ≈ 3,080 W), se convierte a VA con factor de potencia 0.9, se aplica margen de crecimiento del 30%, y se redondea al tamaño comercial más próximo (6 kVA, **APC Smart-UPS 6kVA Rack 6U 208V**, modelo real disponible en Kemik.gt) — luego se **duplica** (2N: dos UPS de 6kVA independientes trabajando en paralelo activo, no uno solo) porque Tier 4 exige que cada rama pueda cargar el 100% sola. Todo el cálculo paso a paso, más el checklist explícito de qué criterios Tier IV se cumplen y cómo (incluida la tabla de "lenguaje permitido vs. sobre-reclamo" para no decir más de lo que se puede sostener), en doc. 06 §1.1-1.3 y §3.

**Memoria de cálculo del enfriamiento (HVAC)**: misma lógica — Watts de carga IT × 3.412 = BTU/hr, se suma ganancia térmica del cuarto, se convierte a toneladas de refrigeración (÷12,000), y se redondea a 2 unidades CRAC de 2 toneladas en 2N — doc. 06 §4. Conecta directamente con tu experiencia dimensionando recursos para VMs: es el mismo tipo de razonamiento de capacidad, aplicado a energía/clima en vez de CPU/RAM.

**Por qué NO se construye un Data Center Tier 4 real para la demo**: el documento distingue explícitamente qué se diseña en papel (Tier 4 completo) vs. qué se demuestra en el laboratorio (1 UPS pequeño, backups automatizados) — doc. 06 §8. Igual que tu Nube Privada, que se diseña completa mientras el laboratorio usa una versión reducida.

### Este punto no tiene diagrama propio pendiente
El Punto 6 es memoria de cálculo y tablas, no topología de red — no hay un diagrama Mermaid que pasar a visual acá (a diferencia de los puntos 3 y 4).

## Decisiones clave de tu módulo de Fase 4 y por qué se tomaron

### 1. Por qué Proxmox VE + Open vSwitch + VyOS, y no OpenStack
El enunciado exige una SDN 100% open source con mínimo un switch y un router virtual. OpenStack es "el" estándar de nube privada real, pero necesita 32+ GB de RAM y semanas de curva de aprendizaje — total sobre-ingeniería para 3-4 VMs en una laptop de 8-16GB. Proxmox VE (hipervisor) + Open vSwitch (switch programable, cumple el rol de SV1) + VyOS (router Linux open source con OSPF/BGP, cumple el rol de VR1) da exactamente lo mismo que pide el enunciado con una fracción de la complejidad. Detalle completo del análisis en [Arquitectura-General.md](../00-Documentacion-General/00-Arquitectura-General.md) sección 1.2.

### 2. Por qué NO GNS3 en tu parte (esto es una regla, no una preferencia)
El enunciado dice literalmente: *"GNS3 es utilizado únicamente para la implementación de nuestro Core en el Data Center, no puede ser utilizado en nuestra solución SDN"*. Como al final decidimos comprar el Core físico (MikroTik, lo compra Luis), **GNS3 no entra en el proyecto en absoluto** — ni siquiera como opción de respaldo para tu parte. Tu SV1/VR1 tienen que ser Proxmox+OVS+VyOS reales, sin excepción.

### 3. Por qué Terraform con el provider `bpg/proxmox`
Es el provider de Terraform para Proxmox más activo y completo (soporta cloud-init, VMs QEMU, LXC). Con él, crear una VM pasa de ser "clicks en la consola de Proxmox" a un archivo `.tf` versionado en git — reproducible y con `terraform plan` como red de seguridad antes de aplicar cualquier cambio.

### 4. `network-inventory.yaml` — la pieza más importante de tu módulo
Todo el proyecto (Terraform, Ansible, diagramas) se genera a partir de un único archivo YAML que describe VLANs, subredes y VMs (ejemplo completo en el doc 05, sección 3). Es tuyo escribirlo completo y mantenerlo actualizado — cualquier VLAN nueva que agregue Jeferson en su tabla ([Direccionamiento-IP-VLANs.md](../00-Documentacion-General/07-Direccionamiento-IP-VLANs.md)) tiene que reflejarse ahí.

### 5. Tipo de CPU genérico (`kvm64`, no `host`)
Como el ambiente se mueve entre la laptop de 8GB de Samuel y una laptop prestada de 16GB el día de la demo, cada VM tiene que configurarse con tipo de CPU `kvm64` o `x86-64-v2` en vez de `host` — si no, una VM creada en un procesador puede no arrancar en el otro. Detalle en [Terraform-Ansible-IaC.md](../00-Documentacion-General/05-Terraform-Ansible-IaC.md) sección 8.2.

## Diagrama de tu módulo de Fase 4 — ya está definido, solo falta pasarlo a visual

En [Fase4-Nube-Privada-SDN.md](../Fase-4-Nube-Privada-SDN/04-Fase4-Nube-Privada-SDN.md) sección 2 hay un diagrama Mermaid completo de la topología física + virtual (R1 → switch físico → 2 hosts de prueba, y VR1 → SV1 → Web/DHCP/Proxy). Es el diagrama más importante del proyecto para la Fase 4 — pasalo a draw.io con la librería de íconos de red (tiene formas específicas para routers, switches y nubes). No hay decisiones de diseño pendientes, es la topología ya acordada — coordiná con Luis para que el lado físico del diagrama (R1, switch, hosts) quede fiel a lo que él realmente compra e instala. No hay apuro, es para la Entrega 3.

## Qué te toca hacer ahora

**Para el 23 de agosto (urgente, tu Fase 1):**
1. Revisar/pulir tu [06-Data-Center-Tier4.md](../Fase-1-Diseno-Red-Corporativa/06-Data-Center-Tier4.md) — es tu entrega más próxima.
2. Prepararte para defender la diferencia entre Tier 3 y Tier 4 (doc. 06 §1) — es la pregunta más frecuente sobre este punto.

**Para el 17 de octubre (tu Fase 4, con más tiempo):**
3. Avisale a Samuel cuando quieras acceso remoto (Tailscale) para empezar a probar tu código Terraform sin depender de estar físicamente con él.
4. Escribir el `network-inventory.yaml` completo junto con Jeferson (su tabla de VLANs es el insumo).
5. Escribir los módulos Terraform reales (`sdn/`, `compute/`, `network-vlans/`) — lo que hay hoy en el doc 05 es un ejemplo/esqueleto, no el código final.
6. Pasar el diagrama de topología a draw.io cuando el diseño esté estable.
