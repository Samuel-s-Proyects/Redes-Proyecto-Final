# Módulo de Jeferson — Fase 1 Punto 3 (Diseño Lógico) + Fase 3 (LAN/WAN/VPN/Intranet/Monitoreo) + Respaldo del Ambiente

**Entrega 1 — 23 de agosto de 2026 (tu punto de Fase 1) — Entrega 2 — 19 de septiembre de 2026 (tu Fase 3, junto con la parte de Samuel).**

## Qué es tu módulo

Tenés dos partes, y no es casualidad que estén relacionadas — la Fase 1 Punto 3 es la continuación lógica de lo que ya hacías en Fase 3, solo que aplicado a toda la empresa (184 usuarios) en vez de a la Fase 3 específicamente (ver por qué en [11-Equipo-y-Responsabilidades.md](../00-Documentacion-General/11-Equipo-y-Responsabilidades.md) §1.1):

1. **Fase 1, Punto 3 — Diseño Lógico** (Entrega 1, la más próxima): [03-Diseno-Logico.md](../Fase-1-Diseno-Red-Corporativa/03-Diseno-Logico.md) — modelo jerárquico, diagrama de los 4 switches de la red completa, QoS/DSCP, enrutamiento.
2. **Fase 3 completa** (Entrega 2): cómo se conecta y se protege la red hacia afuera (WAN, VPN, DMZ) y cómo se monitorea y colabora hacia adentro (intranet, monitoreo) — [Fase3-LAN-WAN-VPN-Seguridad.md](../Fase-3-LAN-WAN-VPN-Seguridad/03-Fase3-LAN-WAN-VPN-Seguridad.md).
3. **La tabla maestra de VLANs y direccionamiento IP**, que es el documento del que depende literalmente todo el resto del proyecto: [Direccionamiento-IP-VLANs.md](../00-Documentacion-General/07-Direccionamiento-IP-VLANs.md).

## Tu punto de Fase 1 (Punto 3 — Diseño Lógico) — es tu entrega más próxima

[03-Diseno-Logico.md](../Fase-1-Diseno-Red-Corporativa/03-Diseno-Logico.md) responde "¿qué switches lleva la red completa y qué VLAN corre por cada uno?" — a diferencia de la demo de laboratorio de la Fase 4 (que usa 1 solo switch con 2 puertos), este es el diseño de **producción completo**: 4 switches (1 de distribución en el Data Center + 3 IDF de piso), con sus enlaces trunk.

### Decisiones clave de este punto y por qué se tomaron

**Modelo jerárquico de 3 capas (Core-Distribución-Acceso)**: es el marco estándar de la industria para una red de este tamaño. Con 184 usuarios en 4 niveles y necesidad de política de firewall diferenciada por VLAN, separar la función de distribución (que agrega los 3 switches de piso) del Core (que solo enruta) evita sobrecargar a R1 con la administración de cada puerto de acceso individual — detalle completo en doc. 03 §1.

**Por qué no hace falta Spanning Tree (por ahora)**: la topología distribución→IDF es un árbol (cada IDF cuelga de un único enlace) — no hay una segunda ruta que pueda crear un bucle, así que RSTP no es obligatorio en la configuración base. Sí dejamos documentado que **se vuelve obligatorio** si en el futuro se agregan enlaces redundantes por IDF (doc. 03 §5) — un detalle que muestra que pensaste en la evolución del diseño, no solo en el estado actual.

**Esquema de QoS/DSCP**: se definió marcado de prioridad (EF para VoIP, AF31 para tráfico de gestión, AF21 para transaccional, AF11/CS1 para tráfico masivo de Desarrollo) para que, ante congestión, una llamada VoIP no se degrade por una transferencia grande compitiendo por el mismo ancho de banda — doc. 03 §4, conecta directamente con el análisis de clasificación de tráfico que hizo Melany en el Punto 2.

### Diagrama de este punto — ya está definido, solo falta pasarlo a visual

En [03-Diseno-Logico.md](../Fase-1-Diseno-Red-Corporativa/03-Diseno-Logico.md) §2 está el diagrama Mermaid completo de los 4 switches con sus VLANs — es probablemente el diagrama más citado de toda la Fase 1 porque responde directamente la pregunta "¿qué switches lleva la red?". Pasalo a draw.io (Extras → Edit Diagram → pegar el Mermaid) esta semana, es tu entrega más próxima.

## Actualización: el código de tu Fase 3 ya está escrito

Los 3 servicios ya tienen su rol de Ansible completo en [`infra/ansible/roles/`](../../infra/ansible/roles/): `vpn/` (WireGuard), `intranet/` (Nextcloud) y `monitoring/` + `zabbix_agent/` (Zabbix). También se llenó el `network-inventory.yaml` completo con las 12 VLANs de tu tabla. Ver [infra/README.md](../../infra/README.md) para desplegarlos en cuanto existan las VMs, y [infra/scripts/test-fase3-services.sh](../../infra/scripts/test-fase3-services.sh) para validarlos.

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

La tabla en [Direccionamiento-IP-VLANs.md](../00-Documentacion-General/07-Direccionamiento-IP-VLANs.md) es la **fuente de verdad** de todo el proyecto: tu propio diseño lógico (Punto 3 de Fase 1) la usa como base, Luis la referencia en su diseño físico (Punto 4), Sergio la va a convertir en el `network-inventory.yaml` de Terraform, y Luis (Fase 4) configura el firewall de R1 basado en esas mismas VLANs. Si cambiás algo ahí (un rango, una VLAN nueva), avisale al equipo — es un cambio que se propaga a todos lados.

## Diagrama de tu módulo de Fase 3 — ya está definido, solo falta pasarlo a visual

En [Fase3-LAN-WAN-VPN-Seguridad.md](../Fase-3-LAN-WAN-VPN-Seguridad/03-Fase3-LAN-WAN-VPN-Seguridad.md), al final de la sección 4 (Zonas Desmilitarizadas), hay un diagrama Mermaid completo de las 3 zonas con las reglas de tráfico permitido/denegado entre Internet, DMZ y LAN interna. No hay ninguna decisión pendiente — son exactamente las reglas ya escritas en prosa arriba del diagrama, solo traducidas a dibujo. Este lo pasás a draw.io más adelante (Entrega 2, no hay apuro todavía).

## Responsabilidad extra: respaldo del ambiente

Sos el **punto de respaldo designado** del proyecto (ver [Equipo-y-Responsabilidades.md](../00-Documentacion-General/11-Equipo-y-Responsabilidades.md) sección 5). Como Samuel es el único con el hardware físico, necesitás tener una copia — de la imagen del SSD externo, o al menos del repositorio de código completo (que basta para reconstruir todo con `terraform apply` + `ansible-playbook` si hiciera falta). Coordiná con él cuándo hacer esa copia, idealmente antes de la Entrega 2.

## Qué te toca hacer ahora

**Para el 23 de agosto (urgente, tu Fase 1):**
1. Revisar/pulir tu [03-Diseno-Logico.md](../Fase-1-Diseno-Red-Corporativa/03-Diseno-Logico.md) — es tu entrega más próxima.
2. Pasar el diagrama de los 4 switches a draw.io.
3. Revisar la tabla de VLANs (doc 07) con calma — es más fácil ajustarla ahora que después de que Sergio ya tenga Terraform construido sobre esa base.

**Para el 19 de septiembre (tu Fase 3, con más tiempo):**
4. Coordinar con Samuel quién configura qué en Ansible (Nextcloud, Zabbix, WireGuard) — vos diseñaste el "qué", falta el "cómo".
5. Pasar el diagrama de zonas DMZ a draw.io.
6. Organizar con Samuel la copia de respaldo del ambiente.
7. Tu entrega (19 de septiembre) va junto con la de Samuel — mantengan sincronizado el avance.
