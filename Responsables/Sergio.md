# Módulo de Sergio — Infraestructura como Código / SDN (Fase 4)

**Entrega 3 — 17 de octubre de 2026 (la más pesada del proyecto, pero con más tiempo por delante).**

## Qué es tu módulo

Sos el responsable de la **parte virtual de la Fase 4**: todo lo que corre dentro de Proxmox — el bridge Open vSwitch (SV1), la VM de VyOS (VR1), y el provisionamiento (creación) de las VMs de servicio (Web, DHCP, Proxy) usando **Terraform**. Documentos que son tuyos:

1. [Fase4-Nube-Privada-SDN.md](../Fase-4-Nube-Privada-SDN/04-Fase4-Nube-Privada-SDN.md) — secciones 3.4 (SV1) y 3.5 (VR1)
2. [Terraform-Ansible-IaC.md](../00-Documentacion-General/05-Terraform-Ansible-IaC.md) — toda la parte de Terraform (secciones 2 a 4)

La división con Samuel es clara: **vos creás las VMs (Terraform), él configura lo que corre dentro (Ansible)**. Con Luis coordinás el lado físico: tu VR1 se conecta al R1 físico que él compra y configura.

## Decisiones clave de tu módulo y por qué se tomaron

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

## Diagrama de tu módulo — ya está definido, solo falta pasarlo a visual

En [Fase4-Nube-Privada-SDN.md](../Fase-4-Nube-Privada-SDN/04-Fase4-Nube-Privada-SDN.md) sección 2 hay un diagrama Mermaid completo de la topología física + virtual (R1 → switch físico → host de prueba, y VR1 → SV1 → Web/DHCP/Proxy). Es el diagrama más importante del proyecto para la Fase 4 — pasalo a draw.io con la librería de íconos de red (tiene formas específicas para routers, switches y nubes). No hay decisiones de diseño pendientes, es la topología ya acordada — coordiná con Luis para que el lado físico del diagrama (R1, switch, host) quede fiel a lo que él realmente compra e instala.

## Qué te toca hacer ahora

1. Avisale a Samuel cuando quieras acceso remoto (Tailscale) para empezar a probar tu código Terraform sin depender de estar físicamente con él.
2. Escribir el `network-inventory.yaml` completo junto con Jeferson (su tabla de VLANs es el insumo).
3. Escribir los módulos Terraform reales (`sdn/`, `compute/`, `network-vlans/`) — lo que hay hoy en el doc 05 es un ejemplo/esqueleto, no el código final.
4. Pasar el diagrama de topología a draw.io cuando el diseño esté estable (no hay apuro, tu entrega es en octubre).
