# Asignación de Diagramas Pendientes, Fase 3

Este documento reparte entre 3 personas del equipo los diagramas que respalda el documento de Fase 3, para que cada quien los rehaga en una herramienta de diagramación real (Draw.io / diagrams.net, Visio, o Packet Tracer para la topología física) en vez de dejarlos como el render automático de Mermaid que trae el Word.

El contenido de cada diagrama (qué nodos, qué conexiones, qué etiquetas) ya está resuelto en el documento de Fase 3, en la sección indicada. El trabajo pendiente es el dibujo profesional, no el diseño: cada persona debe abrir esa sección, ver el diagrama Mermaid ya renderizado como referencia de contenido, y reproducirlo con las herramientas y el estilo visual que el equipo use para el resto del proyecto.

La repartición busca equilibrar la carga: 3 diagramas por persona, mezclando diagramas simples y complejos en cada grupo.

## Persona 1: Conectividad externa y acceso remoto

| # | Diagrama | Sección | Qué debe mostrar |
|---|---|---|---|
| 1 | Failover de enlaces WAN | 3.2 | Los dos proveedores de Internet, el router Core, la verificación del gateway primario (IP SLA) y la conmutación hacia el enlace de respaldo |
| 2 | Arquitectura de la Intranet | 6.3 | Usuarios internos y usuarios remotos (vía VPN) llegando al contenedor de Nextcloud y su base de datos |
| 3 | Diseño lógico de la VPN | 5.3 | El cliente WireGuard en una ubicación externa, el NAT/port-forward en el Core, el servidor VPN y el pool de direcciones hacia los recursos internos |

Estos tres diagramas comparten tema: cómo entra y sale el tráfico de la organización hacia el exterior. Tiene sentido que una sola persona los dibuje con un estilo visual consistente (mismos íconos para "Internet", "Router Core", "servidor").

## Persona 2: Extranet, monitoreo y despliegue

| # | Diagrama | Sección | Qué debe mostrar |
|---|---|---|---|
| 4 | Acceso de Extranet (despacho contable) | 4.5 | El tercero externo con su túnel restringido, el Core, el servidor VPN, y la denegación explícita hacia el resto de las VLANs internas |
| 5 | Arquitectura de Monitoreo | 7.2 | El servidor Zabbix (interfaz web, servidor, base de datos), los agentes de cada VM de servicio, y el router Core reportando por SNMP |
| 6 | Arquitectura de despliegue (Ansible) | 13.2 | El inventario de red como fuente de verdad, el rol base común, los roles de VPN/Intranet/Monitoreo, y el rol de agente aplicado a todas las VMs |

Este grupo cubre el "quién entra desde fuera" (Extranet) y el "cómo se opera y se despliega por dentro" (Monitoreo, Ansible), dos vistas que se complementan.

## Persona 3: Zonas de seguridad e infraestructura física

| # | Diagrama | Sección | Qué debe mostrar |
|---|---|---|---|
| 7 | Zonas de la DMZ | 8.5 | Las 3 zonas (Internet, DMZ, LAN interna), el servidor Web, el servidor de correo, y las reglas de acceso entre cada zona, incluida la denegación DMZ → LAN interna |
| 8 | Distribución de racks del centro de datos | 9.3 | Los 3 racks (telecomunicaciones con Core-A/Core-B y Distribución-A/Distribución-B, cómputo, almacenamiento), los dos UPS alimentando a los tres por igual, y las conexiones entre racks |
| 9 | Topología física de red | 9.5 | Los 2 ISP, el par Core-A/Core-B, el par Distribución-A/Distribución-B, los enlaces Layer 3 con OSPF entre ambos pares, los switches de Acceso de cada piso conectados a ambas Distribución, y el clúster de virtualización conectado directamente al Core |

Este es el grupo más orientado a infraestructura física (el "dónde vive todo esto"), un tema distinto al de los otros dos grupos y con más peso en dibujo (9 y 8 tienen más nodos), compensado por ser solo 3 diagramas.

## Notas para las 3 personas

- Usar el mismo juego de íconos/colores en los 9 diagramas finales, aunque los dibuje gente distinta, para que el documento se vea como un solo trabajo y no como 3 estilos pegados.
- Los 9 diagramas ya están numerados y ubicados exactamente en las secciones de arriba dentro de `03-Fase3-LAN-WAN-VPN-Seguridad.md` y del Word final (`Fase3-VirtualSolutions-ENTREGA-FORMAL.docx`); no hace falta inventar contenido nuevo, solo llevar el que ya existe a la herramienta de diagramación.
- Al terminar, cada quien exporta su diagrama como imagen (PNG o SVG) con nombre `diagrama-0X-descripcion.png` (usando el número de esta tabla) para que sea fácil reemplazar el render de Mermaid por la versión final en el documento.
