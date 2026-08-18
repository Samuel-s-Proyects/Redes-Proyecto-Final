# 06 — Automatización con IA (Claude Code)

Objetivo del enunciado que esto resuelve: "automatizarlo con IA lo máximo posible y dejar los procesos manuales lo menos [posible]". Esta es la guía concreta de **qué pedirle a la IA** en cada etapa, no solo la idea general.

## 1. Principio: la IA genera, vos revisás y aplicás

La IA (Claude Code corriendo en esta misma carpeta del proyecto) nunca debe tener acceso directo a aplicar cambios en producción sin revisión — pero sí debe encargarse de todo el trabajo mecánico:

| Tarea manual tradicional | Reemplazo con IA |
|---|---|
| Escribir `dhcpd.conf` con un scope por VLAN a mano | "Genera `dhcpd.conf` desde `network-inventory.yaml`" |
| Escribir reglas de firewall de R1 una por una | "Genera el script `.rsc` de firewall de R1 a partir de la tabla de VLANs y las reglas DMZ/LAN/Internet del documento 03" |
| Escribir el playbook de Postfix multi-dominio | "Genera el rol Ansible `mailserver` para Postfix+Dovecot+rspamd con estos 2 dominios" |
| Mantener diagramas Mermaid sincronizados con la topología real | "Actualiza el diagrama Mermaid del documento 00 con el nuevo VLAN que acabo de agregar" |
| Redactar el manual de seguridad y el costeo para la entrega | "Genera el manual de seguridad en base a las reglas de firewall reales implementadas" |
| Debuggear por qué OSPF no forma adyacencia | Pegar el output de `/routing ospf neighbor print` y `show ip ospf neighbor` y pedir diagnóstico |
| Escribir pruebas de conectividad reproducibles | "Genera un script bash que corra todo el checklist de la sección 6 de la Fase 4 y devuelva PASS/FAIL" |

## 2. Flujo recomendado por fase

### Fase 1 (diseño)
- Pedir a la IA que genere/actualice diagramas de planta y de red a partir de las decisiones tomadas en los documentos.
- Pedir que calcule automáticamente cantidades de materiales (cable, patch panels, conectores) a partir del número de puestos y la distribución por piso — evita cálculos manuales propensos a error.

### Fase 2 (correo)
- Pedir el rol Ansible completo de `mailserver` a partir del stack elegido en [02](../Fase-2-Servidor-Correo/02-Fase2-Servidor-Correo.md).
- Pedir que genere los registros DNS necesarios (SPF, DKIM, DMARC, MX) como archivo de zona listo para copiar/pegar.
- Pedir un script de prueba automatizada (`swaks` o similar) que mande correos de prueba intra/inter-dominio y valide que llegan.

### Fase 3 (LAN/WAN/VPN)
- Pedir los playbooks de Nextcloud, Zabbix y WireGuard.
- Pedir plantillas de dashboards de Zabbix específicas para MikroTik (SNMP) y VyOS.

### Fase 4 (SDN — la más pesada)
- Pedir los módulos Terraform completos (`sdn`, `compute`, `network-vlans`) a partir del `network-inventory.yaml`.
- Pedir la configuración OSPF de VyOS y el script `.rsc` de OSPF/firewall de R1 en paralelo, para que ambos lados del enlace queden consistentes desde el primer intento.
- Pedir que genere el script de pruebas de conectividad end-to-end (ping, traceroute, verificación de adyacencia OSPF, prueba de DHCP por VLAN, prueba de bloqueo de proxy) como un solo comando ejecutable antes de cada demo.

## 3. Buenas prácticas al usar IA en este proyecto

1. **Nunca pegar credenciales reales** (tokens de API de Proxmox, contraseñas) en el chat — usar variables/placeholders y cargarlas por `terraform.tfvars` o Ansible Vault localmente.
2. **Pedir siempre `terraform plan` antes de `apply`** y revisar el diff generado — la IA puede cometer errores de sintaxis o de lógica igual que una persona; el `plan` es la red de seguridad.
3. **Versionar todo en git** (excepto secretos) — así cada sugerencia de la IA es un commit revisable, no un cambio invisible.
4. **Usar la IA para explicar, no solo generar** — pedirle que explique por qué OSPF eligió cierta ruta, o por qué Squid bloqueó una solicitud, ayuda a defender el proyecto oralmente ante el catedrático (que seguramente preguntará "¿por qué elegiste esto?").
5. **Mantener el inventario YAML como única fuente editada a mano** — todo lo demás (Terraform, Ansible vars, diagramas, tablas de este README) se regenera desde ahí con ayuda de la IA, para que nunca queden documentos desincronizados de la implementación real.

## 4. Qué NO automatizar con IA (decisiones que siguen siendo tuyas)

- La elección de protocolo de enrutamiento (ya decidido: OSPF, pero el criterio de por qué es tuyo para defenderlo).
- El presupuesto final y las marcas/modelos específicos a comprar (la IA puede investigar precios, pero la decisión de compra es tuya).
- Cualquier cambio que afecte el enlace a Internet real de tu casa mientras el laboratorio esté activo (para no perder conectividad tuya por error).
- La aplicación final de comandos sobre el MikroTik físico en producción — siempre revisar el script `.rsc` generado antes de importarlo.
