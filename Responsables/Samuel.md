# Módulo de Samuel — Correo (Fase 2) + Configuración de Máquinas + Custodio del Hardware

**Entrega 2 — 19 de septiembre de 2026 (junto con la parte de Jeferson).**

## Qué es tu módulo

Tenés dos sombreros en este proyecto:

1. **Dueño de la Fase 2 completa**: el servidor de correo on-premise — [Fase2-Servidor-Correo.md](../Fase-2-Servidor-Correo/02-Fase2-Servidor-Correo.md).
2. **Capa de configuración (Ansible) de todo el proyecto**: mientras Sergio provisiona las VMs con Terraform, vos configurás lo que corre *dentro* de cada una (Web, DHCP, Proxy, VyOS, y tu propio correo) — ver [Terraform-Ansible-IaC.md](../00-Documentacion-General/05-Terraform-Ansible-IaC.md) sección 5.

Además sos el **custodio del hardware físico** (tu laptop + el SSD externo portátil) y el aprobador de los Pull Requests que tocan `terraform/` o `ansible/` en el repo del equipo, porque sos el único que puede probarlos contra el ambiente real antes de aceptarlos.

## Decisiones clave de tu módulo y por qué se tomaron

### 1. Por qué Postfix + Dovecot + rspamd (y no un servicio en la nube)
El enunciado prohíbe explícitamente resolver el correo con un proveedor externo ("no se puede realizar ninguna parte de esta fase en la nube") — se interpretó como: nada de Gmail/Office365/AWS SES. Postfix (motor SMTP) + Dovecot (buzones IMAP) + rspamd (anti-spam moderno, reemplaza a SpamAssassin con mejor rendimiento) es el stack open source estándar para esto, y corre completo en una sola VM.

### 2. Por qué 2 dominios simulados
El enunciado dice "cada grupo tendrá un dominio asignado y todos deben poder enviar correos entre sí" — para demostrar entrega **inter-dominio** real (no solo intra-dominio), se simulan 2 dominios (`virtualsolutions.lab` y `partner-demo.lab`) en el mismo servidor, con `virtual_mailbox_domains` de Postfix. Es exactamente lo que hace cualquier proveedor de correo que aloja varios dominios de clientes.

### 3. Atajo legítimo: docker-mailserver
Si armar Postfix+Dovecot+rspamd desde cero a mano te consume mucho tiempo, [docker-mailserver](https://github.com/docker-mailserver/docker-mailserver) es el mismo stack pero empaquetado y configurado por variables de entorno — reduce el trabajo de días a horas sin dejar de ser 100% open source ni cambiar el diseño. Está documentado como alternativa en la sección 5 del doc de Fase 2.

### 4. Por qué Ansible es tu herramienta (y no Terraform)
Terraform es bueno para decir "qué debe existir" (una VM, un disco, una IP) — pero es malo para configurar lo que corre *dentro*. Ansible sí — instala paquetes, genera archivos de configuración (`dhcpd.conf`, `squid.conf`, la config de VyOS) a partir de plantillas. La división de trabajo con Sergio es: **él provisiona, vos configurás** — es el mismo patrón que usan equipos de infraestructura profesionales.

### 5. Por qué vos sos el aprobador de PRs de infraestructura
No es jerarquía, es logística: sos el único con acceso físico al ambiente real, así que sos quien puede confirmar que un cambio de Sergio o de Jeferson realmente funciona antes de que quede en la rama principal del repo.

## Diagrama de tu módulo — ya está definido, solo falta pasarlo a visual

En [Fase2-Servidor-Correo.md](../Fase-2-Servidor-Correo/02-Fase2-Servidor-Correo.md) sección 2.1 ya hay un diagrama Mermaid completo del flujo de correo (entrada SMTP → rspamd → buzones intra/inter-dominio → Dovecot → Roundcube, más el rechazo de spam). No hay ninguna decisión pendiente ahí — solo pasarlo a draw.io igual que los diagramas de Melany, si querés dejarlo con la misma consistencia visual del resto del proyecto. No es obligatorio para tu entrega (la Fase 2 no exige diagrama específico en el enunciado), pero ayuda a explicar el flujo en la defensa.

## Qué te toca hacer ahora

1. Definir si vas por Postfix+Dovecot+rspamd "a mano" o por docker-mailserver (afecta cuánto tiempo necesitás).
2. Preparar el SSD externo (instalar Proxmox VE ahí, no en el disco interno) — ver [Terraform-Ansible-IaC.md](../00-Documentacion-General/05-Terraform-Ansible-IaC.md) sección 8 para el detalle completo de esa estrategia.
3. Instalar Tailscale en tu Proxmox para que el resto del equipo pueda colaborar remoto (sección 8.3 del mismo documento).
4. Coordinar con Jeferson: su Fase 3 y tu Fase 2 se entregan juntas el 19 de septiembre — sincronicen avance.
5. Cuando Sergio tenga los primeros módulos de Terraform, empezás a escribir los roles de Ansible correspondientes.
