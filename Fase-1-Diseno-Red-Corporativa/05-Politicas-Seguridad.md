# Políticas de Seguridad — Virtual Solutions

## 0. Objetivo, alcance y marco de referencia

### 0.1 Objetivo
Establecer el conjunto de políticas, normas y controles que garanticen la **confidencialidad, integridad y disponibilidad** (tríada CIA) de la información y de la infraestructura de red de Virtual Solutions, cubriendo tanto los controles **lógicos** (identidad, red, aplicaciones, datos) como los **físicos** (acceso al Data Center, cableado, activos).

### 0.2 Alcance
Aplica a los 184 usuarios de la organización, a todo equipo propiedad de la empresa (estaciones de trabajo, servidores, equipo de red, telefonía IP), a la Nube Privada (SDN) y a cualquier tercero con acceso remoto o físico a las instalaciones (proveedores, contratistas, visitantes).

### 0.3 Marco de referencia
Este conjunto de políticas se inspira en los controles de **ISO/IEC 27001:2022 (Anexo A)** y el **NIST Cybersecurity Framework (Identify–Protect–Detect–Respond–Recover)** como estructura de referencia — no se declara cumplimiento certificado de ninguno de los dos (eso requeriría una auditoría formal fuera del alcance de este proyecto), pero las políticas siguen la misma lógica de organización que usan esos marcos, que es el estándar de facto en la industria.

### 0.4 Principios rectores
1. **Menor privilegio (least privilege)**: ningún usuario o servicio tiene más acceso del estrictamente necesario para su función.
2. **Defensa en profundidad**: no se depende de un solo control — la segmentación VLAN, el firewall, el proxy y el monitoreo son capas independientes y redundantes.
3. **Denegar por defecto, permitir por excepción**: toda regla de firewall/ACL parte de "denegado" y se abre explícitamente, nunca al revés.
4. **Separación de funciones**: quien solicita un acceso no es quien lo aprueba; quien administra un sistema no es el único que revisa sus logs.
5. **Trazabilidad**: toda acción administrativa relevante debe quedar registrada y asociada a una persona, no a una cuenta compartida.

---

## 1. Políticas Lógicas

### 1.1 Política de gestión de identidad y control de acceso
- Cada usuario tiene **una cuenta nominal única** (`nombre.apellido`) — prohibidas las cuentas genéricas o compartidas (`admin`, `soporte1`) salvo cuentas de servicio documentadas y con dueño responsable asignado.
- El acceso a cada recurso (VLAN, servidor, aplicación) se otorga por **rol/grupo**, no por excepción individual — un usuario nuevo en Ventas hereda automáticamente los permisos del grupo "Ventas", no se configuran permisos ad-hoc.
- **Revisión periódica de accesos (access review)**: cada 6 meses, Soporte I/T audita la lista de accesos activos contra la nómina vigente y revoca lo que ya no corresponde (bajas, cambios de puesto).
- **Baja inmediata**: al desvincularse un empleado, sus credenciales (correo, VPN, VLAN, Nextcloud) se desactivan el mismo día — no se elimina la cuenta de inmediato (se retiene 90 días para continuidad de negocio: reasignar correo, recuperar archivos) pero sí se bloquea el acceso.

### 1.2 Política de contraseñas y autenticación
- Longitud mínima de 12 caracteres, combinando mayúsculas, minúsculas, números y símbolo — alineado con la recomendación NIST SP 800-63B de priorizar longitud sobre complejidad forzada con expiración corta.
- **Sin expiración forzada arbitraria** (ej. cada 30 días) para cuentas de usuario estándar — la evidencia de la industria (NIST 800-63B) muestra que la rotación forzada frecuente lleva a patrones predecibles (`Enero2026!`, `Febrero2026!`). En su lugar: rotación inmediata ante sospecha de compromiso, y rotación obligatoria cada 90 días **solo** para cuentas administrativas/privilegiadas (root, admin de Proxmox, admin de R1/VyOS).
- **Autenticación multifactor (MFA) obligatoria** para: acceso VPN (WireGuard + llave del dispositivo, que ya es de por sí un segundo factor físico), acceso administrativo a Proxmox, MikroTik y VyOS, y acceso a Nextcloud desde fuera de la LAN.
- Bloqueo de cuenta tras 5 intentos fallidos consecutivos, con desbloqueo automático a los 15 minutos o manual por Soporte I/T.
- Prohibido almacenar contraseñas en texto plano en cualquier medio (correo, notas, chat) — se exige el uso de un gestor de contraseñas corporativo (ver también 1.15).

### 1.3 Política de segmentación de red
- La red se segmenta en VLANs por función/área (ver justificación completa en [00-Arquitectura-General.md](../00-Documentacion-General/00-Arquitectura-General.md) §3 y tabla en [07-Direccionamiento-IP-VLANs.md](../00-Documentacion-General/07-Direccionamiento-IP-VLANs.md)).
- El tráfico **inter-VLAN** solo se permite explícitamente por matriz de reglas (ver 1.3.1) — cualquier flujo no contemplado en la matriz queda denegado por defecto en R1.
- Ningún puerto de acceso de usuario final puede configurarse como puerto trunk — solo los enlaces switch-a-switch y switch-a-router lo son.
- Puertos físicos no utilizados en switches de acceso se desactivan administrativamente (`shutdown`) hasta que se asignen a un equipo, para evitar conexión no autorizada de dispositivos.

#### 1.3.1 Matriz de acceso inter-VLAN (resumen — regla base "todo lo no listado se deniega")

| Origen \ Destino | Servidores (50) | DMZ (70) | Cloud-Mgmt (80) | Internet |
|---|---|---|---|---|
| Usuarios (10/20/30/31/40/60) | Puertos de servicio específicos (SMTP, IMAP, HTTP intranet, DHCP relay) | Solo por Proxy | Denegado | Solo vía Proxy |
| DMZ (70) | Denegado | — | Denegado | Solo puertos publicados |
| Soporte I/T (40) | SSH/HTTPS de gestión | SSH/HTTPS de gestión | SSH/HTTPS de gestión | Sí |
| VPN (200) | Igual que la VLAN de origen del usuario remoto | Igual que su VLAN | Denegado | Vía Proxy |

### 1.4 Política de acceso remoto y VPN
- Todo acceso remoto a recursos internos se realiza exclusivamente vía **WireGuard** (ver [03-Fase3-LAN-WAN-VPN-Seguridad.md](../Fase-3-LAN-WAN-VPN-Seguridad/03-Fase3-LAN-WAN-VPN-Seguridad.md)) — prohibido exponer directamente a Internet cualquier servicio interno (RDP, SSH, SMB) sin pasar por la VPN.
- Cada perfil de VPN es individual y trazable a una persona; no se comparten perfiles.
- Los equipos personales (BYOD) que se conecten por VPN deben cumplir el mínimo de la política 1.7 (antivirus activo, SO actualizado) — verificación honor-based en esta fase del proyecto, con roadmap a verificación automática (postura de dispositivo) como mejora futura.

### 1.5 Política de gestión de parches y actualizaciones
- Servidores críticos (correo, proxy, DHCP, intranet, monitoreo): ventana de mantenimiento mensual para aplicar actualizaciones de seguridad del sistema operativo y de los contenedores Docker (`docker compose pull && docker compose up -d`).
- Vulnerabilidades **críticas** (CVSS ≥ 9.0) publicadas para software en producción: parche o mitigación en un plazo máximo de 72 horas, sin esperar la ventana mensual.
- Firmware de equipo de red (RouterOS, VyOS): actualización solo tras validar en el ambiente de laboratorio — nunca directo a producción — dado el riesgo de romper la adyacencia OSPF o reglas de firewall.

### 1.6 Política de uso aceptable (AUP — Acceptable Use Policy)
- Los recursos informáticos son para uso laboral; se tolera uso personal razonable siempre que no comprometa seguridad, productividad o ancho de banda (ej. streaming en horario laboral queda filtrado por categoría en el Proxy).
- Prohibido expresamente: instalar software no autorizado, deshabilitar controles de seguridad (antivirus, firewall local), conectar medios extraíbles no escaneados en equipos de la VLAN de Servidores.
- El uso de Internet queda sujeto a monitoreo y registro (ver 1.10) — se informa a cada usuario de esto al firmar su carta de ingreso, para cumplir el principio de transparencia.

### 1.7 Política de dispositivos y endpoints
- Todo equipo corporativo debe tener antivirus/EDR activo, disco cifrado (BitLocker/LUKS) y el sistema operativo con actualizaciones automáticas de seguridad habilitadas.
- Los servidores (VMs) siguen hardening base aplicado por el rol Ansible `common` (ver [05-Terraform-Ansible-IaC.md](../00-Documentacion-General/05-Terraform-Ansible-IaC.md)): sin cuentas por defecto, SSH solo por llave (nunca contraseña), `fail2ban` activo.

### 1.8 Política de clasificación y manejo de la información
| Nivel | Ejemplo | Controles mínimos |
|---|---|---|
| **Pública** | Página web corporativa | Sin restricción |
| **Interna** | Comunicados, manuales internos | Acceso solo desde la LAN/VPN, sin cifrado obligatorio |
| **Confidencial** | Contratos, datos de RRHH, código fuente | Cifrado en tránsito (TLS) y en reposo, acceso por rol, logging de acceso |
| **Restringida** | Datos financieros, credenciales, llaves privadas | Cifrado obligatorio, acceso nominal auditado, prohibido salir por correo sin cifrar |

### 1.9 Política de respaldo y recuperación
- **RPO (Recovery Point Objective)** objetivo: 24 horas para servidores de aplicación (snapshots diarios vía Proxmox `vzdump`), 1 hora para el buzón de correo (replicación de `/var/mail` a disco secundario).
- **RTO (Recovery Time Objective)** objetivo: 4 horas para restaurar cualquier servicio crítico desde el último respaldo válido.
- Los respaldos se prueban trimestralmente con una restauración real (no solo se verifica que el job "corrió sin error") — un respaldo no probado no cuenta como respaldo.
- Copia de los backups críticos fuera del host físico principal (disco externo separado o, a futuro, un segundo sitio), dado que actualmente todo corre en 1 solo nodo Proxmox.

### 1.10 Política de monitoreo, logging y auditoría
- Zabbix centraliza métricas de disponibilidad; los logs de autenticación, firewall (R1) y proxy (Squid) se envían a syslog centralizado en `vm-monitor`.
- Retención mínima de logs: 90 días en línea, con posibilidad de exportar a almacenamiento frío para retención extendida si el volumen de negocio lo justifica.
- Los logs de acceso administrativo (quién entró a R1, VyOS, Proxmox) se consideran **restringidos** (política 1.8) y solo los revisa Soporte I/T y, en auditorías, la gerencia.

### 1.11 Política de respuesta a incidentes
1. **Detección**: alerta de Zabbix, reporte de usuario, o hallazgo en revisión de logs.
2. **Contención**: aislar el equipo/VLAN afectado (mover a una VLAN de cuarentena o desconectar el puerto del switch) sin apagar el equipo (preservar evidencia en memoria si aplica).
3. **Erradicación**: identificar causa raíz, remover el vector (malware, credencial comprometida, vulnerabilidad sin parchar).
4. **Recuperación**: restaurar desde respaldo conocido-bueno si hubo compromiso de integridad.
5. **Lecciones aprendidas**: post-mortem documentado en un plazo de 5 días hábiles, con acciones correctivas asignadas y fecha de cierre.
- Se designa un responsable de incidentes por turno (rota entre Soporte I/T) con autoridad para tomar decisiones de contención sin esperar aprobación gerencial en incidentes activos.

### 1.12 Política de gestión de cambios
- Todo cambio a infraestructura de red o servidores en producción sigue el flujo de [Terraform-Ansible-IaC.md](../00-Documentacion-General/05-Terraform-Ansible-IaC.md): `terraform plan`/revisión de Pull Request antes de aplicar — nadie aplica un cambio no revisado directamente en producción, salvo remediación de emergencia (que se documenta retroactivamente en 24h).
- Cambios de alto impacto (cambio de esquema de direccionamiento IP, actualización mayor de RouterOS/VyOS, cambio de reglas de firewall del Core) requieren ventana de mantenimiento anunciada con 48 horas de anticipación.

### 1.13 Política de terceros y proveedores
- Cualquier proveedor con acceso remoto a un sistema (ej. soporte de un fabricante) recibe un perfil VPN temporal con fecha de expiración automática y acceso limitado únicamente al sistema que va a intervenir.
- Contratos con proveedores que procesen datos confidenciales de la empresa deben incluir cláusula de confidencialidad y notificación obligatoria de incidentes que los involucren.

### 1.14 Política de correo electrónico y anti-phishing
- Todo el correo entrante pasa por el filtro anti-spam de rspamd (SPF/DKIM/DMARC + scoring, ver [02-Fase2-Servidor-Correo.md](../Fase-2-Servidor-Correo/02-Fase2-Servidor-Correo.md)).
- Se realiza capacitación de concientización (phishing awareness) al ingreso y de forma anual — el phishing sigue siendo, según reportes de la industria (Verizon DBIR), uno de los vectores de compromiso inicial más comunes, y ningún control técnico lo sustituye por completo.
- Prohibido reenviar correo corporativo a cuentas personales (Gmail, Hotmail, etc.).

### 1.15 Política de cifrado
- Todo tráfico administrativo (SSH, HTTPS, API de Proxmox) usa cifrado en tránsito — nunca Telnet, HTTP plano ni SNMPv1/v2 para gestión (se usa SNMPv3 si se requiere monitoreo SNMP de los equipos de red).
- Certificados TLS autofirmados son aceptables dentro del laboratorio (dominios `.lab`, sin CA pública disponible); en un rollout de producción con dominio público real, se exige Let's Encrypt o CA corporativa.
- Contraseñas y secretos en el código de automatización (Ansible) se manejan exclusivamente vía Ansible Vault — nunca en texto plano en el repositorio (ver nota en [group_vars/all.yml](../../infra/ansible/group_vars/all.yml)).

---

## 2. Políticas Físicas

### 2.1 Política de control de acceso físico al Data Center
- Acceso restringido a personal autorizado explícitamente por lista (Soporte I/T + gerencia de I/T) — no "todo el departamento de I/T" por defecto.
- Control de acceso de doble factor como mínimo viable (tarjeta + PIN); biométrico (huella o facial) como estándar deseable en la implementación de producción, alineado a lo esperado en una instalación Tier 4 (ver [06-Data-Center-Tier4.md](06-Data-Center-Tier4.md) §6).
- Bitácora de acceso (quién, cuándo, motivo) retenida mínimo 1 año.
- Ningún acceso individual sin acompañamiento para personal no habitual (contratistas, proveedores) — siempre escoltados por alguien de la lista autorizada.

### 2.2 Política de videovigilancia
- Cobertura CCTV en accesos al Data Center, pasillos críticos y en el propio cuarto de equipos (ver diagrama de referencia del enunciado original).
- Grabación en NVR con almacenamiento redundante (RAID), retención mínima de 30 días, extensible ante un incidente en investigación.
- Acceso a las grabaciones restringido a gerencia y Soporte I/T — solicitud registrada, no acceso libre.

### 2.3 Política de gestión de visitantes y contratistas
- Todo visitante se registra en bitácora física/digital con nombre, motivo, empresa (si aplica) y hora de entrada/salida.
- Ningún visitante permanece a solas en el Data Center ni en cuartos de telecomunicaciones (IDF) bajo ninguna circunstancia.
- Contratistas que requieran trabajar en cableado o equipo activo deben coordinarse con Soporte I/T y quedar dentro de la ventana de mantenimiento correspondiente (política 1.12).

### 2.4 Política de protección ambiental
- Monitoreo continuo de temperatura y humedad del Data Center, con alertas automáticas hacia Zabbix ante desviación del rango objetivo (18–27 °C, 40–60% HR, ver ASHRAE TC9.9 y detalle en 06-Data-Center-Tier4.md).
- Detección temprana de humo por aspiración (VESDA o equivalente) antes que un detector puntual convencional — crítico en un cuarto con corriente de aire forzada donde el humo se dispersa rápido.

### 2.5 Política de gestión de activos físicos
- Todo equipo de red y servidor se inventaría con: número de serie, ubicación, fecha de compra, garantía, y responsable asignado — mantenido en el inventario de activos del proyecto (hoja de cálculo compartida por ahora; un IPAM/DCIM dedicado como NetBox se evalúa como mejora de producción, no es parte del stack activo del laboratorio — ver [00-Arquitectura-General.md](../00-Documentacion-General/00-Arquitectura-General.md)).
- Etiquetado físico obligatorio en todo equipo y en ambos extremos de cada cable, siguiendo el esquema de [04-Diseno-Fisico.md](04-Diseno-Fisico.md) §4.5 (norma TIA-606-B).
- **Disposición segura de medios**: discos duros/SSD dados de baja se destruyen físicamente o se borran con un método de sobrescritura certificado (ej. NIST SP 800-88) antes de salir de las instalaciones — nunca se revenden ni desechan con datos legibles.

### 2.6 Política de cableado y etiquetado
- Todo cableado sigue TIA/EIA-568-C (horizontal) y se etiqueta conforme TIA-606-B — ver esquema de nomenclatura en [04-Diseno-Fisico.md](04-Diseno-Fisico.md) §4.5.
- Prohibidos los empalmes fuera de patch panels o cajas de empalme certificadas.
- Separación física mínima entre bandejas de datos y de energía eléctrica (evitar interferencia electromagnética) según TIA-569-D.

### 2.7 Política de continuidad de energía
- Ver detalle completo de dimensionamiento y redundancia 2N en [06-Data-Center-Tier4.md](06-Data-Center-Tier4.md) §3-4.
- Pruebas de transferencia a planta eléctrica (ATS) programadas trimestralmente, en horario de bajo impacto, documentadas con resultado (éxito/falla y tiempo de transferencia real medido).

---

## 3. Roles y responsabilidades de seguridad (RACI resumido)

| Actividad | Responsable | Aprueba | Consultado |
|---|---|---|---|
| Administración de firewall (R1) | Soporte I/T | Gerencia I/T | — |
| Revisión de accesos (semestral) | Soporte I/T | Gerencia I/T | Cada jefe de área |
| Respuesta a incidentes | Responsable de turno (Soporte I/T) | Gerencia I/T | Legal (si aplica) |
| Gestión de cambios de infraestructura | Quien propone el cambio | Custodio del ambiente (ver [11-Equipo-y-Responsabilidades.md](../00-Documentacion-General/11-Equipo-y-Responsabilidades.md)) | Equipo técnico |
| Control de acceso físico al DC | Soporte I/T | Gerencia I/T | — |
| Revisión de video/bitácoras | Gerencia I/T | — | Soporte I/T |

## 4. Cumplimiento y sanciones

El incumplimiento de estas políticas se gradúa según impacto: desde llamada de atención documentada (ej. contraseña anotada en post-it) hasta suspensión de accesos y proceso disciplinario formal (ej. exfiltración deliberada de datos confidenciales, o desactivación intencional de un control de seguridad). Todo caso de incumplimiento con impacto en datos de terceros o clientes se evalúa además bajo la normativa de protección de datos aplicable en Guatemala.

## 5. Revisión y vigencia

Este documento se revisa como mínimo **una vez al año**, y de forma extraordinaria tras cualquier incidente de seguridad significativo o cambio mayor de infraestructura (ej. adopción de un nuevo Data Center, cambio de proveedor de VPN). Control de versiones del documento vía git, igual que el resto del proyecto — el historial de cambios de este archivo **es** el historial de revisiones de la política.
