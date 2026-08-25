# Punto 2 — Análisis de Requerimientos, Costos, Tráfico de Red y Cultura Organizacional

## 1. Requerimientos funcionales y no funcionales

Separar estas dos categorías es una práctica estándar de ingeniería de requerimientos que evita mezclar "qué debe hacer el sistema" con "qué tan bien debe hacerlo" — ambas son necesarias para poder validar el diseño al final.

### 1.1 Requerimientos funcionales (qué debe hacer la red)
- Debe enrutar tráfico entre todas las VLANs de la organización.
- Debe proveer salida redundante a Internet.
- Debe proveer VoIP interno.
- Debe permitir acceso remoto seguro (VPN).
- Debe alojar correo electrónico propio con anti-spam.
- Debe exponer un sitio/aplicación web públicamente sin comprometer la red interna (DMZ).
- Debe asignar direccionamiento IP automáticamente (DHCP) a todos los segmentos.
- Debe registrar y monitorear el estado de los servicios críticos.

### 1.2 Requerimientos no funcionales (qué tan bien debe hacerlo — SLA objetivo)

| Atributo | Objetivo | Cómo se mide/verifica |
|---|---|---|
| Disponibilidad del Core de red | 99.9% (≈ 8.76 h/año de inactividad tolerable) | Monitoreo Zabbix, uptime de R1 |
| Disponibilidad del Data Center (infraestructura) | 99.995% (Tier 4, ≈ 26.3 min/año) | Ver [06-Data-Center-Tier4.md](06-Data-Center-Tier4.md) |
| Latencia interna (LAN) | < 5 ms entre cualquier VLAN y el Core | Prueba de `ping` entre segmentos |
| Latencia WAN (a Internet) | < 80 ms promedio hacia destinos regionales | Monitoreo continuo |
| RPO (pérdida de datos máxima tolerable) | 24 h (servicios generales), 1 h (correo) | Ver política de respaldo, [05-Politicas-Seguridad.md](05-Politicas-Seguridad.md) §1.9 |
| RTO (tiempo de recuperación máximo) | 4 h para cualquier servicio crítico | Prueba trimestral de restauración |
| Capacidad de crecimiento sin rediseño | Hasta 4x la planta actual por VLAN | Direccionamiento `/24`, ver [03-Diseno-Logico.md](03-Diseno-Logico.md) |
| Tiempo de detección de incidentes | < 5 minutos desde que ocurre la falla | Alertas automáticas Zabbix |

## 2. Análisis de tráfico de red

### 2.1 Clasificación del tráfico por tipo (para diseño de QoS)

No todo el tráfico de red tiene los mismos requisitos de calidad de servicio — clasificarlo primero es lo que permite luego priorizarlo correctamente (ver marcado DSCP en [03-Diseno-Logico.md](03-Diseno-Logico.md)):

| Clase de tráfico | Ejemplos en Virtual Solutions | Sensibilidad | Prioridad relativa |
|---|---|---|---|
| Tiempo real | VoIP (VLAN 60) | Muy sensible a latencia/jitter, tolera algo de pérdida | Más alta |
| Interactivo | SSH/RDP de Soporte I/T, acceso a intranet | Sensible a latencia, no tolera pérdida | Alta |
| Transaccional | Correo, aplicaciones de Administración/Ventas | Moderadamente sensible | Media |
| Masivo (bulk) | Git, CI/CD, respaldos, transferencias de Desarrollo I/T | Tolera latencia, no debe afectar a las demás clases | Baja (pero garantizada, no bloqueada) |
| Mejor esfuerzo | Navegación web general | Tolerante | Más baja |

### 2.2 Estimación de demanda por área

| Área | Usuarios | Perfil de uso | Ancho de banda promedio/usuario | Total promedio | Total en hora pico (factor ×1.5) |
|---|---|---|---|---|---|
| Administración | 14 | Ofimática, ERP, correo | 0.5 Mbps | 7 Mbps | 10.5 Mbps |
| Ventas | 30 | CRM, videollamadas con clientes | 1 Mbps | 30 Mbps | 45 Mbps |
| Desarrollo I/T | 110 | Git, CI/CD, entornos de prueba, descargas grandes | 1.5 Mbps | 165 Mbps | 247.5 Mbps |
| Soporte I/T | 12 | Acceso remoto a equipos, tickets | 1 Mbps | 12 Mbps | 18 Mbps |
| Telefonía IP | 6 líneas concurrentes | VoIP (G.711 ≈ 87 Kbps/llamada + overhead RTP) | 0.1 Mbps | 0.6 Mbps | 0.6 Mbps (constante durante la llamada, no tiene "pico") |
| **Total** | | | | **≈ 214.6 Mbps** | **≈ 321.6 Mbps** |

### 2.3 Metodología de dimensionamiento — por qué el tráfico interno "supera" el ancho de banda de Internet (y por qué eso es correcto)

La demanda interna en hora pico (~321 Mbps) supera ampliamente los 2×10 Mbps de Internet contratados. **Esto no es un error de diseño** — es el resultado esperado de aplicar correctamente el concepto de **sobre-suscripción (oversubscription)** que se usa en cualquier diseño jerárquico de red:

- El tráfico de Desarrollo I/T (Git interno, builds, CI/CD, transferencias entre servidores) es en su enorme mayoría **tráfico este-oeste (LAN interna)** — nunca toca el enlace WAN. Un `git push` a un repositorio interno, un build de CI, o una consulta a la base de datos interna se mueve dentro de la LAN a velocidad de switch (1 Gbps por puerto), no a velocidad de Internet.
- Los enlaces de **acceso** (puerto de usuario, 1 Gbps) siempre están sobre-suscritos frente al **uplink** de su switch, y ese uplink a su vez frente al enlace WAN — es la razón de ser del modelo jerárquico (ver [03-Diseno-Logico.md](03-Diseno-Logico.md) §1): no se dimensiona cada capa para el 100% simultáneo de todos los usuarios, se dimensiona con una relación de sobre-suscripción razonable (típicamente 4:1 a 20:1 entre acceso y distribución en diseños empresariales), confiando en que no todos los usuarios saturan su enlace al mismo tiempo.
- Los 20 Mbps de WAN contratados son el ancho de banda relevante **solo** para lo que realmente sale a Internet: navegación, correo externo, videollamadas con clientes (Ventas), actualizaciones de paquetes, y el tráfico saliente vía Proxy.

**Conclusión de dimensionamiento**: 20 Mbps agregados (2×10) son suficientes para navegación + correo + videollamadas moderadas de Ventas en el estado actual. Recomendación formal al cliente: monitorear el consumo real vía Zabbix (ver [Fase3-LAN-WAN-VPN-Seguridad.md](../Fase-3-LAN-WAN-VPN-Seguridad/03-Fase3-LAN-WAN-VPN-Seguridad.md) §3) y evaluar ampliar a 2×20 Mbps en un horizonte de 12–18 meses si el uso de videoconferencia externa crece de forma sostenida.

## 3. Análisis de costos

### 3.1 CAPEX vs. OPEX
- **CAPEX (inversión inicial)**: equipo de red, cableado estructurado, racks, UPS — se deprecia en el tiempo (vida útil típica de 5-7 años para equipo de red, 10-15 años para cableado estructurado bien instalado).
- **OPEX (costo operativo recurrente)**: ninguno de software (100% open source), consumo eléctrico del Data Center, eventual soporte/mantenimiento de terceros si se contrata. La ausencia de licenciamiento de software es una decisión de diseño que convierte gran parte de lo que normalmente sería OPEX (licencias anuales) en costo cero recurrente — ahorro estructural, no puntual.

### 3.2 Costo evitado (cost avoidance) — el caso de VoIP
Comparación conceptual: líneas telefónicas tradicionales facturan por línea + consumo; un troncal VoIP interno sobre la infraestructura de datos ya construida no añade costo marginal por extensión interna, solo el costo del enlace hacia el proveedor de telefonía externa (si se requiere salida a la PSTN). El ahorro exacto depende de la tarifa actual del cliente con su proveedor telefónico (dato que no se provee en el enunciado) — se documenta la lógica del ahorro, no una cifra sin sustento.

### 3.3 Resumen de costos del proyecto
Ver desglose línea por línea, con precios reales cotizados en Guatemala, en [08-Equipo-Fisico-Presupuesto.md](../00-Documentacion-General/08-Equipo-Fisico-Presupuesto.md). Se distinguen dos alcances:

| Alcance | Qué incluye | Total aproximado |
|---|---|---|
| **Laboratorio de demostración** | 1 router, 1 switch VLAN, cableado mínimo, SSD portátil como servidor | ≈ Q1,480–1,890 |
| **Rollout de producción real** | Cableado completo para 166 puestos de trabajo, 12 switches (10 de acceso + 2 de distribución redundante, dimensionados por punto de red real, ver [04-Diseno-Fisico.md](04-Diseno-Fisico.md) §1.1), 2 routers Core redundantes, racks, UPS, teléfonos IP | ≈ Q186,391 (sin UPS central/generador del Data Center) |

## 4. Cultura organizacional

Un análisis de cultura organizacional superficial (reducirlo a 2-3 bullets) es uno de los errores más comunes en propuestas de red que después fallan en adopción — la tecnología correcta implementada sin entender cómo la organización realmente trabaja genera resistencia, bypass de controles (shadow IT) o subutilización. Este análisis se desarrolla en 7 dimensiones.

### 4.1 Perfil demográfico y estructura organizacional
Virtual Solutions tiene una distribución de personal fuertemente inclinada hacia perfiles técnicos: **110 de 184 personas (59.8%) son de Desarrollo I/T**, más 12 de Soporte I/T (6.5%) — es decir, **el 66.3% de la plantilla tiene perfil técnico/tecnológico**. Esto es atípico frente a una empresa de manufactura o servicios tradicionales, donde I/T suele representar 5-15% de la plantilla, y tiene consecuencias directas de diseño:

- Una plantilla mayoritariamente técnica tiende a tener **mayor capacidad de auto-servicio** (menor necesidad de soporte guiado paso a paso) pero también **mayor capacidad de encontrar y explotar debilidades de configuración** — el diseño de seguridad no puede asumir que "nadie va a intentar nada", debe ser robusto ante usuarios técnicamente capaces.
- La estructura implícita (Administración 14, Ventas 30, Desarrollo 110, Soporte 12) sugiere una organización **relativamente plana dentro de I/T** pero con áreas de negocio (Admin/Ventas) más tradicionales — el diseño de políticas (ver [05-Politicas-Seguridad.md](05-Politicas-Seguridad.md)) refleja esto tratando a Desarrollo como un bloque grande con reglas consistentes internamente, mientras Administración recibe controles diferenciados por su exposición a datos financieros.

### 4.2 Cultura técnica y su impacto en la adopción de tecnología
Al ser mayoritariamente una organización de desarrollo de software:
- **Alta tolerancia y hasta preferencia por herramientas open source self-hosted** — validado directamente por el hecho de que el enunciado exige "solo open source" para la Nube Privada, lo cual encaja naturalmente con esta cultura en vez de generar fricción, como podría ocurrir en una organización acostumbrada a suites comerciales (Microsoft 365, Google Workspace).
- **Expectativa de automatización e infraestructura como código** — un equipo de desarrollo espera (y sabe operar) herramientas como Terraform/Ansible/Git; esto reduce significativamente la resistencia al modelo de gestión de cambios basado en Pull Requests (ver [05-Politicas-Seguridad.md](05-Politicas-Seguridad.md) §1.12) que en una organización no técnica requeriría capacitación extensa.
- **Riesgo de "shadow IT"**: personal técnico capaz de levantar sus propias herramientas no autorizadas (un servidor casero, un túnel no oficial) si el proceso oficial es percibido como lento — el diseño mitiga esto ofreciendo herramientas self-service reales (VPN simple con WireGuard, Nextcloud) en vez de procesos burocráticos que inviten a buscar atajos.

### 4.3 Modelo de trabajo remoto/híbrido y sus implicaciones
El cliente declaró explícitamente el fomento de "trabajo a distancia y formación de equipos de trabajo virtuales". Esto no es un beneficio adicional en el diseño — es un requisito de primera clase:
- La VPN (WireGuard) y la intranet (Nextcloud) se diseñaron con la misma prioridad que la LAN física, no como un anexo de "acceso remoto básico" (ver [Fase3-LAN-WAN-VPN-Seguridad.md](../Fase-3-LAN-WAN-VPN-Seguridad/03-Fase3-LAN-WAN-VPN-Seguridad.md)).
- Un modelo híbrido implica que la capacidad de VPN debe dimensionarse pensando en picos de uso (ej. un evento que obligue a todo Ventas a trabajar remoto simultáneamente), no solo en el uso promedio diario.
- La cultura de trabajo remoto de una empresa de software suele acompañarse de **comunicación asíncrona** (tickets, chat, documentación) más que de reuniones presenciales constantes — refuerza la necesidad de una intranet robusta como fuente única de verdad de la información interna, no solo un repositorio de archivos.

### 4.4 Gestión del cambio y capacitación
- Dado el perfil técnico dominante, la **curva de adopción de MFA, VPN y políticas de acceso más estrictas será más corta** que en una organización no técnica — se puede introducir con documentación técnica directa (un README, no necesariamente una capacitación presencial extensa) para el 66% técnico de la plantilla.
- Para el 34% no técnico (Administración, Ventas, parte de Recepción), sí se requiere una estrategia de capacitación diferenciada: sesiones guiadas, no solo documentación — reconocido explícitamente en la política de concientización de phishing ([05-Politicas-Seguridad.md](05-Politicas-Seguridad.md) §1.14), que aplica a todos pero se refuerza particularmente en las áreas de menor exposición técnica previa.

### 4.5 Cultura de cumplimiento (dado el manejo de recursos monetarios)
El manejo explícito de "recursos monetarios" introduce una tensión cultural real: una empresa de desarrollo de software tiende culturalmente hacia la velocidad y la experimentación (moverse rápido, iterar), mientras que el manejo de datos financieros exige el opuesto (controles, aprobaciones, trazabilidad). El diseño resuelve esta tensión **segmentando el rigor por área**, no aplicando el mismo nivel de control a toda la empresa:
- Administración/Finanzas y Servidores reciben el tratamiento más estricto (VLAN separada, MFA, revisión de accesos, ver [05-Politicas-Seguridad.md](05-Politicas-Seguridad.md) §1.1).
- Desarrollo I/T mantiene mayor agilidad operativa (acceso amplio a herramientas de desarrollo) porque su perfil de riesgo es distinto y una fricción excesiva ahí sí tendría costo real en productividad, sin beneficio proporcional de seguridad.

### 4.6 Comunicación interdepartamental y necesidades de colaboración
Ventas necesita interactuar fluidamente con clientes **externos** (videollamadas, CRM), mientras Desarrollo colabora mayoritariamente **interno** (Git, code review, CI/CD) — esta diferencia de "frontera de confianza" ya está reflejada en el diseño de VLANs y en la DMZ ([03-Diseno-Logico.md](03-Diseno-Logico.md)): Ventas no requiere una DMZ propia, pero sí buena salida a Internet; Desarrollo requiere alto ancho de banda interno más que salida externa.

### 4.7 Rotación de personal e implicaciones en gestión de identidad
Las empresas de desarrollo de software tienen, como patrón de industria, una rotación de personal técnico más alta que áreas administrativas tradicionales. Esto se traduce directamente en un requerimiento técnico: la política de alta/baja de usuarios ([05-Politicas-Seguridad.md](05-Politicas-Seguridad.md) §1.1) exige revocación **el mismo día** de desvinculación, precisamente porque en un contexto de alta rotación, un proceso de baja lento acumula cuentas huérfanas activas — uno de los hallazgos de seguridad más comunes en auditorías reales.

### 4.8 Síntesis: de hallazgo cultural a decisión técnica

| Hallazgo cultural | Decisión técnica derivada |
|---|---|
| 66% de plantilla técnica | Seguridad robusta que no depende de "nadie sabe cómo saltársela"; automatización con IaC aceptada culturalmente |
| Preferencia por open source | Viable exigir 100% open source sin generar resistencia de adopción |
| Fomento de trabajo remoto | VPN + intranet con prioridad de diseño equivalente a la LAN física |
| Manejo de fondos + cultura ágil de desarrollo | Rigor de seguridad segmentado por área, no uniforme |
| Necesidad de Ventas de contacto externo | DMZ y salida a Internet priorizada para esa VLAN específica |
| Rotación esperable de personal técnico | Política de baja inmediata de accesos, cuentas nominales auditable |
