# 09 — Políticas de Seguridad (Lógicas y Físicas)

## 1. Políticas lógicas

### 1.1 Control de acceso por usuario
- Autenticación centralizada recomendada (LDAP/FreeIPA como extensión futura); para el alcance de este proyecto, autenticación local por servicio con contraseñas fuertes (mínimo 12 caracteres, rotación cada 90 días para cuentas administrativas).
- Principio de menor privilegio: cada usuario/servicio solo tiene acceso a lo que su rol requiere (ej. Ventas no tiene acceso SSH a ningún servidor; Soporte I/T sí).
- MFA recomendado como roadmap para VPN y accesos administrativos (WireGuard + llave, o TOTP en Nextcloud) — documentado como mejora, no bloqueante para esta entrega.

### 1.2 Control de acceso por computador/dispositivo
- Segmentación por VLAN (ver documento 07) como primer control — un dispositivo en VLAN Ventas no puede alcanzar la VLAN Servidores salvo por los puertos de servicio explícitamente permitidos.
- Filtrado por MAC/802.1X como mejora futura para puertos de acceso en switches de piso (evita que alguien conecte un equipo no autorizado a un jack libre).

### 1.3 Control de acceso a Internet
- Todo el tráfico saliente de las VLANs de usuario pasa por el Proxy (Squid) — ver [04-Fase4-Nube-Privada-SDN.md](../Fase-4-Nube-Privada-SDN/04-Fase4-Nube-Privada-SDN.md) sección 3.3.
- Filtrado de categorías/dominios en Squid (listas de bloqueo para contenido no laboral) configurable por VLAN.
- Egreso directo tcp/80 y tcp/443 bloqueado salvo excepción documentada (destino = Web Server propio).
- Logging centralizado de accesos a Internet en Squid, retenido mínimo 90 días para auditoría.

### 1.4 Firewall y segmentación
- Firewall stateful en R1 (Core) con chains por VLAN — política por defecto **denegar**, permitir explícitamente.
- Reglas DMZ↔LAN↔Internet documentadas en [03-Fase3-LAN-WAN-VPN-Seguridad.md](../Fase-3-LAN-WAN-VPN-Seguridad/03-Fase3-LAN-WAN-VPN-Seguridad.md) sección 4.
- No se permiten rutas estáticas (requisito del enunciado) — todo el enrutamiento es dinámico vía OSPF, reduciendo el riesgo de rutas mal configuradas manualmente que abran caminos no auditados.

### 1.5 Correo y anti-spam
- Ver [02-Fase2-Servidor-Correo.md](../Fase-2-Servidor-Correo/02-Fase2-Servidor-Correo.md) — SPF/DKIM/DMARC obligatorios, rspamd con greylisting y RBLs.

### 1.6 Monitoreo y auditoría
- Zabbix centraliza disponibilidad y alertas (ver documento 03).
- Logs de firewall, proxy y VPN centralizados (syslog hacia la VM de monitoreo) para correlación de incidentes.

## 2. Políticas físicas

### 2.1 Acceso al Data Center
- Acceso restringido por niveles: solo Soporte I/T y Administración de red tienen acceso físico al cuarto de equipos.
- Control de acceso recomendado: lector de tarjeta/PIN como mínimo viable; biométrico como estándar deseable para Tier 4 (ver documento 10).
- Bitácora de acceso físico (quién entró, cuándo, motivo) — manual o con sistema de control de acceso si el presupuesto lo permite.
- CCTV en accesos al Data Center y pasillos críticos (mencionado en el diagrama de referencia del enunciado).

### 2.2 Protección ambiental (detalle completo en documento 10)
- Piso elevado, aire acondicionado de precisión, sistema de extinción por agente limpio (no agua ni CO2 puro por daño a equipo), tierra física dedicada, UPS + planta eléctrica.

### 2.3 Cableado
- Cableado estructurado certificado (ver documento 01), etiquetado obligatorio, sin empalmes fuera de patch panels.
- Bandejas/canaletas separadas para datos y energía (evitar interferencia electromagnética).

### 2.4 Gestión de activos
- Inventario de todo equipo de red/servidores con número de serie, ubicación, fecha de garantía — mantenido en NetBox (mismo IPAM sugerido en [00-Arquitectura-General.md](../00-Documentacion-General/00-Arquitectura-General.md)) para no duplicar herramientas.

## 3. Resumen de responsabilidades

| Política | Aplica a | Mecanismo técnico | Documento relacionado |
|---|---|---|---|
| Segmentación de red | Todos los usuarios | VLANs + firewall en R1 | [07](../00-Documentacion-General/07-Direccionamiento-IP-VLANs.md) |
| Acceso a Internet controlado | Todos los usuarios | Proxy Squid + firewall | [04](../Fase-4-Nube-Privada-SDN/04-Fase4-Nube-Privada-SDN.md) |
| Acceso remoto seguro | Empleados remotos | VPN WireGuard | [03](../Fase-3-LAN-WAN-VPN-Seguridad/03-Fase3-LAN-WAN-VPN-Seguridad.md) |
| Anti-spam/anti-phishing | Todo correo entrante/saliente | rspamd + SPF/DKIM/DMARC | [02](../Fase-2-Servidor-Correo/02-Fase2-Servidor-Correo.md) |
| Acceso físico al Data Center | Personal autorizado | Control de acceso + CCTV | Sección 2 de este documento, [10](10-Data-Center-Tier4.md) |
| Disponibilidad de servicios | Toda la operación | OSPF dinámico + HA de energía/enfriamiento | [03](../Fase-3-LAN-WAN-VPN-Seguridad/03-Fase3-LAN-WAN-VPN-Seguridad.md), [10](10-Data-Center-Tier4.md) |
