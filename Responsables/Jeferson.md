# Módulo de Jeferson — Fase 3 (LAN/WAN/VPN/Intranet/Monitoreo) + Respaldo del Ambiente

**Entrega 2 — 19 de septiembre de 2026 (junto con la parte de Samuel).**

## Qué es tu módulo

Sos el responsable de la **Fase 3 completa** — cómo se conecta y se protege la red hacia afuera (WAN, VPN, DMZ) y cómo se monitorea y colabora hacia adentro (intranet, monitoreo). También armaste la **tabla maestra de VLANs y direccionamiento IP**, que es el documento del que depende literalmente todo el resto del proyecto. Documentos que son tuyos:

1. [Fase3-LAN-WAN-VPN-Seguridad.md](../Fase-3-LAN-WAN-VPN-Seguridad/03-Fase3-LAN-WAN-VPN-Seguridad.md) — el documento completo
2. [Direccionamiento-IP-VLANs.md](../00-Documentacion-General/07-Direccionamiento-IP-VLANs.md) — la tabla de VLANs/subredes

## Decisiones clave de tu módulo y por qué se tomaron

### 1. Por qué WireGuard y no OpenVPN para la VPN de acceso remoto
El enunciado pide VPN de acceso remoto para fomentar trabajo a distancia. Se eligió **WireGuard** sobre OpenVPN (la opción "clásica" que muchos esperan) porque: configuración más simple (menos superficie de error), mejor rendimiento (corre en espacio de kernel, no en espacio de usuario), y una base de código mucho más pequeña y auditable. OpenVPN se documenta como alternativa por si el catedrático lo pide explícitamente — pero WireGuard es la recomendación técnica real, no solo una preferencia.

### 2. Por qué Nextcloud como intranet
El enunciado pide "aplicaciones basadas en la Web, incluidas conferencias, e-Learning y herramientas de colaboración" para fomentar trabajo a distancia. Nextcloud cubre archivos compartidos + calendario + chat (Nextcloud Talk) en una sola suite open source, con apps de escritorio/móvil — evita depender de Zoom/Teams/Google Workspace, que no son open source ni on-premise.

### 3. Por qué Zabbix y no LibreNMS para monitoreo
LibreNMS es mejor específicamente para auto-descubrimiento de topología de red (SNMP/LLDP), pero Zabbix cubre **red + servidores** con una sola herramienta (agentes en cada VM más SNMP en los equipos de red) — para un equipo de 5 con varios servicios corriendo, tener un solo panel de monitoreo en vez de dos es más práctico.

### 4. Por qué las 3 zonas (Internet / DMZ / Interna) y esas reglas específicas
Patrón clásico de firewall: Internet solo puede tocar la DMZ (Web Server, y el relay de correo si se publica ahí) en los puertos exactos que se publican; la DMZ **nunca** puede iniciar conexión hacia la LAN interna (si comprometen el Web Server, no debe poder pivotar hacia Administración o Servidores); la LAN interna sí puede llegar a la DMZ en los puertos de servicio necesarios. Todo esto se implementa con ACLs en el mismo Router Core (R1), sin necesitar firewall dedicado aparte.

### 5. Por qué `/24` uniforme para las VLANs (y no VLSM)
Con 184 dispositivos totales, usar una subred `/24` (254 hosts) por VLAN es simple de justificar y consistente con lo que se ve en el curso (direccionamiento clase C). Se dejó documentado un anexo opcional de VLSM (en el doc 07, al final) para sumar puntos extra si querés mostrar ese dominio — pero no es necesario para que el diseño esté completo y correcto.

## Tu tabla de VLANs — la pieza de la que depende todo el equipo

La tabla en [Direccionamiento-IP-VLANs.md](../00-Documentacion-General/07-Direccionamiento-IP-VLANs.md) es la **fuente de verdad** de todo el proyecto: Melany la referencia en su diseño lógico (Fase 1), Sergio la va a convertir en el `network-inventory.yaml` de Terraform, y Luis configura el firewall de R1 basado en esas mismas VLANs. Si cambiás algo ahí (un rango, una VLAN nueva), avisale al equipo — es un cambio que se propaga a todos lados.

## Diagrama de tu módulo — ya está definido, solo falta pasarlo a visual

En [Fase3-LAN-WAN-VPN-Seguridad.md](../Fase-3-LAN-WAN-VPN-Seguridad/03-Fase3-LAN-WAN-VPN-Seguridad.md), al final de la sección 4 (Zonas Desmilitarizadas), hay un diagrama Mermaid completo de las 3 zonas con las reglas de tráfico permitido/denegado entre Internet, DMZ y LAN interna. No hay ninguna decisión pendiente — son exactamente las reglas ya escritas en prosa arriba del diagrama, solo traducidas a dibujo. Pasalo a draw.io como los demás.

## Responsabilidad extra: respaldo del ambiente

Sos el **punto de respaldo designado** del proyecto (ver [Equipo-y-Responsabilidades.md](../00-Documentacion-General/11-Equipo-y-Responsabilidades.md) sección 5). Como Samuel es el único con el hardware físico, necesitás tener una copia — de la imagen del SSD externo, o al menos del repositorio de código completo (que basta para reconstruir todo con `terraform apply` + `ansible-playbook` si hiciera falta). Coordiná con él cuándo hacer esa copia, idealmente antes de la Entrega 2.

## Qué te toca hacer ahora

1. Revisar la tabla de VLANs (doc 07) con calma — es más fácil ajustarla ahora que después de que Sergio ya tenga Terraform construido sobre esa base.
2. Coordinar con Samuel quién configura qué en Ansible (Nextcloud, Zabbix, WireGuard) — vos diseñaste el "qué", falta el "cómo".
3. Pasar el diagrama de zonas DMZ a draw.io.
4. Organizar con Samuel la copia de respaldo del ambiente.
5. Tu entrega (19 de septiembre) va junto con la de Samuel — mantengan sincronizado el avance.
