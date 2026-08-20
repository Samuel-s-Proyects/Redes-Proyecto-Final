# Módulo de Samuel — Fase 1 Punto 5 (Políticas de Seguridad) + Correo (Fase 2) + Configuración de Máquinas + Custodio del Hardware

**Entrega 1 — 23 de agosto de 2026 (tu punto de Fase 1) — Entrega 2 — 19 de septiembre de 2026 (tu Fase 2, junto con la parte de Jeferson).**

## Qué es tu módulo

Tenés tres sombreros en este proyecto:

1. **Fase 1, Punto 5 — Políticas de Seguridad** (Entrega 1, la más próxima): [05-Politicas-Seguridad.md](../Fase-1-Diseno-Red-Corporativa/05-Politicas-Seguridad.md) — te tocó por conexión directa con tu rol de gobernanza (aprobador de PRs) y con la seguridad del correo que vas a implementar en Fase 2 (ver por qué en [11-Equipo-y-Responsabilidades.md](../00-Documentacion-General/11-Equipo-y-Responsabilidades.md) §1.1).
2. **Dueño de la Fase 2 completa**: el servidor de correo on-premise — [Fase2-Servidor-Correo.md](../Fase-2-Servidor-Correo/02-Fase2-Servidor-Correo.md).
3. **Capa de configuración (Ansible) de todo el proyecto**: mientras Sergio provisiona las VMs con Terraform, vos configurás lo que corre *dentro* de cada una (Web, DHCP, Proxy, VyOS, y tu propio correo) — ver [Terraform-Ansible-IaC.md](../00-Documentacion-General/05-Terraform-Ansible-IaC.md) sección 5.

Además sos el **custodio del hardware físico** (tu laptop + el SSD externo portátil) y el aprobador de los Pull Requests que tocan `terraform/` o `ansible/` en el repo del equipo, porque sos el único que puede probarlos contra el ambiente real antes de aceptarlos.

## Tu punto de Fase 1 (Punto 5 — Políticas de Seguridad) — es tu entrega más próxima

[05-Politicas-Seguridad.md](../Fase-1-Diseno-Red-Corporativa/05-Politicas-Seguridad.md) es un marco completo: **15 políticas lógicas + 7 físicas**, no una lista corta de buenas intenciones — cubre desde gestión de identidad hasta clasificación de información, respuesta a incidentes, y control de acceso físico al Data Center.

### Decisiones clave de este punto y por qué se tomaron

**Marco de referencia real (ISO 27001 / NIST CSF)**: las políticas no se inventaron sueltas — siguen la misma lógica de organización que usan esos dos marcos estándar de la industria (sin declarar cumplimiento certificado, que requeriría auditoría formal fuera de alcance) — doc. 05 §0.3. Da mucho más peso a la defensa que "se nos ocurrieron estas reglas".

**Por qué NO hay expiración forzada de contraseñas cada 30 días**: puede sonar contraintuitivo, pero es la recomendación real de NIST SP 800-63B — la rotación forzada frecuente lleva a patrones predecibles (`Enero2026!`, `Febrero2026!`). Se usa longitud mínima (12 caracteres) + MFA en accesos críticos en su lugar, con rotación obligatoria de 90 días solo para cuentas administrativas — doc. 05 §1.2. Es un punto que sorprende al catedrático si lo esperás con la respuesta lista.

**Matriz de acceso inter-VLAN explícita**: en vez de decir "hay firewall entre VLANs" en abstracto, doc. 05 §1.3.1 tiene la tabla completa de qué VLAN puede llegar a qué destino y por qué puerto — es lo que le da sustento operativo a la política de segmentación del Punto 3 de Jeferson.

**Gestión de cambios ligada al propio flujo de trabajo del equipo**: la política 1.12 no es teórica — literalmente describe el flujo de Pull Requests que ya usás como aprobador (`terraform plan`/revisión antes de aplicar). La política de seguridad y la forma real de trabajar del equipo son la misma cosa, no dos documentos separados que no se hablan entre sí.

### Este punto no tiene diagrama propio
A diferencia de los puntos 3 y 4, el Punto 5 no tiene un diagrama Mermaid pendiente de pasar a visual — es un documento de políticas (tablas y prosa), no de topología. Si querés, la matriz de acceso inter-VLAN (doc. 05 §1.3.1) se presta para una tabla visual simple en draw.io, pero no es obligatorio.

## Actualización: el código de tu Fase 2 ya está escrito

Ya no es solo diseño — el rol de Ansible completo vive en [`infra/ansible/roles/mailserver/`](../../infra/ansible/roles/mailserver/) (docker-mailserver: Postfix+Dovecot+rspamd+Roundcube, 2 dominios, generación de DKIM, script de registros DNS de referencia). También está el script de pruebas [`infra/scripts/test-mailflow.sh`](../../infra/scripts/test-mailflow.sh). Ver [infra/README.md](../../infra/README.md) para los pasos exactos de despliegue una vez tengas la VM `vm-mail` arriba (necesitás: crear la VM, poner su IP en `ansible/inventory/hosts.ini`, crear `ansible/group_vars/vault.yml` con las contraseñas reales, y correr `ansible-playbook site.yml`).

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

## Diagrama de tu módulo de Fase 2 — ya está definido, solo falta pasarlo a visual

En [Fase2-Servidor-Correo.md](../Fase-2-Servidor-Correo/02-Fase2-Servidor-Correo.md) sección 2.1 ya hay un diagrama Mermaid completo del flujo de correo (entrada SMTP → rspamd → buzones intra/inter-dominio → Dovecot → Roundcube, más el rechazo de spam). No hay ninguna decisión pendiente ahí — solo pasarlo a draw.io si querés dejarlo con la misma consistencia visual del resto del proyecto. No es obligatorio para tu entrega (la Fase 2 no exige diagrama específico en el enunciado), pero ayuda a explicar el flujo en la defensa. No hay apuro, es para la Entrega 2.

## Qué te toca hacer ahora

**Para el 23 de agosto (urgente, tu Fase 1):**
1. Revisar/pulir tu [05-Politicas-Seguridad.md](../Fase-1-Diseno-Red-Corporativa/05-Politicas-Seguridad.md) — es tu entrega más próxima.
2. Prepararte para defender por qué no hay expiración forzada de contraseñas (doc. 05 §1.2) — es el punto que más suele generar preguntas por sonar contraintuitivo.

**Para el 19 de septiembre (tu Fase 2, con más tiempo):**
3. Definir si vas por Postfix+Dovecot+rspamd "a mano" o por docker-mailserver (afecta cuánto tiempo necesitás).
4. Preparar el SSD externo (instalar Proxmox VE ahí, no en el disco interno) — ver [Terraform-Ansible-IaC.md](../00-Documentacion-General/05-Terraform-Ansible-IaC.md) sección 8 para el detalle completo de esa estrategia.
5. Instalar Tailscale en tu Proxmox para que el resto del equipo pueda colaborar remoto (sección 8.3 del mismo documento).
6. Coordinar con Jeferson: su Fase 3 y tu Fase 2 se entregan juntas el 19 de septiembre — sincronicen avance.
7. Cuando Sergio tenga los primeros módulos de Terraform, empezás a escribir los roles de Ansible correspondientes.
